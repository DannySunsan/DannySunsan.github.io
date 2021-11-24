---
title: c++并发之<future>
date: 2021-11-24 09:52:45
tags:
     - c++
     - 多线程
     - 并发
categories: 编程类
---

# future

std::future 可以用来获取异步任务的结果，因此可以把它当成一种简单的线程间同步的手段。std::future 通常由某个 Provider 创建，你可以把 Provider 想象成一个异步任务的提供者，Provider 在某个线程中设置共享状态的值，与该共享状态相关联的 std::future 对象调用 get（通常在另外一个线程中） 获取该值，如果共享状态的标志不为 ready，则调用 std::future::get 会阻塞当前的调用者，直到 Provider 设置了共享状态的值（此时共享状态的标志变为 ready），std::future::get 返回异步任务的值或异常（如果发生了异常）。

\<future\>头文件包含以下几个类和函数：

+ Providers类：std::promise，std::package_task
+ Future类：std::future，shared_future
+ Provides函数：std::async()
+ 其他类型：std::future_error，std::future_errc，std::future_status，std::launch

## std::promise

promise 对象可以保存某一类型 T 的值，该值可被 future 对象读取（可能在另外一个线程中），因此 promise 也提供了一种线程同步的手段。在 promise 对象构造时可以和一个共享状态（通常是std::future）相关联，并可以在相关联的共享状态(std::future)上保存一个类型为 T 的值。

可以通过 get_future 来获取与该 promise 对象相关联的 future 对象，调用该函数之后，两个对象共享相同的共享状态(shared state)

+ promise 对象是异步 Provider，它可以在某一时刻设置共享状态的值。
+ future 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 ready，然后才能获取共享状态的值。

另外，std::promise 的 operator= 没有拷贝语义，即 std::promise 普通的赋值操作被禁用，operator= 只有 move 语义，所以 std::promise 对象是禁止拷贝的。

```c++
 void get_input(std::promise<int>& pi)
    {
        int x;
        std::cout << "Enter an inter va==\n";
        std::cin.exceptions(std::ios::failbit);

        try
        {
            std::cin >> x;
            pi.set_value(x); //set_value会将共享标志位置为ready
        }
        catch (std::exception&)
        {
            pi.set_exception(std::current_exception());
        }
    }

    void printSth(std::future<int>& fut)
    {
        try
        {
            int x = fut.get();//在共享标志位置为ready之前会被阻塞
            std::cout << x << endl;
        }
        catch (std::exception& e)
        {
            std::cout << "exception" << e.what() << endl;
        }
    }

    void startProcess()
    {
        std::promise<int> pi;
        std::future<int> fut = pi.get_future();

        std::thread th1(get_input, std::ref(pi));
        std::thread th2(printSth, std::ref(fut));

        th1.join();
        th2.join();
    }
```

std::promise::set_value_at_thread_exit 设置共享状态的值，但是只有当线程退出的时候promise对象才会置为ready

## std::packaged_task

std::packaged_task 对象内部包含了两个最基本元素，一、被包装的任务(stored task)，任务(task)是一个可调用的对象，如函数指针、成员函数指针或者函数对象，二、共享状态(shared state)，用于保存任务的返回值，可以通过 std::future 对象来达到异步访问共享状态的效果。

可以通过 std::packged_task::get_future 来获取与共享状态相关联的 std::future 对象。在调用该函数之后，两个对象共享相同的共享状态，具体解释如下：

+ std::packaged_task 对象是异步 Provider，它在某一时刻通过调用被包装的任务来设置共享状态的值。
+ std::future 对象是一个异步返回对象，通过它可以获得共享状态的值，当然在必要的时候需要等待共享状态标志变为 ready.

std::packaged_task 的共享状态的生命周期一直持续到最后一个与之相关联的对象被释放或者销毁为止。

``` c++
 void processWorks()
    {
        std::packaged_task<int(int)> task([](int total) {
            for (int i = 0; i < total; ++i)
            {
                Sleep(1000);
                cout << this_thread::get_id() << " : " << i << endl;
            }
            return total;
            });

        std::future<int> fut = task.get_future();       
        task(3);
        std::cout << fut.get() << std::endl;//此处会等待task执行完成，并返回结果
        
        task.reset();//重置 packaged_task 的共享状态 
        fut = task.get_future();
        std::thread(std::move(task), 5).detach();
        std::cout << fut.get() << std::endl;//此处会等待task执行完成，并返回结果
    }
```

## std::async()

```c++
// a non-optimized way of checking for prime numbers:
bool is_prime (int x) {
  std::cout << "Calculating. Please, wait...\n";
  for (int i=2; i<x; ++i) if (x%i==0) return false;
  return true;
}

int main ()
{
  // call is_prime(313222313) asynchronously:
  std::future<bool> fut = std::async (is_prime,313222313);

  std::cout << "Checking whether 313222313 is prime.\n";
  // ...

  bool ret = fut.get();      // waits for is_prime to return

  if (ret) std::cout << "It is prime!\n";
  else std::cout << "It is not prime.\n";

  return 0;
}
```

### Exception safety

| exception type | error condition | description|
| :------------- | :--------------- | :--------- |
| system_error | errc::resource_unavailable_try_again | The system isunable to start a new thread |

## shared_future

shared_future对象的行为类似于future对象，不同之处在于它可以被复制，并且多个shared_future对象可以共享其共享状态的所有权。它们还允许在准备就绪后多次检索处于共享状态的值。

Shared_future对象可以从future对象隐式转换(参见其构造函数)，也可以通过调用future::share显式获取。在这两种情况下，获取它的future对象将其与共享状态的关联转移到shared_future，并使其自身失效。

共享状态的生存期至少持续到与之关联的最后一个对象被销毁为止。从shared_future(使用成员get)中检索值不会释放其对共享状态的所有权(与futures不同)。因此，如果与shared_future对象关联，共享状态可以在最初获取它的对象中保存(如果有的话)。

```c++
 void shareWorks()
    {
        std::future<int> fut = async([] {
            cout << "async\n";
            return 1;
            });
        std::shared_future<int> sFut = fut.share();

        // 共享的 future 对象可以被多次访问.
        std::cout << sFut.get() << std::endl;
        std::cout << sFut.get() << std::endl;
    }
```
