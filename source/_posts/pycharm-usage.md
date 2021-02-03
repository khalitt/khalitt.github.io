---
title: pycharm
date: 2019-10-07 10:43:26
tags: pycharm
categories: pycharm

---



## IdeaVim

实用查询网址：[IdeaVim键位查询](http://ideavim.sourceforge.net/vim/index.html)

### z系列：折叠相关

### g系列：查找相关（查找reference，usage等等）

### 常用命令

- 左右移动: b, ge/ w
- 查找定义: K
- 查找使用:gd(当前函数中的定义)，gD(当前文件中的定义)
- 代码折叠：zc 
- 代码折叠打开：zo
- 所有代码折叠：zM
- 所有代码折叠都打开: zR

### 大小写W w E e B b的区别

默认的小写把各种标点符号作为分隔，而大写则是把space作为分隔，**因此大写应该是比较常用的**

[What is difference between w and W in escape mode of vim?](https://unix.stackexchange.com/a/106429)

> ```
> w   jump by start of words (punctuation considered words)
> W   jump by words (spaces separate words)
> e   jump to end of words (punctuation considered words)
> E   jump to end of words (no punctuation)
> ```



### multiple cursors

> ## multiple-cursors
>
> - Setup:
>
>    
>
>   ```
>   set multiple-cursors
>   ```
>
>   - Alternative vim-plug / vundle syntax`Plug 'https://github.com/terryma/vim-multiple-cursors'`
>     `Plug 'terryma/vim-multiple-cursors'`
>     `Plug 'vim-multiple-cursors'`
>
> - Emulates [vim-multiple-cursors](https://github.com/terryma/vim-multiple-cursors)
>
> - Commands: `<A-n>`, `<A-x>`, `<A-p>`, `g<A-n>`



而原版的vim-multiple-cursors

> ## Mapping
>
> If you don't like the plugin taking over your key bindings, you can turn it off and reassign them the way you want:
>
> ```bash
> let g:multi_cursor_use_default_mapping=0
> 
> " Default mapping
> let g:multi_cursor_start_word_key      = '<C-n>'
> let g:multi_cursor_select_all_word_key = '<A-n>'
> let g:multi_cursor_start_key           = 'g<C-n>'
> let g:multi_cursor_select_all_key      = 'g<A-n>'
> let g:multi_cursor_next_key            = '<C-n>'
> let g:multi_cursor_prev_key            = '<C-p>'
> let g:multi_cursor_skip_key            = '<C-x>'
> let g:multi_cursor_quit_key            = '<Esc>'
> ```

比较下来：

* A指的是Alt，**任何模式都可以使用**
* ideavim这里只有简单的几个功能，具体用起来只有`<A-n>, <A-x>`起效

最佳还是用idea自身的multiple cursors 功能，参考 [IntelliJ IDEA Tips & Tricks: Multiple Cursors](https://www.vojtechruzicka.com/intellij-idea-tips-tricks-multiple-cursors/)



## Remote Debug

### 法1：通过Pycharm的Remote Dedugging(强推！！)

- 参考：

1. 官方教程：[Remote debugging with the Python remote debug server configuration](https://www.jetbrains.com/help/pycharm/remote-debugging-with-product.html#remote-debug-config)
2. 这个帖子说的非常清楚，详参这个：[How do I start up remote debugging with PyCharm?](https://stackoverflow.com/a/7061956)
3. 还可以参考这个关于docker的：[From inside of a Docker container, how do I connect to the localhost of the machine?](https://stackoverflow.com/a/24326540)
4. 详细中文解释：[利用 PyCharm 进行 Python 远程调试](https://debugtalk.com/post/remote-debugging-with-pycharm/)



- server和client的定义：这里实际上是通过 **本地机扮演server，remote的server扮演client的角度**，来实现remote debugging的。

  > PyCharm (or your ide of choice) acts as the "server" and your application is the "client"; so you start the server first - tell the IDE to 'debug' - then run the client - which is some code with the `settrace` statement in it. When your python code hits the `settrace` it connects to the server - pycharm - and starts feeding it the debug data.

- 具体的原理就是通过ssh，当在client（服务器上）运行的code到达了`settrace`的地方时，则会通过制定好的地址和端口进行通讯，把服务器的数据回传到本地的IDE中（即server端）

- set_trace的具体参数详解，`pydevd_pycharm.settrace(<host name>, port=<port number>)`：

  1. `host name`指的是运行在本地的host，**具体而言指的是本机IDE在局域网中的IP（可通过ipconfig）查看**

2. `port name`默认是0，表示随机生成一个端口，也可以指定为IDE本机上空闲的端口
3. `suspend` 默认为True，即会在`settrace`的地方停下，如果设为`False`，**则会在断点或者exception处停下（如果在View Breakpoint中设定了any exception）**

- 具体使用方法：

  1. 到Configuration----Edit Configuration----Python Remote Debug中新增一个Configuration

  2. 设定好本地ide在局域网中的IP和port之后，根据图中的提示在remote server上安装对应的版本的pydev-pycharm（注意！**一定要安装对应版本的pydev-pycharm**，命令中`~=`部分后的数字即版本号）

  3. 设定好映射的路径，注意只用`base_name`（即待运行文件所在folder的映射）即可。

  4. 在待运行的文件开头插入对应的代码片段

  5. 本地IDE开启调试模式，remote server上用`srun`等工具运行命令，则可进入调试模式。





### Automatically upload 可能失效的原因：

- 关键在于文件要 **被保存！！**，因此需要触发文件被保存
- 最佳打开方式：通过Ctrl+S。因此此时很有可能是Ctrl+S失效了，原因是因为和VIm emulation冲突了
- 第二个还可以注意打开`Save files automatically if application is idle for `