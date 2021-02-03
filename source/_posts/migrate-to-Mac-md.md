---
title: hexo博客迁移到另一台电脑
date: 2020-04-21 23:50:27
tags:
- Mac
- hexo
categories: hexo
---

参考[hexo博客迁移到另一台电脑](https://wungjyan.github.io/2018/08/17/move-hexo/)

主要步骤：

1. 原来的电脑备份（用git push到github）

2. 在新的电脑上安装hexo

   ```bash
   npm install -g hexo
   ```

   

3. 在新的电脑（此处为MacOS）git clone， 此时应该两个branch（master和publish，分别对应网页的数据和markdown file等源数据）都有

4. 进入目录，切换到publish这一branch

   ```bash
   cd ...
   git checkoout publish
   ```

5. 依次安装：

   ```bash
   npm install
   npm install hexo-deployer-git --save
   npm install hexo-generator-feed --save
   npm install hexo-generator-sitemap --save
   
   # 额外的包辅助图片上传的包
   npm install https://github.com/xcodebuild/hexo-asset-image --save
   ```

   

6. 检查是否成功：

   ```bash
   # hexo clean && hexo g OR hexo clean; hexo g
   hexo clean
   hexo g
   
   ```
