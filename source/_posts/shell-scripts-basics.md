---
title: shell-scripts-basics
typora-root-url: shell-scripts-basics
typora-copy-images-to: shell-scripts-basics
date: 2020-06-17 16:46:34
tags: 
- shell
- linux
categories: linux
---

# 判断输入脚本（函数）的参数个数

[Check number of arguments passed to a Bash script](https://stackoverflow.com/a/18568726)



都是使用`$#`

```bash
if [ "$#" -eq 2 ]
then
fi
```

可以使用 `-eq`, `-ne`, `-lt`, `-le`, `-gt` 判断个数

# logical and / or / not

`||` or , `&&` and, `!` not

用`[[]]`或者`[]`

```bash
HOST=user1
if  [[ $HOST == user1 ]] || [[ $HOST == node* ]] ;
then
    echo yes1
fi

HOST=node001
if [[ $HOST == user1 ]] || [[ $HOST == node* ]] ;
then
    echo yes2
fi
```



更复杂的例子（参考[Check number of arguments passed to a Bash script](https://stackoverflow.com/a/18568726)）：

```bash
[[ ($# -eq 1 || ($# -eq 2 && $2 == <glob pattern>)) && $1 =~ <regex pattern> ]]
```

```bash
A=1
[[ 'A + 1' -eq 2 ]] && echo true  ## Prints true.
```



参考 [Checking for the correct number of arguments](https://stackoverflow.com/a/4341647)

```bash
#!/bin/sh
if [ "$#" -ne 1 ] || ! [ -d "$1" ]; then
  echo "Usage: $0 DIRECTORY" >&2
  exit 1
fi
```



## 判断字符串是否以某子串开始或者结束

[In Bash, how can I check if a string begins with some value?](https://stackoverflow.com/a/2172367)



```bash
# The == comparison operator behaves differently within a double-brackets
# test than within single brackets.

[[ $a == z* ]]   # True if $a starts with a "z" (wildcard matching).
[[ $a == "z*" ]] # True if $a is equal to z* (literal matching).
```

使用`z*`而不要使用`"z*"`，前者才是真正的wildcard matching



## 获取文件夹下的所有文件

[Linux Shell获取文件夹下的文件名](https://blog.csdn.net/Quincuntial/article/details/54348471)

```bash
#!/bin/bash
# get all filename in specified path

path=$1
files=$(ls $path)
for filename in $files
do
   echo $filename >> filename.txt
done
```



## 对array的一些操作

### 遍历array

[Loop through an array of strings in Bash?](https://stackoverflow.com/a/8880633)

elements可以使numbers或者直接strings

```bash
arr=("element1" "element2" "element3")

## now loop through the above array
for i in "${arr[@]}"
do
   echo "$i"
   # or do whatever with individual element of the array
done

# You can access them using echo "${arr[0]}", "${arr[1]}" also
```



同时获得index和元素（类似python的enumerate）

```bash
for index in "${!array[@]}"
do
    echo "$index ${array[index]}"
done
```



### 访问array的元素

[Split string into an array in Bash](https://stackoverflow.com/a/10586169)

根据索引访问：

```bash
echo "${array[0]}"
```

访问最后一个元素：

```bash
echo "${array[-1]}"
```

或者更通用的

```bash
echo "${array[@]: -1:1}"
```



## trap命令

> **trap命令**用于指定在接收到信号后将要采取的动作，常见的用途是**在脚本程序被中断时完成清理工作**。

[trap命令](https://man.linuxde.net/trap)



### trap在退出时清除后台

[Simple Way to Start (and Stop!) Background Processes in Bash](https://spin.atomicobject.com/2017/08/24/start-stop-bash-background-process/)



```bash
trap "exit" INT TERM ERR
trap "echo killing whole process; kill 0" EXIT
```



## shell bash判断文件或文件夹是否存在

[shell bash判断文件或文件夹是否存在](https://www.cnblogs.com/emanlee/p/3583769.html)

> ```bash
> #shell判断文件夹是否存在
> 
> #如果文件夹不存在，创建文件夹
> if [ ! -d "/myfolder" ]; then
>   mkdir /myfolder
> fi
> 
> #shell判断文件,目录是否存在或者具有权限
> 
> 
> folder="/var/www/"
> file="/var/www/log"
> 
> # -x 参数判断 $folder 是否存在并且是否具有可执行权限
> if [ ! -x "$folder"]; then
>   mkdir "$folder"
> fi
> 
> # -d 参数判断 $folder 是否存在
> if [ ! -d "$folder"]; then
>   mkdir "$folder"
> fi
> 
> # -f 参数判断 $file 是否存在
> if [ ! -f "$file" ]; then
>   touch "$file"
> fi
> 
> # -n 判断一个变量是否有值
> if [ ! -n "$var" ]; then
>   echo "$var is empty"
>   exit 0
> fi
> 
> # 判断两个变量是否相等
> if [ "$var1" = "$var2" ]; then
>   echo '$var1 eq $var2'
> else
>   echo '$var1 not eq $var2'
> fi
> ```
>
> 



重点看这个

```bash
# -d 参数判断 $folder 是否存在
if [ ! -d "$folder"]; then
  mkdir "$folder"
fi

# -f 参数判断 $file 是否存在
if [ ! -f "$file" ]; then
  touch "$file"
fi
```



## 



## 提取字符串中的数字

```bash
temp=`echo $filename | tr -cd "[0-9]"` 
```

* `-c`表示取反

*  `-d`表示去除

意思就是去除掉非数字字符（不在`[0-9]`的字符），剩下的自然是数字





## 多进程/多线程

[Linux: shell实现多线程, Forking / Multi-Threaded Processes | Bash, shell 线程池](https://justcode.ikeepstudying.com/2018/02/linux-shell%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%BA%BF%E7%A8%8B-forking-multi-threaded-processes-bash/) 强烈推荐这个，说的非常详细，很ok！

[Shell 脚本实现并发多进程 了解一下~](https://cloud.tencent.com/developer/article/1527555)，这个对应下面的第四种方法

[Shell多进程执行任务](https://segmentfault.com/a/1190000020531589) 详细介绍了前三种方法，特别是管道的含义



主要有四种方式：

1. 循环开多个前台任务，串行的
2. 循环开多个后台任务，由于所有任务都一起放到了后台，因此比较耗费资源
3. 开管道队列，类似python的Queue，多进程
4. parallel命令（TODO，**强烈推荐，需要深入学习**）
5. 用array模拟队列（Deprecated，太复杂没意义）

### 循环开多个前台任务

```bash
#!/bin/bash

start=`date "+%s"`
for i in {1..10}
do
    echo $i;
    sleep 2
done
end=`date "+%s"`
echo "Time: `expr $end - $start`"
```



### 循环直接开多个后台任务

```bash
#!/bin/bash
Njob=15
for ((i=0; i<$Njob; i++)); do          
echo  "progress $i is sleeping for 3 seconds zzz…"          
sleep  3 &       #循环内容放到后台执行
done
wait      #等待循环结束再执行wait后面的内容
echo -e "time-consuming: $SECONDS    seconds"    #显示脚本执行耗时
```



### 开管道队列，类似python的Queue，多进程

就是使用一个文件描述符+管道，模拟一个FIFO的队列

命令的一些含义可以参考 [[Linux]Linux Shell多进程并发以及并发数控制](https://blog.csdn.net/yeweiouyang/article/details/52512522)



1. trap的含义：

> * 第26行：表示在脚本运行过程中，如果接收到`Ctrl+C`中断命令，则关闭文件描述符1000的读写，并正常退出
>   * `exec 1000>&-;`表示关闭文件描述符1000的写
>   * `exec 1000<&-;`表示关闭文件描述符1000的读
>   * trap是捕获中断命令

2. 描述符的含义：

> * 第27～29行：
>   * 第27行，创建一个管道文件
>   * 第28行，将文件描述符1000与FIFO进行绑定，<读的绑定，>写的绑定，<>则标识对文件描述符1000的所有操作等同于对管道文件$tempfifo的操作
>   * 第29行，可能会有这样的疑问：为什么不直接使用管道文件呢？事实上这并非多此一举，管道的一个重要特性，就是读写必须同时存在，缺失某一个操作，另一个操作就是滞留，而第28行的绑定文件描述符（读、写绑定）正好解决了这个问题

3. 其余命令的含义

> * 第31～34行：对文件描述符1000进行写入操作。通过循环写入8个空行，这个8就是我们要定义的后台并发的线程数。为什么是写空行而不是写其它字符？因为管道文件的读取，是以行为单位的
> * 第37～42行：
>   * 第37行，read -u1000的作用就是读取管道中的一行，在这里就是读取一个空行；每次读取管道就会减少一个空行
>   * 第39～41行，注意到第42行结尾的&吗？它表示进程放到linux后台中执行
>   * 第41行，执行完后台任务之后，往文件描述符1000中写入一个空行。这是关键所在了，由于read -u1000每次操作，都会导致管道减少一个空行，当linux后台放入了8个任务之后，由于文件描述符1000没有可读取的空行，将导致read -u1000一直处于等待。



模版：

```bash
trap "exit" INT TERM ERR
trap "echo killing whole process; kill 0" EXIT # 推出时清除所有进程

# 对应的进程数
Nproc=5
mkfifo ./fifo.$$
exec 666<> ./fifo.$$
rm -f ./fifo.$$

trap "exec 666>&-;exec 666<&-;exit 0" 2

for((i=0;i<$Nproc;i++)); do
echo >& 666
done


path=$1
files=$(ls $path)
for filename in $files; do
if [[ $filename == *.pb ]]
then
{
read -u 666
# 循环要执行的放在这
echo >& 666
}&
fi
done
wait
echo -e "time-consuming:$SECONDS seconds"

```





## 在shell scripts中切换conda环境（conda activate）

参考 [Can't execute `conda activate` from bash script](https://github.com/conda/conda/issues/7980#issuecomment-441358406)



1. 通过`conda info | grep -i 'base environment'`查找conda的安装位置
2. 通过以下命令

```bash
source ~/anaconda3/etc/profile.d/conda.sh # replace ~/anaconda3/ with path found above
conda activate my_env
```



比如下面，就可以看到已经切换过来了

```bash
source /cm/shared/apps/anaconda3/etc/profile.d/conda.sh
conda activate multimodal
conda env list
```



## 拼接路径

参考

1. [https://stackoverflow.com/a/30187926](https://stackoverflow.com/a/30187926)
2. [https://stackoverflow.com/a/11226444](https://stackoverflow.com/a/11226444)

有两种方法：

1. 第一种针对字符串
2. 第二种是针对命令嵌套。**因为第二种等价于开一个sub shell执行命令然后返回结果**

```bash
##### 法一：用${}
#!/bin/bash

read -p "Enter a directory: " BASEPATH

SUBFOLD1=${BASEPATH%%/}/subFold1
SUBFOLD2=${BASEPATH%%/}/subFold2

echo "I will create $SUBFOLD1 and $SUBFOLD2"

# mkdir -p $SUBFOLD1
# mkdir -p $SUBFOLD2

###### 法二：用$()
path="$(pwd)/some/path"
```

