---
title: hexo algolia 使用
typora-root-url: hexo-algolia-usage
typora-copy-images-to: hexo-algolia-usage
date: 2021-10-16 12:26:33
tags:
categories:
---

参考

1. https://blog.ccknbc.cc/posts/hexo-butterfly-algolia/ 相当详细，介绍了仅仅搜索标题了搜索全文的两种方法
2. [Hugo博客搭配Algolia实现站内搜索](https://raycoder.me/p/hugo-search-with-algolia/)，介绍了全文搜索的情况下自动更新



# hexo-algolia和hexo-algoliasearch的区别



> 分别是 [hexo-algolia](https://github.com/oncletom/hexo-algolia) 和 [hexo-algoliasearch](https://github.com/LouisBarranqueiro/hexo-algoliasearch)，他们的介绍分别为
>
> > Index your hexo website content to Algolia Search.
> > 🔎 A plugin to index posts of your Hexo blog on Algolia
>
> 也就很明显了，**如果你想要全站搜索可选择前者，如果你只想搜索文章两者兼可**。**但前者不能将文章内容作为索引上传**，后者可全文上传。
> 然后就是 HEXO 配置文件中添加以下内容，下文基本以 `hexo-algoliasearch` 为例，因为我个人认为访客只会搜文章吧（事实上是搜索根本没人用，毕竟也根本没人访问），hexo-algolia 可查看官方文档，注意配置和命令的区别







# 仅仅搜索标题

直接用hexo-algolia即可



然后修改目录下的`_config.butterfly.yml.yml`即可

```yaml
algolia_search:
  enable: true
  hits:
    per_page: 6
```



最后再按照 [hexo-algolia](https://github.com/oncletom/hexo-algolia) 配置一下`_config.yml`中关于Algolia的选项

![image-20211016124101223](/image-20211016124101223.png)





# 搜索全文（文章内容）



基本按照这篇文章来就可以了https://blog.ccknbc.cc/posts/hexo-butterfly-algolia/
