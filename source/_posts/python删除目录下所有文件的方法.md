---
title: python删除目录下所有文件的方法
tags: python
date: 2021-12-14 15:58:05
categories: 编程类
description: python3 递归删除文件及文件夹的简单尝试
---


## 删除所有文件及文件夹  

python中没有专门用于删除文件的接口，不过实现起来也不难，我这里提供了一种一种递归的方式，先利用os.listdir(path)获取目录下的所有文件，如果是文件夹就进行递归，直到最后一级；如果是文件，修改只读为可读写，删除文件。最后利用shutil.rmtree(path)删除所有目录。

```python  
import os
import shutil
import stat
 
def delFiles(path) -> bool:
    try:
        lstFiles = os.listdir(path)    
        for file in lstFiles:
            full_path = os.path.join(path,file)
            if os.path.isdir(full_path):
                delFiles(full_path)
            else:
                os.chmod(full_path,stat.S_IWRITE|stat.S_IWGRP|stat.S_IWOTH)
                os.remove(full_path)
        if os.path.exists(path):
            shutil.rmtree(path)
    except Exception as err:
        print(err)
        return False
    return True
```
