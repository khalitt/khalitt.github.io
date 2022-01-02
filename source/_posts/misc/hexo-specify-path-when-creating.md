---
title: hexo new 时的路径创建
typora-root-url: hexo-specify-path-when-creating
typora-copy-images-to: hexo-specify-path-when-creating
date: 2021-10-16 12:15:37
tags: hexo
categories:
---



基本用法如下

```bash
hexo new layout [article_name] -p [path]
```



**注意path末尾不用加`.md`**



# `hexo new layout [article_name] -p [article_name]`

这种默认是在`source/_posts`下创建

比如

```bash
hexo new article-img tttt -p tttt
```

![image-20211016122015940](/image-20211016122015940.png)





# `hexo new layout [article_name] -p subdir/[article_name]`

这种等价于在`source/_post/subdir`创建，如果`subdir`不存在则会自动创建

```
hexo new article-img tttt -p misc_1/tttt
```

![image-20211016122510969](/image-20211016122510969.png)
