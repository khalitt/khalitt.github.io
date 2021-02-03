---
title: Hexo+Github搭建blog笔记（踩过的坑）
date: 2019-09-09 21:29:56
typora-root-url: hexo-blog-notes
typora-copy-images-to: hexo-blog-notes
tags: 
- hexo
- github
catogories: 
- blog
---
# Hexo

部署参考：

[使用Hexo + GitHub Pages搭建个人博客详解][0]

[0]: https://www.crazypudding.com/2016/10/01/Building-my-blog-by-Hexo-and-GitHubPages/	"使用Hexo + GitHub Pages搭建个人博客详解"
[1]: https://juejin.im/post/5c2e22fcf265da615d72c596	"submodule"





## 大坑1：github pages（即默认的blog）只支持master分支

默认是这么说的：

![](https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20190909220233.png)



真的是大坑啊！！然而github pages默认**只能是master分支的**（可能是因为个人用户的关系？？）



![](https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20190909220407.png)



**正确做法**

1. 把_config.yaml中的deploy部分的branch改成**master**。

2. 新建一个branch，然后本地add remote到那个branch，用来备份所有文件（包括主题什么的）。具体而言，就是：
	- 如果远程没有branch:(默认remote叫origin 分支名叫publish)
	    ```bash
	    git push --set-upstream origin publish
	    git add .
	    git commit -am 'test'
	    git push
	    ```
	- 如果已经有branch了（为空）
	    ```bash
	    git branch –-set-upstream-to origin/publish
	    git add .
	    git push
	    ```
	

## 大坑2：theme是作为git submodule还是普通的folder好？

现阶段暂时是作为git submodule，不知道同步会不会出问题。



暂无问题。



参考:

[在 hexo 中使用 git submodules 管理主题][1]



## hexo常用命令

参考：

[hexo常用命令](https://segmentfault.com/a/1190000002632530)

### 简写

`hexo n "我的博客"` == `hexo new "我的博客"` #新建文章
`hexo p` == `hexo publish`
`hexo g` == `hexo generate`#生成
`hexo s` == `hexo server` #启动服务预览
`hexo d` == `hexo deploy`#部署



### 写作

Hexo 有三种默认布局：`post`、`page` 和 `draft`。



第一个是用的最多的，直接hexo new即可。

第二个就是新建一个网页的页面，对应的会生成一个对应名字的文件夹。如`hexo new page test`会生成一个/source/test文件夹，而且其中包含一个index.md。

第三个：这种布局在建立时会被保存到 `source/_drafts` 文件夹，您可通过 `publish` 命令将草稿移动到 `source/_posts` 文件夹，该命令的使用方式与 `new` 十分类似，您也可在命令中指定 `layout` 来指定布局。

```bash
hexo publish [layout] <title>
```



草稿默认不会显示在页面中，您可在执行时加上 `--draft` 参数，或是把 `render_drafts` 参数设为 `true` 来预览草稿。



---

还可以用模板（scaffold）





## latex公式支持

参考：

[如何在Hexo中渲染公式]([https://thea-r.github.io/2018/07/01/%E5%A6%82%E4%BD%95%E5%9C%A8Hexo%E4%B8%AD%E6%B8%B2%E6%9F%93%E5%85%AC%E5%BC%8F/](https://thea-r.github.io/2018/07/01/如何在Hexo中渲染公式/))

总体就是替换render 从marked为kramed，以及取出math 替换为 mathjax



## 对于含有shell脚本的markdown文件



[hexo title中有特殊字符报错](https://cloud.tencent.com/developer/article/1515142)

由于&#123 &#125 &#35这三个符号都是markdown的保留字，因此每次出现这些符号的时候，要不放到代码块中，**要不然直接输入对应的HTML字符**



> ```shell
> ! &#33; — 惊叹号 Exclamation mark
> " &#34; &quot; — 双引号 Quotation mark
> # &#35; — 数字标志 Number sign
> $ &#36; — 美元标志 Dollar sign
> % &#37; — 百分号 Percent sign
> & &#38; &amp; — 与符号(&) Ampersand
> ' &#39; — 单引号 Apostrophe
> ( &#40; — 小括号左边部分 Left parenthesis
> ) &#41; — 小括号右边部分 Right parenthesis
> * &#42; — 星号 Asterisk
> + &#43; — 加号 Plus sign
> < &#60; &lt; 小于号 Less than
> = &#61; — 等于符号 Equals sign
> - &#45; &minus; — 减号
> > &#62; &gt; — 大于号 Greater than
> ? &#63; — 问号 Question mark
> @ &#64; — Commercial at
> [ &#91; — 中括号左边部分 Left square bracket
> \ &#92; — 反斜杠 Reverse solidus (backslash)
> ] &#93; — 中括号右边部分 Right square bracket
> { &#123; — 大括号左边部分 Left curly brace
> | &#124; — 竖线Vertical bar
> } &#125; — 大括号右边部分 Right curly brace
> 空格 &nbsp;
> ```
>
> 

## hexo插入图片

* 这一篇很详细 [Hexo开启post_asset_folder后, 安装hexo-asset-image,不起作用的问题](https://github.com/ssttm169/hexo_blog)

* 这一篇主要可以参考其中typora结合的方式 [hexo与typora结合](https://www.jianshu.com/p/81a40a2c6514)

* 详细的三种方式的介绍：[Hexo中图片处理正确姿势](https://merrier.wang/20190111/image-skills-in-hexo.html)
* 这篇也很详细 [在Hexo博客中插入图片的各种方式](https://fuhailin.github.io/Hexo-images/)

注意

1. 安装hexo asset image时候要指定github，才能安装到最新的版本（否则结尾有io）

```shell
npm install https://github.com/xcodebuild/hexo-asset-image --save
```



### 删除带图片资源的文章

直接加多一个`*`wildcard即可，就能把文章和资源文件夹一起删除了

```shell
# rm -rf source/_post/tilte*.md
rm -rf source/_post/pyspark-basics*
```



### 20210107 update

最佳姿势：**typora + hexo-asset-image**组合使用（hexo-asset-image需要按照上面的参考链接安装并配置好）

参考`scaffold/article-img`模版，注意配置`typora-root-url`和`typora-copy-images-to`

```yml
title: {{ title }}
date: {{ date }}
tags:
categories:
typora-root-url: {{ title }}
typora-copy-images-to: {{ title }}
```





## hexo deploy时不断弹出github登陆窗口

参考 [fatal: could not read Username for 'https://github.com': Invalid argument](https://github.com/hexojs/hexo-deployer-git/issues/71#issuecomment-382253106)



像下面这种情况：

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20210107110409.png" align=center/>



1. 首先检查是否争取配置了SSH`ssh -T git@github.com`
2. 没问题的话，如果依然出现上述问题，则按照上面的参考链接，从https方式换成SSH方式，可能话再把github token也加上

```yml
deploy:
  type: git
  repo: git@github.com:khalitt/khalitt.github.io.git
  branch: master
  token: 
  message: 
```



## 在`_post`中设立次级目录

参考 [Hexo - how can you set page path?](https://stackoverflow.com/a/55525531)



通过设置`-p`参数即可，**注意要带上`title`，否则会用默认的post格式，然后把原本layout的参数作为title**



```bash
# Correct
hexo new article-img ipython-hanging -p problems-record/ipython-hanging

# Wrong, without explicit title article-img will be regared as title with default layout
hexo new article-img -p problems-record/ipython-hanging
```



## 更换butterfly 主题(或者其他主题)后始终渲染错误

![image-20210203213700457](/image-20210203213700457.png)



尝试在本地开启服务，则无法开启，报错：

![image-20210203213726114](/image-20210203213726114.png)



参考 [大佬能看下这是啥错吗？hexo版本5.1.1，hexo-cli版本4.2.0 ](https://github.com/jerryc127/hexo-theme-butterfly/issues/363#issuecomment-691639082)



删除的[方法](https://segmentfault.com/q/1010000002972327)如下：

```bash
npm install rimraf -g
rimraf node_modules
```



重装即可

```bash
npm i
```

