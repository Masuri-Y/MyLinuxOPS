### 重定向到多个目标(tee)

在使用重定向或管道时，一旦重定向到文件或管道发送到下一个命令时，终端将不再显示。有时我们希望即能够在屏幕上输出又能将其重定向行或管道，这时候就需要使用`tee`命令来实现。

#### `tee`命令

```bash
cmd1 | tee [-a] FILENAME | cmd2
```

把`cmd1`的`stdout`保存到文件中，做为`cmd2`的输入

* `-a` 追加（tee命令保存到文件时默认为覆盖，如果需要追加内容使用`-a`参数）



#### tee使用

> 将a-d保存到文件，并输出到终端

```bash
[root@mylinuxops data]# echo {a..d} | tee f1.txt
a b c d
[root@mylinuxops data]# cat f1.txt 
a b c d
```

> 将主机名追加到f1.txt，并输出到终端

```bash
[root@mylinuxops data]# hostname | tee -a f1.txt 
mylinuxops.com
[root@mylinuxops data]# cat f1.txt 
a b c d				# 原来的a b c d 依旧被保存
mylinuxops.com
```

>将主机名追加到f1.txt，并输出到终端，并作大小写转换

```bash
[root@mylinuxops data]# hostname | tee -a f1.txt | tr 'a-z' 'A-Z'
MYLINUXOPS.COM
[root@mylinuxops data]# cat f1.txt 
a b c d
mylinuxops.com
mylinuxops.com
# 这里需要主机f1.txt中存放的依旧为小写，应为其没有转换。
```



