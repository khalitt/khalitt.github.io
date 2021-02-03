---
title: docker基础
date: 2019-11-14 09:33:47
tags:
categories:
---

## 在Dockerfile中更改apt（等）的源路径

直接通过`ADD`命令即可。对于apt-get来说，直接新建一个sources.list然后添加进去即可

```bash
ADD sources.list /etc/apt
```

## 软链(symlink)在docker中是基本不可用的
- 对于需要在container中直接使用的文件，用挂载即可`-v`
- 对于在Dockerfile中使用的文件，则需要用`COPY`或者`ADD`使用即可


## Dockerfile中ADD, COPY的路径问题
根据官网的docs，**应该是只接受absolute path或者path relative to WORKDIR**

因此**最好用绝对路劲，如用`/roor/`代替`~`**


## Dockerfile中更改时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```


## 解决docker tty尺寸过小的问题

[Strange terminal behaviour inside docker ](https://github.com/moby/moby/issues/33794#issuecomment-375051884https://github.com/moby/moby/issues/33794)

直接用设定`COLUMNS`和`LINES`环境变量并不好用，反而只是如上的命令最好

```bash
docker exec -it container_name sh -c "stty rows `tput lines` && stty cols `tput cols` && bash"
```

## 在Dockerfile中往.bashrc等配置在文件中写入

1. 通过`sed`命令实现
2. 通过`>>`实现，参考：[注意这个Dockerfile，还有很多 值得参考的](https://stackoverflow.com/a/45626868) 特别要注意 **单引号和双引号的使用**

```bash
echo 'tmux=...' >> /root/.bashrc
```

## 单引号、双引号、反引号的使用

[单引号，双引号，反引号的区别](https://blog.csdn.net/u014636245/article/details/82919144)

1. 单引号：直接输出，不对其中的`$, \`等转义字符以及命令替换符`` ` ``进行解析
2. 双引号：会对字符串其中的`$`以及`\`命令替换符`` ` ``进行解析
3. 反引号：本质等价于`$()`，作用就是把这其中的命令先执行，然后把执行结果返回，替换在原来的地方

综上，对于`sed`命令和`>>`命令写入时 **需要输入单引号的情况，可以按照下面的方法来** （在双引号中`` ' `` 不会解析）


```bash
sed -i "\$a alias nvim='/root/squashfs-root/usr/bin/nvim -u /root/.SpaceVim/init.vim'" /root/.bashrc
```

## Docker中的Pytorch:ERROR: Unexpected bus error encountered in worker. This might be caused by insufficient shared memory (shm)

[PyTorch踩过的坑](https://zhuanlan.zhihu.com/p/59271905)

> 5. ERROR: Unexpected bus error encountered in worker. This might be caused by insufficient shared memory (shm)

>出现这个错误的情况是，在服务器上的docker中运行训练代码时，batch size设置得过大，shared memory不够（因为docker限制了shm）.解决方法是，将Dataloader的num_workers设置为0.
