---
title: jupyter lab配置教程
date: 2019-09-21 11:05:43
tags: jupyter_lab
categories: 
- python
- jupyter

---

参考：

[服务器集成anaconda安装jupyter-notebook]([https://543877815.github.io/2019/04/10/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%9B%86%E6%88%90anaconda%E5%AE%89%E8%A3%85jupyter-notebook/](https://543877815.github.io/2019/04/10/服务器集成anaconda安装jupyter-notebook/))

[JUPYTERLAB 的安装与配置](https://lyric.im/c/the-craft-of-selfteaching/T-appendix.jupyter-installation-and-setup)

## 安装

直接通过conda安装即可

```python
conda install jupyter-lab
```



## 配置

### 1. 先生成配置文件

```bash
jupyter lab --generate-config
```

### 2. 修改password

在本地/home/xxx/.jupyter/jupyter_notebook_config.py，打开这个py文件

```python
c.NotebookApp.ip = '*' # 允许访问此服务器的 IP，星号表示任意 IP
c.NotebookApp.password = u'sha1:xxx:xxx' # 之前生成的密码 hash 字串
c.NotebookApp.open_browser = False # 运行时不打开本机浏览器
c.NotebookApp.port = 12035 # 使用的端口，随意设置
c.NotebookApp.enable_mathjax = True # 启用 MathJax
c.NotebookApp.allow_remote_access = True # 允许远程访问
```

### 3. 安装插件

```python
conda install -c conda-forge jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
jupyter nbextensions_configurator enable --user
```



### 4. 配置中文

[](https://blog.csdn.net/github_33934628/article/details/77874674)

## 使用

1. ### 配置mobaxterm的ssh tunnel转发

   Tools - MobaSSHTuneel(port forwarding) - New SSH Tunnel


在上述地方中：

- Remote Server, SSH server都是服务器的地址
- Remote Port指的是Jupyter lab的端口，即上面中填入的端口
- SSH port指的是登陆服务器要用的端口，一般是22
- Forward Port指的是映射到本地的端口，随便填



### 使用extensions

直接使用很可能出现 jupyter build failed的错误，很多时候是因为nodejs和npm的安装问题。因此需要先通过conda安装这两个

```bash
conda install -c conda-forge nodejs
conda install -c tejzpr npm
```

### 修改字体

参考

[jupyter notebook中显示字体如何调整？](https://www.zhihu.com/question/40012144)

直接安装stylus，然后在里面找jupyter lab的主题

### 修改matplotlib图片大小

```python
%matplotlib inline
import matplotlib.pyplot as plt
plt.rcParams["figure.figsize"] = (6, 4)
plt.rcParams['figure.dpi'] = 300
```



### 本地访问远程的jupyter

把远程的8888映射到本地的1234

```bash
ssh -N -f -L localhost:1234:localhost:8888 tangweitao.walter@devbox
```



### 让conda环境在notebook中被发现

[ipython kernel](https://ipython.readthedocs.io/en/stable/install/kernel_install.html)

```python
pip install ipykernal
python -m ipykernel install --user --name myenv --display-name "Python (myenv)"
```



