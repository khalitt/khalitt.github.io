---
title: spacevim的基本使用
date: 2020-02-25 12:48:06
tags: spacevim
categories: vim
---

## Windows
### 安装
windows下强烈推荐用neovim-qt, 像gvim用起来有各种bug. 不建议.

而且neovim直接下载下来后, 可以直接打开, 记得把解压的文件夹加入到Path即可. 

具体的安装步骤可以参考 [Hack-Spacevim](https://github.com/Gabirel/Hack-SpaceVim/blob/master/zh_CN/installation/installation-for-windows.md)

安装中具体要注意的的点在上述的 [Hack-Spacevim](https://github.com/Gabirel/Hack-SpaceVim/blob/master/zh_CN/installation/installation-for-windows.md)
中已经说得很清楚了, 为了正常使用还需要添加一些别的插件, 比如在这里就加入了
[im-select](#im-select)

windows下使用官网的`install.cmd`完成下载之后, 直接运行, 再次打开nvim-qt
即可直接进行安装. 但是注意到可能会缺少一些dll文件等等导致失败的, 可以
通过上述[Hack-Spacevim](https://github.com/Gabirel/Hack-SpaceVim/blob/master/zh_CN/installation/installation-for-windows.md)
进行下载, 配置等.


### 基本配置
windows下, 配置文件都在`%User\.SpaceVim.d\init.toml`中, 具体设定参照官网文档即可

但是如果需要额外的配置, 就要需要`bootstrap`函数来实现上述的功能了.
具体的设定可以参照[`bootstrap`函数](#bootstrap函数)

## 常用额外设定
### markdown
如官网所示加入markdown层即可, 会自动安装一系列的插件. 

但是要注意, 其中部分的插件可能一直显示`Building`即无法编译成功,
这时候就需要手动用yarn进行安装了.

具体的方法是:

1. 先安装好npm和yarn
2. 把`%User\.cache\vimfiles\repos\github\iamcco\markdown-preview.nvim`
这一文件夹删除; 然后重新打开nvim-qt, 就会重新下载这一文件夹.
3. 卡在`Building`阶段, 此时退出文件夹, 进入到`%User\.cache\vimfiles\repos\github.com\iamcco\markdown-preview.nvim\app`
然后在此处打开powershell或者cmd, 直接yarn install, 即可安装成功.

### latex
直接添加`latax`层即可.

需要注意:

1. 需要单独配置好编译`latax`的环境, 像此处可以使用Miktex + Sumatra pdf的组合.
2. vim默认使用的是latexmk进行编译, 而这个功能还需要安装perl. 因此还需要单独安装perl. 
3. 为了实现inverse search即在PDF上双击对应位置返回neovim中对应的tex文档, 还需要:
    - 需要在python3环境中安装remote-neovim
    - Sumatra pdf中进行参数设定, 直接在选项中设定即可. 但是这里也有个[坑](http://)


## 常用额外插件
### vim-im-select
很好用的插件, 主要针对的是使用vim的过程中需要中
中英文来回切换的场景. 

注意事项:

1. 需要提前设定好的选项, 主要有两项, `im-select.exe`所在的位置和默认的
输入法,输入法的话可以运行`im-select.exe`来确定(一般是数字):
  ```bash
    let g:im_select_command = 'E:/download/im-select.exe' 
    let g:im_select_default = 1033
  ```


## 遇到的问题
### neovim中的python
neovim中通常需要python2,3的支持, 虽然可以自动检测当前`Path`中所对应的python路径,
但是比较好的方法仍然是通过将对应的变量名设定为路径即可.
具体的做法如[`bootstrap`函数](#bootstrap函数)中所示.

需要注意的是, **两个python环境均需要安装neovim和pynvim这两个包. 可以直接通过pip安装**


### `bootstrap`函数
1. 放的位置要注意,不是`%\User\.SpaceVim\autoload`(即SpaceVim默认的文件夹),
而是`%\User\.SpaceVim.d\autoload`!!, **而这一文件夹通常默认是不存在的,
因此需要单独创建** 
 

2. 在上述的`autoload`文件夹中放入对应的`.vim`文件, 
通常文件名和函数明对应即可, 如按照官网的基础设定,
可以定义`myspacevim.vim`如下:

```vim
function! myspacevim#before() abort
  set mouse=a
  "For python3 and python2, respectively
  let g:python3_host_prog = 'C:/Users/twt96/Python37/python.exe' 
  let g:python_host_prog = 'C:/Users/twt96/Python27/python2.exe'

  let g:tagbar_ctags_bin = 'F:/ctags/ctags.exe'
  " set pythonthreedll=C:/Users/twt96/Python37/python3.dll
  " set pythonhome=C:/Users/twt96/Python27
  inoremap <silent> <S-Insert> <C-R>+
  vnoremap <silent> <RightMouse> "+y
  vnoremap <silent> <C-C> "+y

  " Copy to clipboard
  " vnoremap  <leader>y  "+y
  " nnoremap  <leader>Y  "+yg_
  " nnoremap  <leader>y  "+y
  " nnoremap  <leader>yy  "+yy
"
  " Paste from clipboard
  " nnoremap <leader>p "+p
  " nnoremap <leader>P "+P
  " vnoremap <leader>p "+p
  " vnoremap <leader>P "+P
  
  " Below are for 
  let g:vimtex_view_general_viewer = 'SumatraPDF'
  let g:vimtex_view_general_options_latexmk = '-reuse-instance'
  " let g:vimtex_compiler_progname = 'nvr'
  let g:vimtex_view_general_options
      \ = '-reuse-instance -forward-search @tex @line @pdf'
  let g:vimfiler_ignore_pattern = []

  " change input method automatically
  let g:im_select_command = 'E:/download/im-select.exe'
  let g:im_select_default = 1033

endfunction
```

其中`before`表示在加载其他文件前会载入的函数, 对应的也也可以在
这个文件里写`after`对应的函数.



### Sumatra PDF中inverse search的设定
vimtex帮助中有提到这一点, 但是给定的参数是:

```vim
nvr --remote +"%line" "%file"
```

其实不正确, 这样会导致返回的文件是`.texile`编译后的文件而非`.tex`

正确的做法应该是实际在[vimtex的docs](https://github.com/lervag/vimtex/blob/master/doc/vimtex.txt)
所说的:

```vim
nvr --remote-silent +%l "%f"
```



