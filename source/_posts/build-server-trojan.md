---
title: trojan搭建
typora-root-url: build-server-trojan
typora-copy-images-to: build-server-trojan
date: 2020-06-14 17:44:07
tags: 
- trojan
- GFW
categories: trojan
---

主要参考

[atrandys大神的github，切换到master分支](https://github.com/atrandys/trojan/tree/master)



## 配置域名(freenom和cloudfare相关的设置)

[自建梯子教程 --Trojan版本](https://trojan-tutor.github.io/2019/04/10/p41.html)

主要配置freenom即可，注意**trojan不应该设置cloudfare**：

![image-20200615000851804](/image-20200615000851804.png)





## centos安装依赖：unzip, zip, gcc, wget, curl

```bash
yum install curl wget
```



### 还要注意安装yum utils

参考 [yum中途中断]((yum中途中断)There are unfinished transactions remaining. You might consider running yum-complete-tra)

```bash
yum install yum-utils
```

```
yum clean all
```

```bash
yum-complete-transaction –cleanup-only
```

## 安装魔改bbr（如果是cloudcone的centos7.5版本可以跳过此步，这个rom默认开启bbr）

参考[Linux NetSpeed](https://github.com/cx9208/Linux-NetSpeed)

```bash
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```





## 一键安装

### atrandys大神的一键安装

具体的脚本可以**去github切换至origin分支查看**，下面这个是支持多系统的

```bash
curl -O https://raw.githubusercontent.com/atrandys/trojan/master/trojan_mult.sh && chmod +x trojan_mult.sh && ./trojan_mult.sh
```



### Jrohy大神的一键安装+网页前端管理用户

```bash
#安装/更新
source <(curl -sL https://git.io/trojan-install)

#卸载
source <(curl -sL https://git.io/trojan-install) --remove
```

