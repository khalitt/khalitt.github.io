---
title: ipython无法启动
typora-root-url: ipython-hanging
typora-copy-images-to: ipython-hanging
date: 2021-01-13 09:41:50
tags: ipython
categories: problems
---



# 问题

ipython无法启动：

* 无法使用remote debugger，表现为在ipython console中一输入命令就卡死。而后尝试单独用ipython/ipdb，发现症状类似，同样是输入ipython就会卡死
* 细查进程，发现一旦在ipython console开始输入命令后，就从原来的`SL+`变成了`DL+`
* `ipython --log-version=DEBUG`发现卡死在搜索config中

![image-20210113111830260](/image-20210113111830260.png)



# 原因

参考 [ipython won't start](https://github.com/ipython/ipython/issues/11678#issuecomment-527597571)

NFS文件系统挂载目录发生了变化，导致默认的sqlite无法使用。

> ```bash
> #  By default, IPython will put the history database in the IPython profile
> #  directory.  If you would rather share one history among profiles, you can set
> #  this value in each, so that they are consistent.
> #
> #  Due to an issue with fcntl, SQLite is known to misbehave on some NFS mounts.
> #  If you see IPython hanging, try setting this to something on a local disk,
> #  e.g::
> #
> #      ipython --HistoryManager.hist_file=/tmp/ipython_hist.sqlite
> ```



# 解决方法

1. 通过`ipython create profile`在`~/.ipython/profile_default`下创建两个config python脚本：`ipython_config.py` 和`ipython_kernel_config.py`
2. 修改`ipython_config.py`中的`c.HistoryAccessor.hist_file`为有效的目录，比如按照注释中的`/tmp/ipython_hist.sqlite`（**推荐：由于NFS存在问题，因此最好在`/tmp`目录下创建属于自己的`sqlite`，比如`/tmp/xiaoming_ipython_hist.sqlite`**）：

```bash
 c.HistoryAccessor.hist_file = '/tmp/ipython_hist.sqlite'
```





# PS

## 修改ipython默认的configure files

参考 [Setting iPython default profile](https://stackoverflow.com/a/28254181)

最简单的方法是**直接修改`profile_default`中的文件**，或者**启动时指定`profile`：`--profile=testing`**

> Take a backup of profile_default and then rename profile_testing to profile_default.
>
> Or you can copy the contents of profile_testing to profile_default.



## jupyter notebook有类似的hanging的问题

原因和ipython hanging的原因一样，因此解决方法类似：

1. 如果没有config文件， 首先通过`jupyter notebook --generate-config`
2. 然后在`~/.jupyter`下找到`jupyter_notebook_config.py`，然后编辑其中的`NotebookNotary.db_fileUnicode`选项

> NotebookNotary.db_fileUnicode
>
> Default: `''`
>
> The sqlite file in which to store notebook signatures. By default, this will be in your Jupyter data directory. You can set it to ‘:memory:’ to disable sqlite writing to the filesystem.

```python
## The sqlite file in which to store notebook signatures. By default, this will
#  be in your Jupyter data directory. You can set it to ':memory:' to disable
#  sqlite writing to the filesystem.
c.NotebookNotary.db_file = '/tmp/weitaotang_jupyter.sqlite'
#c.NotebookNotary.db_file = ''

```

