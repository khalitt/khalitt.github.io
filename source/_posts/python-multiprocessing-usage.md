---
title: python的多进程使用
typora-root-url: python-multiprocessing-usage
typora-copy-images-to: python-multiprocessing-usage
date: 2020-06-18 19:00:28
tags:
- python
- 多进程
categories: python
---

参考：

* 强烈推荐这个 [【Multiprocessing系列】Multiprocessing基础](https://thief.one/2016/11/23/Python-multiprocessing/)，清晰易懂，尤其推荐里面的两篇资源共享系列 [【Multiprocessing系列】共享资源](https://thief.one/2016/11/24/Multiprocessing%E5%85%B1%E4%BA%AB%E8%B5%84%E6%BA%90/)  [【Multiprocessing系列】子进程返回值](https://thief.one/2016/11/24/Multiprocessing%E5%AD%90%E8%BF%9B%E7%A8%8B%E8%BF%94%E5%9B%9E%E5%80%BC/)，其中后者更通用，因为前者里面的方法Windows不适用

* 简单的模版可以参考[正确使用 Multiprocessing 的姿势](https://jingsam.github.io/2015/12/31/multiprocessing.html)

* [一文看懂Python多进程与多线程编程(工作学习面试必读)](https://zhuanlan.zhihu.com/p/46368084) 开篇关于线程、进程的说明很清晰

* [一篇文章搞定Python多进程(全)](https://juejin.im/post/5cce9e20f265da036b4a76d6#heading-4) 可以参考里面的例子。**特别是最后一个关于map的例子**

* 莫凡的例子也不错 [进程池 Pool](https://morvanzhou.github.io/tutorials/python-basic/multiprocessing/5-pool/)

  



# 基础概念

## 一些api

* 创建Process相关的：[`multiprocessing.Process()`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Process)
  * 可以直接调用默认的构造函数，创建新的进程
  * 也可以继承`Process`，像Thread的使用那样
  * `.join()`方法用来阻塞主进程（多线程时也是一样的含义），**即某子进程调用`.join()`方法，则主进程会一直等到该子进程介绍，或者到达设定的时间，才会继续执行`join`后面的代码。详参 [python多进程的理解 multiprocessing Process join run](https://www.cnblogs.com/lipijin/p/3709903.html)**：

![image-20200618191653558](/image-20200618191653558.png)

* 进程池 Pool，设定好进程数，避免了手动管理的麻烦，**这个很重要**：
  * 顺序执行任务（串行）：
    * apply
    * map（类似map的用法，输入一个可迭代的变量即可），**但最后的输出不同于类似map（无需在外面套一个list)，而是直接返回一个list**

```python
# coding=utf-8
from multiprocessing import Pool
import multiprocessing
import time


def task(pid):
    # do something
    result = str(pid)
    return result


def main():
    # multiprocessing.freeze_support()
    cpus = multiprocessing.cpu_count()
    pool = multiprocessing.Pool(cpus)
    results = []

    rslt = pool.map(task, range(10))
    pool.close()
    pool.join()

    # 输出['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
    print(rslt) 
    
    


if __name__ == '__main__':
    main()

```



* 异步执行（并行），注意返回的并不是list或者最后的结果，而是[`multiprocessing.pool.AsyncResult` ](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.pool.AsyncResult)，需要再调用`get` （还有一些别的方法）:

  * apply_async
  * map_async，同样类似上面的的map，只是返回的是AsyncResult，需要再get获得最后的结果

  ```python
  # coding=utf-8
  from multiprocessing import Pool
  import multiprocessing
  import time
  from time import sleep
  
  
  def task(pid):
      # do something
      result = str(pid)
      if pid % 2 == 0:
          sleep(5)
      return result
  
  
  def main():
      # multiprocessing.freeze_support()
      cpus = multiprocessing.cpu_count()
      pool = multiprocessing.Pool(cpus)
      results = []
  
      rslt = pool.map_async(task, range(10))
      pool.close()
      pool.join()
      
      # 注意，只有当所有进程都结束之后才会执行下面的print，因此上面task中的sleep无效
      # 结果：['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
      print(rslt.get())
  
  
  if __name__ == '__main__':
      main()
  
  ```

  

* 通信，数据共享相关的：Value，Array（这两个都只能用c的数据类型，不推荐），**Queue(最常用的）**，Manager



## 基本的思路

1. 创建并开启多个进程：

- 可以类似Thread那样，通过循环创建并start
- 或者通过进程池Pool创建，就不需要通过start

2. （如果是Pool，关闭进程池`Pool.close`）
3. join，等所有的子进程结束，处理结果



# 一些模版

## 用Process创建进程，并用[shared variable](http://docs.python.org/library/multiprocessing.html#sharing-state-between-processes)收集结果（略复杂）

[Python多进程如何获取函数的返回值](https://segmentfault.com/q/1010000010403117)

```python
def worker(procnum, return_dict):
    '''worker function'''
    print str(procnum) + ' represent!'
    return_dict[procnum] = procnum

if __name__ == '__main__':
    manager = Manager()
    return_dict = manager.dict()
    jobs = []
    for i in range(5):
        p = multiprocessing.Process(target=worker, args=(i,return_dict))
        jobs.append(p)
        p.start()

    for proc in jobs:
        proc.join()
    # 最后的结果是多个进程返回值的集合
    print return_dict.values()
```

* Manager支持`list`, [`dict`](https://docs.python.org/2.7/library/stdtypes.html#dict), [`Namespace`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.managers.Namespace), [`Lock`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Lock), [`RLock`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.RLock), [`Semaphore`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Semaphore), [`BoundedSemaphore`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.BoundedSemaphore), [`Condition`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Condition), [`Event`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Event), [`Queue`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Queue), [`Value`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Value) and [`Array`](https://docs.python.org/2.7/library/multiprocessing.html#multiprocessing.Array).



来自官方的例子 [Sharing state between processes](https://docs.python.org/2.7/library/multiprocessing.html#sharing-state-between-processes)

```python
from multiprocessing import Process, Manager

def f(d, l):
    d[1] = '1'
    d['2'] = 2
    d[0.25] = None
    l.reverse()

if __name__ == '__main__':
    manager = Manager()

    d = manager.dict()
    l = manager.list(range(10))

    p = Process(target=f, args=(d, l))
    p.start()
    p.join()

    print d
    print l
```



## 创建进程池Pool，使用map_async或者apply_async，用list把所有结果收集起来

`apply_async`：

```python
def task(pid):
    # do something
    return result

def main():
    pool = multiprocessing.Pool()
    cpus = multiprocessing.cpu_count()
    results = []

    for i in xrange(0, cpus):
        result = pool.apply_async(task, args=(i,))
        results.append(result)

    pool.close()
    pool.join()

    for result in results:
        print(result.get())
```



`map_async`：

```python
# coding=utf-8
from multiprocessing import Pool
import multiprocessing
import time
from time import sleep


def task(pid):
    # do something
    result = str(pid)
    if pid % 2 == 0:
        sleep(2)
    return result


def main():
    # multiprocessing.freeze_support()
    cpus = multiprocessing.cpu_count()
    pool = multiprocessing.Pool(cpus)
    results = []

    rslt = pool.map_async(task, range(10))
    pool.close()
    pool.join()
    print(rslt.get())


if __name__ == '__main__':
    main()

```



## Pool.map/ Pool.map_async的实践

[进程池map方法](https://juejin.im/post/5cce9e20f265da036b4a76d6#heading-9)

> 这段代码的主要工作就是将遍历传入的文件夹中的图片文件，一一生成缩略图，并将这些缩略图保存到特定文件夹中。这我的机器上，用这一程序处理 6000 张图片需要花费 27.9 秒。 map 函数并不支持手动线程管理，反而使得相关的 debug 工作也变得异常简单。

```python
import os 
import PIL 

from multiprocessing import Pool 
from PIL import Image

SIZE = (75,75)
SAVE_DIRECTORY = \'thumbs\'

def get_image_paths(folder):
    return (os.path.join(folder, f) 
            for f in os.listdir(folder) 
            if \'jpeg\' in f)

def create_thumbnail(filename): 
    im = Image.open(filename)
    im.thumbnail(SIZE, Image.ANTIALIAS)
    base, fname = os.path.split(filename) 
    save_path = os.path.join(base, SAVE_DIRECTORY, fname)
    im.save(save_path)

if __name__ == \'__main__\':
    folder = os.path.abspath(
        \'11_18_2013_R000_IQM_Big_Sur_Mon__e10d1958e7b766c3e840\')
    os.mkdir(os.path.join(folder, SAVE_DIRECTORY))

    images = get_image_paths(folder)

    pool = Pool()
    pool.map(creat_thumbnail, images) #关键点，images是一个可迭代对象
    pool.close()
    pool.join()

```



# 一些tricks

## Pool.map/Pool.map_async传入多个参数

[pool.map - multiple arguments](http://python.omics.wiki/multiprocessing_map/multiprocessing_partial_function_multiple_arguments)

* 通过传入list of list(or anything suitable)

* partial



## 通过ctrl-c结束进程池

大体思路：替换系统的signal int

[Catch Ctrl+C / SIGINT and exit multiprocesses gracefully in python [duplicate]](https://stackoverflow.com/a/35134329)

![image-20200619144551444](/image-20200619144551444.png)

**但是这个是不行的，实测不行**

实测应该用[Python 中 Ctrl+C 不能终止 Multiprocessing Pool 的解决方案](https://segmentfault.com/a/1190000004172444)



# 实战



