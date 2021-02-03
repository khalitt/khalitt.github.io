---
title: python logging模块的一些使用
date: 2020-04-30 22:27:23
tags: logging
categories: python
---

## logging不打印

关键在于**设置根logger的logging的级别**，要在一开始设定g

```python
# 关键！！
logging.basicConfig(level=logging.DEBUG)
# 或者下面这个
logging.getLogger().setLevel(logging.DEBUG)
```



## 基本的场景

参考

[Logging Cookbook](https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook)



### 一些基本的操作/概念：

1. logger才有`.info, .error`等命令，具体来说，`logging.info, logging.error`等都是调用默认的logger输出，获取新的logger可以通过`logging.getLogger`
2. logger设定的级别是指超过该级别的才会被输出到logger对应的输出流中，比如logger设定的level是logging.INFO，那么logging.debug的信息并不会被看到
3. 同一个logger可以设定不同的输出流（通过`logger.addHandler`，`Handler`），输出到不同位置
4. `Handler`里面设置的级别同样是用于过滤信息的，这么做就可以**实现不同级别的信息输出到不同的地方**

### 同时打印到console和文件中

```python
if not os.path.exists('./logs/'):
    os.makedirs('./logs')
# 输出到文件
fh = logging.FileHandler('./logs/logs_per_day.log', mode='w')
fh.setLevel(logging.DEBUG)
# 输出到控制台
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)
logger = logging.getLogger('logs')
logger.addHandler(fh)
logger.addHandler(console)
```



### 设定不同的logger，输出不同的信息

下面第一个logger主要用于主要的输出，第二个logger则用于记录和jobid相关的信息



```python

if not os.path.exists('./logs/'):
    os.makedirs('./logs')
fh = logging.FileHandler('./logs/logs_per_day.log', mode='w')
fh.setLevel(logging.DEBUG)
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)
logger = logging.getLogger('logs')
logger.addHandler(fh)
logger.addHandler(console)

job_logger = logging.getLogger('task_jobid_csv')
job_logger.setLevel(logging.DEBUG)
fh2 = logging.FileHandler('./res/logs/task_jobid_csv.log', mode='w')
fh2.setLevel(logging.DEBUG)
job_logger.addHandler(fh2)
```



### 设置输出log的格式

[LogRecord attributes](https://docs.python.org/3/library/logging.html#logrecord-attributes)



```python
fmttr = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh = logging.FileHandler('./local_logs', mode='w')
fh.setLevel(logging.DEBUG)
fh.setFormatter(fmttr)

console = logging.StreamHandler()
console.setLevel(logging.DEBUG)
console.setFormatter(fmttr)

logger = logging.getLogger('logs')
logger.addHandler(fh)
logger.addHandler(console)
logger.setLevel(logging.DEBUG)
```





## FileHandler用完后需要关闭(release)

有两种方法：

1. https://stackoverflow.com/a/15474586

```python
handlers = self.log.handlers[:]
for handler in handlers:
    handler.close()
    self.log.removeHandler(handler)
```



2. `logger.shutdown()`，直接关闭所有logging