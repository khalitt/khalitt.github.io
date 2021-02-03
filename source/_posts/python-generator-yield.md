---
title: pythons生成器(generator)、yield的理解
date: 2019-09-17 21:31:32
tags: 
- python
- generator
- yield
categories: python
---



参考：

[python中yield的用法详解——最简单，最清晰的解释](https://blog.csdn.net/mieleizhi0522/article/details/82142856)

[Python 中的黑暗角落（一）：理解 yield 关键字](https://liam.page/2017/06/30/understanding-yield-in-python/)

[[What does the “yield” keyword do?](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do)](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do) 推荐这个，第一个回答说的很清楚

[Python Generator](https://lotabout.me/2017/Python-Generator/) `yield`的部分说的很清楚



## 生成器Generator

### 特点

- 生成的时候不会计算内部的值，而是在调用`next()`	的时候才会计算**一次**
- 只能读取一次（read once），再次调用需要重新生成（通过itertools或者再次调用生成器）
- 没有`next()`，需要用`next(generator)`来读取下一个值
- 为什幺要用生成器：**节省空间，特别是对于一个很大list来说，完全算完可能时间要好久或者内存不足**。此时用生成器就可以逐个迭代出来，节省内存。

### 使用

- 先得到一个生成器，然后再next(逐个调用)

  ```python
  gen = generator(...) #some function or class that returns a generator
  while true:
      try:
          next(gen)
          '''
          Some other codes using result of `next(gen)`
          '''
      except StopIteration:
          break
  ```


- 执行完的generator直接调用会返回`StopIteration` error（即例子中直接跳到`except`语句）

  ```python
  while True:
      try:
          print(next(gen))
          print('='*20)
      except StopIteration:
          print('Already empty')
          break
  ```

  ```
  Already empty
  ```

  

## yield

- **用在函数中**，这个函数就变成了一个Generator，在调用这个函数的时候不会直接执行内部的语句，**而是返回一个Generator 对象**

- 直到调用`next(gen)`的时候才会执行函数内部的语句 **一次**，然后在 `yield`的地方停下来。

- 再下一次调用的时候，会从上一次`yield`语句的地方继续执行，直至函数 **执行完毕（即不再碰到`yield`语句**

### 例子

- yield停止，再次next则从这开始，`yield`赋值没用，**因为已经“return”回去了，剩下的是None值**

  ```python
  def test_gen():
      print('Begin')
      for i in range(3):
          res = yield i
          print('Finished yield once with res {}'.format(res))
  
  tg = test_gen()
  while True:
      try:
          print(next(tg))
          print('='*20)
      except StopIteration:
          print('All finished')
          break
  ```

  ```
  Begin
  0
  ====================
  Finished yield once with res None
  1
  ====================
  Finished yield once with res None
  2
  ====================
  Finished yield once with res None
  All finished
  ```

- 利用`generator.send()`改变yield的值，使赋值起效（当第二次迭代的时候，改变yield的值）

  ```python
  tg = test_gen()
  for i in range(2):
      if i==1:
          print(tg.send(5))
          print('+'*20)
      else:
          print(next(tg))
  ```

  ```
  Begin
  0
  Finished yield once with res 5
  1
  ++++++++++++++++++++
  ```

  