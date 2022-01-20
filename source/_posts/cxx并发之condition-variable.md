---
title: cxx并发之condition_variable
date: 2021-11-25 09:46:10
tags:
    - c++
    - 并发
    - 多线程
categories: 编程类
description: condition_variable实现并发的简单介绍
---

## condition_variable

[参考](http://www.cplusplus.com/reference/condition_variable/)

条件变量是一种能够阻塞调用线程的对象，直到通知恢复为止。

当调用它的一个等待函数时，它使用unique_lock(通过互斥锁mutex)来锁定线程。线程保持阻塞状态，直到被另一个线程唤醒，该线程调用同一个condition_variable对象上的通知函数。

类型为condition_variable的对象总是使用unique_lock\<mutex\>来等待;对于适用于任何类型的可锁定类型的替代方法，请参阅condition_variable_any。

三个wait functions，

|            |                                    |
| :--------- | ---------------------------------- |
| wait       | Wait until notified                |
| wait_for   | Wait for timeout or until notified |
| wait_until | Wait until notified or time point  |

两个notify functions,

|            |            |
| :--------- | :--------- |
| notify_one | Notify one |
| notify_all | Notify all |

### wait

```c++
  void fff()
    {
        for (int i = 0; i < 5; ++i)
        {
            cout << this_thread::get_id() << " : " << i << endl;
        }
    }
    
    void startProcess() 
    {
        mutex mu;
        condition_variable cdv;
        bool bLock = false;
        thread th1([&]
            {
                for (;;)
                {
                    {
                        std::unique_lock<std::mutex> lock(mu);
                        cdv.wait(lock,
                            [&bLock] { return bLock; });

                        bLock = false;
                    }
                    fff();
                }
                
            }
        );

        thread th2([&]
            {
                for (;;)
                {
                    {
                        std::unique_lock<std::mutex> lock(mu);
                        cdv.wait(lock,
                            [&bLock] { return bLock; });

                        bLock = false;
                    }
                    fff();
                }        
            }
        );

        for(;;)
        {
            {
                std::unique_lock<std::mutex> lock(mu);
                bLock = true;              
            } 
            Sleep(5000);
            cdv.notify_one();
        }
    }
```

### wait_for

```c++
while (cdv.wait_for(lock, std::chrono::seconds(1)) == std::cv_status::timeout) {
      std::cout << this_thread::get_id() <<'.' << std::endl;
}
```

### wait until

```c++
 xtime t;
 t.sec = 1;
 while (cdv.wait_until(lock, &t) == std::cv_status::timeout) {
       std::cout << this_thread::get_id() << '.' << std::endl;
 }
```
