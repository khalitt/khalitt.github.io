---
title: screen使用
typora-root-url: screen-usage
typora-copy-images-to: screen-usage
date: 2021-02-04 12:10:00
tags:
- screen
- linux
categories:
- linux
---



# 修改`srceenrc`后不重启screen生效

来自 [How to reload screenrc without restarting screen?](https://serverfault.com/questions/194597/how-to-reload-screenrc-without-restarting-screen)

`Ctrl-a :source ~/.screenrc`.



# 开启自动刷新（`altscreen`）

具体原理说明参考：

1. [GNU screen clearing on vim,less,etc. exit](https://serverfault.com/a/270169)
2. [Using screen, commands like less and man don't clear the screen afterwards](https://superuser.com/a/137751)，推荐，说的非常详细



这是非常必要的设置，默认应该是关闭，则在使用vim等应用之后，上一页会残留，比如下面这样：

![image-20210204121301491](/image-20210204121301491.png)



开启`altscreen on`之后，编辑完直接退出，不会有残留

![image-20210204121411454](/image-20210204121411454.png)



