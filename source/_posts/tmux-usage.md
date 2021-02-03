---
title: tmux使用
date: 2019-12-31 10:11:43
tags: tmux
categories: linux
---

## tmux基础概念

[tmux使用详解](https://nightfarmer.top/post/tmux/)

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20191231104630.png" alt="" width="750px" >

> 由tmux创建的终端都是处于三级结构的末级节点，也就是pane节点，默认情况下用tmux创建一个终端，则会创建一个session，session中会包含一个Window，这个window下包含一个pane，而这个pane就被认作是一个新的终端。


另外强烈推荐tmux wiki上推荐的书：[The tao of tmux](https://leanpub.com/the-tao-of-tmux/read)

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20191231111931.png" alt="" width="750px" >

上面这张图把所有的概念都说比较清楚了：

1. server 和 client ：
    1. server：对应着tmux server-client的机制。每当tmux运行时，都会启动tmux server；
    2. client：而访问session的时候，则对应一个client。同一个session，如果用几个不同的terminal访问，**则对应多个client，因此`tmux detach-client -s mysession -t /dev/ttys004` 表示的是detach某个`/dev/ttys004`这一client，但是这个session其他的client（terminal）还是在运行的**
    > When tmux starts, you are connected to a server via a socket connection. What you see presented in your shell is merely a client connection
3. session：包含windows
4. windows：进入的一个session之后，状态上的每一个都对应一个window，每一个window又可以包含多个pane
5. pane：显示在window中的小窗，一个window可以包含多个pane


## tmux 常用命令：

摘自[The tao of tmux的targets这一节](https://leanpub.com/the-tao-of-tmux/read#targets)

主要有：
- attach session（进入session）：`tmux attach -t `
- detach session(退出session，**这一项很重要因为当同一个session被多个client同时连接时，可能会有显示上的错误**，因此意外被中断ssh重新连接上之后，**最好把所有session都detach掉**) ：`tmux detach -s `
- detach client（与上述detach session对应，通常用于关闭某个session的某个client）：`tmux detach-client -s mysession -t /dev/ttys004`
- list sessions（查看所有session以及对应状态，无法看到一个session被几个client连接）: `tmux ls`
- list clients（可以指定查看所有clients或者某个session对应的clients）:`tmux list-clients [-t target-session]`
- rename session：`tmux rename-session [-t target-session] session-name`

## Reorder window
主要有两种方法：`move-window` 和 `swap-window`

强烈建议用`swap-window`，并且绑定 `Shift Left`和`Shift Right`来移动窗口

参考：[Reorder tmux window](https://unix.stackexchange.com/a/151332)


```bash
bind-key -n S-Left swap-window -t -1
bind-key -n S-Right swap-window -t +1
```



## 开启真彩色解决vim颜色奇怪

参考：

[How to use true colors in vim under tmux?](https://github.com/tmux/tmux/issues/1246)

[为 vim + tmux 开启真彩色(true color)](https://lotabout.me/2018/true-color-for-tmux-and-vim/)

### 设置tmux

```bash
set -g default-terminal screen-256color
set-option -ga terminal-overrides ",*256col*:Tc" # 这句是关键

```



或者

```bash
# ！！！importent！！！ 开启24 bit color 其他方式都无效
set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",*256col*:Tc"
```



### 设置vim

```bash
" Enable true color 启用终端24位色
if exists('+termguicolors')
  let &t_8f = "\<Esc>[38;2;%lu;%lu;%lum"
  let &t_8b = "\<Esc>[48;2;%lu;%lu;%lum"
  set termguicolors
endif
```



## 开启正确的鼠标支持

强推[tmux-better-mouse-mode](https://github.com/NHDaly/tmux-better-mouse-mode)





## 设置nested tmux

### 思路1：设置两个`prefix`



参考 [Nested tmux sessions on local and remote servers](https://stackoverflow.com/a/35276527)

针对到slurm的情况，思路类似：

1. 先打开一个tmux
2. 然后再在tmux里面用`srun`开启一个bash：`srun -w node01 -t 4320 --pty bash`
3. 再在新的bash里面开tmux，同时设置`prefix`为`C-n`（或者其他按键，找一个无冲突且方便的）



### 思路2（推荐）：参考samoshkin的配置（推荐配置）

参考 [nested-session](https://github.com/samoshkin/tmux-config#nested-tmux-sessions)

通过`F12`开关（也是推荐配置之一）