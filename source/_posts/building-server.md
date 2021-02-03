---
title: CloudAtCost服务器搭建
date: 2019-09-19 15:51:42
tags:
categories:
---



# CentOS6/7

1. ## 首先安装依赖：unzip, zip, gcc, wget, curl

   ```bash
   yum install gcc wget curl
   ```

   ```bash
   yum install zip unzip
   ```

   ```bash
   yum install -y git
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

   

2. ## 安装魔改bbr

   参考[Linux NetSpeed](https://github.com/cx9208/Linux-NetSpeed)

   ```python
   wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
   chmod +x tcp.sh
   ./tcp.sh
   ```

   

3. ## 然后安装v2ray(包括了Shadowsocks)

   参考

   [V2Ray搭建详细图文教程](https://github.com/233boy/v2ray/wiki/V2Ray搭建详细图文教程)

   ```bash
   bash <(curl -s -L https://git.io/v2ray.sh)
   ```

   

4. ## 最后是秋水大神的脚本
    ### 1121 update：还是建议用逗比的脚本比较好（新，更便于管理）
    [『原创』CentOS/Debian/Ubuntu ShadowsocksR 单/多端口 一键管理脚本](https://doubibackup.com/z2a4lk3l.html)

    特别注意原文中所说的，混淆协议的选择（最佳就是默认的plain）
    
    ```bash
    wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubiBackup/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
    ```
    ### 1028 update: 用下面的新的脚本更好：
    
    ```bash
    wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
    chmod +x shadowsocks-all.sh
    ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
    
    ```

    - First Commit：
   先安装GCC

    ```bash
    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
    chmod +x shadowsocksR.sh
    ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
    ```


    参考

    [ShadowsocksR(SSR)一键安装脚本 By 秋水逸冰](http://www.wangchao.info/1549.html)



    ### 卸载

    ```bash
    ./shadowsocksR.sh uninstall
    ```

    ### 查看状态

    ```bash
    /etc/init.d/shadowsocks status
    ```


# CAC debian8 基本设置

## 安装sudo以及build-essential

1. 首先安装sudo


```bash
apt-get update
apt-get install sudo
```

2. 用Nano添加源，更新，最后直接安装build-essential


```bash
sudo apt edit-sources
```

用Nano添加源


```bash
deb http://httpredir.debian.org/debian jessie main contrib
deb-src http://httpredir.debian.org/debian jessie main contrib

deb http://httpredir.debian.org/debian jessie-updates main contrib
deb-src http://httpredir.debian.org/debian jessie-updates main contrib

deb http://security.debian.org/ jessie/updates main contrib
deb-src http://security.debian.org/ jessie/updates main contrib 
```

最后直接用以下命令即可安装build-essential


```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential
```


最后就能愉快地安装bbr了

## CAC一键dd
debian8直接dd debian99失败，但是ubuntu 14 LTS直接dd debian 9 就成功了


## 检查bbr是否开启
From 秋水大神的[一键安装最新内核并开启 BBR 脚本](https://teddysun.com/489.html)

安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。

重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：


```bash
uname -r
```

查看内核版本，显示为最新版就表示 OK 了


```bash
sysctl net.ipv4.tcp_available_congestion_control
```

返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno

或者为：

net.ipv4.tcp_available_congestion_control = reno cubic bbr


```bash
sysctl net.ipv4.tcp_congestion_control
```

返回值一般为：

net.ipv4.tcp_congestion_control = bbr


```bash
sysctl net.core.default_qdisc
```

返回值一般为：
net.core.default_qdisc = fq


```bash
lsmod | grep bbr
```

返回值有 tcp_bbr 模块即说明 bbr 已启动。注意：并不是所有的 VPS 都会有此返回值，若没有也属正常。