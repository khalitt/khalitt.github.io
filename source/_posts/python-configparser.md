---
title: python config 读取与config parser的使用
date: 2019-09-10 08:57:52
tags:
- python
- configparser
categories:
- python
---



主要参考：

[configparser— Configuration file parser](https://docs.python.org/3/library/configparser.html#supported-ini-file-structure)

# Config简介

- 主要就是本地读取.ini文件，其中放入自己需要的参数。
- .ini文件的编写需要分section，**默认应该都有一个[DEFAULT]区**，来实现fallback, i.e.， 当某个section里的参数没有设定的时候，在文件中强行读取会用[DEFALUT]中的来代替。



# 具体使用

## 格式

- 默认以key = value 或者 key : value 来写。 section names are case sensitive but keys are not

- 注释用 # 来代替
  ```python
  [You can use comments]
  # like this
  ; or this
  
  # By default only in an empty line.
  # Inline comments can be harmful because they prevent users
  # from using the delimiting characters as parts of values.
  # That being said, this can be customized
  ```
- value很长的时候，通过缩进实现：如
  ```python
      [Sections Can Be Indented]
          can_values_be_as_well = True
          does_that_mean_anything_special = False
          purpose = formatting for readability
          multiline_values = are
              handled just fine as
              long as they are indented
              deeper than the first line
              of a value
          # Did I mention we can indent comments, too?
  ```





## 引用：Interpolation of values

- 同一个section里面：%(my_dir)s 直接引用别的参数，如

  ```python
  [Paths]
  home_dir: /Users
  my_dir: %(home_dir)s/lumberjack
  my_pictures: %(my_dir)s/Pictures
  ```

  上述的my_dir 实际就是/Users/lumberjack，my_pictures就是/Users/lumberjack/Pictures。
  
- 跨Section引用：需要用`${section:option}`来实现：

  ```python
  [Common]
  home_dir: /Users
  library_dir: /Library
  system_dir: /System
  macports_dir: /opt/local
  
  [Frameworks]
  Python: 3.2
  path: ${Common:system_dir}/Library/Frameworks/
  ```

  或者同一个section里面也可以这样用（就忽略了section这一项）

  ```python
  [Paths]
  home_dir: /Users
  my_dir: ${home_dir}/lumberjack
  my_pictures: ${my_dir}/Pictures
  ```
  

  

## 写入config文件

像字典一样操作

```python
>>> import configparser
>>> config = configparser.ConfigParser()
>>> config['DEFAULT'] = {'ServerAliveInterval': '45',
...                      'Compression': 'yes',
...                      'CompressionLevel': '9'}
>>> config['bitbucket.org'] = {}
>>> config['bitbucket.org']['User'] = 'hg'
>>> config['topsecret.server.com'] = {}
>>> topsecret = config['topsecret.server.com']
>>> topsecret['Port'] = '50022'     # mutates the parser
>>> topsecret['ForwardX11'] = 'no'  # same here
>>> config['DEFAULT']['ForwardX11'] = 'yes'
>>> with open('example.ini', 'w') as configfile:
...   config.write(configfile)
...
```



## 读取config文件（类似argparser)

```python
>>> config = configparser.ConfigParser()
>>> config.sections()
[]
>>> config.read('example.ini')
['example.ini']
>>> config.sections()
['bitbucket.org', 'topsecret.server.com']
>>> 'bitbucket.org' in config
True
>>> 'bytebong.com' in config
False
>>> config['bitbucket.org']['User']
'hg'
>>> config['DEFAULT']['Compression']
'yes'
>>> topsecret = config['topsecret.server.com']
>>> topsecret['ForwardX11']
'no'
>>> topsecret['Port']
'50022'
>>> for key in config['bitbucket.org']:  
...     print(key)
user
compressionlevel
serveraliveinterval
compression
forwardx11
>>> config['bitbucket.org']['ForwardX11']
'yes'
```



# 注意事项

## 与dict的区别：

- 对key的使用的都是case-insensitve的（dict不是），因此默认**全部都是小写**。
- .clear()不会清除掉所有东西，因为还保留着`DEFAULT`section 



## 使用interpolation要对configparser传入参数

[extended-interpolation-not-working-in-configparser](https://stackoverflow.com/questions/42839889/extended-interpolation-not-working-in-configparser)



具体就是（换成BasicInterpolation也同理）：

```python
from configparser import ConfigParser, ExtendedInterpolation

parser = ConfigParser(interpolation=ExtendedInterpolation())
parser.read('configfile.ini')

lll = parser.get('HUE_310', 'partNewFilePath')
print(lll)  # -> http://some_domain_name:8888
```



## 无法直接使用DEFAULT section

```bash
[DEFAULT]
root = /home/weitaotang/noisy_label/GNN/mnist_gnn_pytorch
log_dir = ${root}/log/
summary_dir = ${root}/summary
g_dataset_dir = ${root}/graph_dataset
num_classes = 10
load_cnn_model = CNN4_r_0.6_acc_86.41_ep_2_lr_0.01
num_split = split_5
g_raw_data_dir = ${g_dataset_dir}/split/${load_cnn_model}/${num_split}
pt = ${g_raw_data_dir}/raw/${num_split}in60.pt
csv = ${g_raw_data_dir}/raw/dist_matrix_0-510554_split_5in60.csv
confidence_rate = 0.3
```

这样会报错：

```python
config = ConfigParser(interpolation=ExtendedInterpolation())
config.read('../config/build_graph.ini')
config['pt']
```

需要这样：

```python
config = ConfigParser(interpolation=ExtendedInterpolation())
config.read('../config/build_graph.ini')
config = config['DEFAULT'] # 把DEFAILT节加入
config['pt']
```





