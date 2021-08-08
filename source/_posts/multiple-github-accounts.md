---
title: github多账号设置
typora-root-url: multiple-github-accounts
typora-copy-images-to: multiple-github-accounts
date: 2021-08-07 21:11:04
tags: hexo
categories: hexo
---

参考：

1. [GitHub 多账户设置](https://segmentfault.com/a/1190000022797854)
2. [**github_multiple-accounts.md**](https://gist.github.com/JoaquimLey/e6049a12c8fd2923611802384cd2fb4a#file-github_multiple-accounts-md)
3. [github 多账号使用ssh key](https://www.huaweicloud.com/articles/5e1b8d3cb0b673fae499eaa34a94205c.html)



适用场景：希望在同一台机器上使用多个github账号时







# 步骤



核心步骤：

1. 创建一个新的SSH key（注意路径）并添加进ssh
2. 在新的账号上添加新创建的public key
3. 在本地添加一个config文件以作区别
4. enjoy！

## 创建一个新的ssh key并添加

```bash
ssh-keygen -t rsa -C "xxx email"
```



注意这里不能一路回车，在下面这步需要输入新的位置，因为通常默认的是id_rsa，而这个已经被原来的账号所使用

![image-20210808103624410](/image-20210808103624410.png)





因为默认只读取id_rsa，为了让SSH识别新的私钥，需将其添加到SSH agent中，比如这里就把上面的key命名为id_rsa_wttong

![image-20210808103853450](/image-20210808103853450.png)





## 在新的账号上添加新创建的public key

进入github setting - SSH key相关的即可添加



## 在本地添加config文件

通常来说是默认是没有config文件的，所以要先创建，然后打开编辑

```bash
touch ~/.ssh/config
vim ~/.ssh/config
```



具体的内容可参考：

```bash
Host github.com # 默认的账号
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa

Host github.com-wttong # 新添加的账号
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_wttong # 特别注意这里

```



核心就是把两个identityFile区分开来



最后检查两个是否都正常使用

![image-20210808104409486](/image-20210808104409486.png)



# 具体使用的要注意的点



1. 在clone等时候，如果使用ssh的方式，需要注意更改的地方：

比如原本是：

```bash
git clone git@github.com:xxxx
```

需要改为

```bash
git clone git@github.com-wttong:xxxx
```

其他诸如push等类似

2. 对于已有的repo，需要注意config中是否需要修改，可以通过 `git config --list`查看，比如下面这种，看看是否需要和上面一样修改github.com

   ![image-20210808104727256](/image-20210808104727256.png)

