---
title: seafile docker服务更新SSL证书
typora-root-url: seafile-update-certificate
typora-copy-images-to: seafile-update-certificate
date: 2022-07-06 00:20:03
tags: seafile
categories:
---



首先确认证书是否已经更新。通过v2ray agent脚本可以看到已经成功更新了证书

![image-20220706002421688](/image-20220706002421688.png)



接下来参考 https://bbs.seafile.com/t/topic/13486

![image-20220706002545114](/image-20220706002545114.png)



确认当前的配置

![image-20220706002645133](/image-20220706002645133.png)

![image-20220706002848090](/image-20220706002848090.png)

可以看到当前证书的位置



先把老证书备份

```bash
cd /opt/seafile-data/ssl && mv www.max2sxz.tk.crt www.max2sxz.tk.crt.bak && mv www.max2sxz.tk.key www.max2sxz.tk.key.bak
```



再把新证书复制过来(注意不能创建软连接，必须直接复制！！否则重启nginx服务报错)

![image-20220706003308404](/image-20220706003308404.png)



最后再重启服务

```bash
docker exec -it seafile /usr/sbin/nginx -s reload
```

![image-20220706003400157](/image-20220706003400157.png)



可以看到成功登陆

![image-20220706003519974](/image-20220706003519974.png)



