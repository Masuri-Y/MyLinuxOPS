### echo命令

回显，将命令后跟随的字符串打印到终端

```bash
[root@mylinuxops ~]# echo hello world
hello world
```

echo选项：

* -E 
* -n 不自动换行
* -e 启用\字符的解释功能



> ##### -e参数使用

-e参数可以将字符串中的\字符进行转义，如\n进行换行

```bash
masuri@XPS-13-9380:~$ echo -e "hello\nworld"
hello
world
masuri@XPS-13-9380:~$ echo "hello\nworld"
hello\nworld
```

启用命令选项-e，若字符串中出现以下字符，则特别加以处理，而不会将它当作一般文字输出

* `\a` 	发出警告
* `\b` 	退格
* `\c`	最后不加上换行符号
* `\e escape`	相当于\033
* `\n`	换行且光标移至行首
* `\r`	回车，即光标移至行首，但不换行
* `\t`	插入tab
* `\\`	插入\字符
* `\0nnn`	插入nnn(八进制)所代表的ASCII字符
* `\xHH`	插入HH(十六进制)所代表的ASCII字符


```bash
[root@mylinuxops ~]# echo -e '\033[43;31;5mmylinuxops\e[0m'
mylinuxops
```

#### 命令行扩展

把一个命令的输出答应给另外一个命令的参数，可以使用`$()`或者``，来实现。

```bash
[root@mylinuxops ~]# echo "echo $PS1"
echo [\u@\h \W]\$ 
[root@mylinuxops ~]# echo 'echo $PS1'
echo $PS1
[root@mylinuxops ~]# echo `echo $PS1`
[\u@\h \W]\$
[root@mylinuxops ~]# echo $(echo $PS1)
[\u@\h \W]\$
```

##### 反向单引号的使用

> 生成一个当前日期的.log文件

```bash
[root@mylinuxops ~]# touch `date +%F`.log
[root@mylinuxops ~]# ls
2021-03-03.log 
```

#### 括号扩展

`{}`括号扩展可以用来打印重复字符串的简化形式

> 括号的使用方法

```bash
[root@mylinuxops ~]# touch file{1,2,3}
[root@mylinuxops ~]# ls
2021-03-03.log  anaconda-ks.cfg  file1  file2  file3  original-ks.cfg
[root@mylinuxops ~]# rm -f file{1,2,3}
[root@mylinuxops ~]# ls
2021-03-03.log  anaconda-ks.cfg  original-ks.cfg
[root@mylinuxops ~]# echo {1..10}
1 2 3 4 5 6 7 8 9 10
[root@mylinuxops ~]# echo {a..d}
a b c d
[root@mylinuxops ~]# echo {Z..a}
Z [  ] ^ _ ` a
[root@mylinuxops ~]# echo {a..z..2}
a c e g i k m o q s u w y
[root@mylinuxops ~]# echo {1..10..2}
1 3 5 7 9
[root@mylinuxops ~]# echo {a,m,z}.{test,log}
a.test a.log m.test m.log z.test z.log
```

