## 文本三剑客之`AWK`

`awk`是一个优良的文本处理工具，是Linux中文本三剑客之一，`awk`的名字取自于其创始人`Alfred Aho`、`Peter Weinberger`和`Brain Kernighan`三人姓式的首字母。

`awk`的功能及其强大，可以进行式样装入、流控制、数学运算符、进程控制语句甚至内置的变量和函数，他具备了一个完整语言所应有的几乎所有特性。

### `awk`语法

`awk`程序由BEGIN语句块、能够使用模式匹配的通用语句块、END语句块，共3部分组成

```bash
awk [option] '[BEGIN{action;...}]pattern{action;...}[END{action;...}]' file
```
|选项|说明|
|:-|:-|
|-F|指定分隔符|
|-f|指定`awk`程序文件|
|-v|变量赋值，每个变量都需要使用-v var=value来赋值|
### `awk`工作原理

1.执行`BEGIN{action;...}`语句块中的语句`BEGIN`语句块在`awk`开始从输入流中读取的行之前被执行，这是个可选的语句块。

2.从文件或标准输入中读取一行，然后执行`pattern{action;...}`语句块，它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。

3.当读至输入流末尾时，执行`END{action;...}`语句块。END语句块，在`awk`从输入流种读取完所有的行之后再执行，比如打印所有行的分析结果这里信息汇总都是在END语句中完成，它也是个可选语句块

### `awk`基本用法

`awk`最简单的使用方法为:

```bash
awk [option] 'pattern{action;...}' file
```
#### 一、`print`

`print`的参数可以是变量、数值或字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起，无法区分。这里逗号的作用与输出文件的分隔符所用是一样的，只是后者是空格而已。当`pattern`不指定时默认打印文件中所有的行。  

示例1：打印`/etc/fstab`中的每一行

```bash
[root@centos7 ~]# awk '{print $0}' /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Mar  5 21:07:19 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=45490aa4-cf29-420d-a606-af32688b6707 /                       xfs     defaults        0 0
UUID=15dcd896-b7cf-48d0-b8bd-4c0b0f2c62b2 /boot                   xfs     defaults        0 0
UUID=4b6e1813-2c46-402a-869a-02cbbcb76ade /data                   xfs     defaults        0 0
UUID=0995b444-48c1-4423-92bc-2deda0d3c082 swap                    swap    defaults        0 0
```
#### 二、`awk`变量

`awk`有内置变量和自定义变量  

**内置变量有`FS`、`OFS`、`RS`、`ORS`、`NF`、`FNR`、`FILENAME`、`ARGC`、`ARGV`，使用时需要启用`-v`选项**

##### 1.`FS`:设置输入域分隔符，等价于命令行选项-F
示例1：使用变量`FS`指定分隔符，取出`/etc/fstab`的第1第3列
```bash
[root@centos7 ~]# awk -v FS=: '{print $1,$3}' /etc/passwd
root 0
bin 1
daemon 2
...以下省略...
```
示例2：使用选项-F指定分隔符，取出`/etc/fstab`的第1第3列
```bash
[root@centos7 ~]# awk -F":" '{print $1,$3}' /etc/passwd
root 0
bin 1
daemon 2
...以下省略...
```
示例3：用正则表达式当分隔符取出分区利用率
```bash
[root@centos7 ~]# df | awk -F"[[:space:]]+|%" '{print $5}'
Use
15
0
0
2
0
1
17
```
示例4：变量在action中也能被引用,在第一个域和第3个域之间加:号
```bash
[root@centos7 ~]# awk -v FS=: '{print $1,FS,$3}' /etc/passwd
root : 0
bin : 1
daemon : 2
...以下省略...
```
##### 2.`OFS`:设置输出时域分隔符，默认为空白
示例1：以:为分隔符分隔字段，输出时以+++为分隔符，取出/etc/passwd第1第2字段
```bash
[root@centos7 ~]# awk -v FS=: -v OFS=+++ '{print $1,$2}' /etc/passwd
root+++x
bin+++x
daemon+++x
...以下省略...
```
##### 3.`RS`:设置记录分隔符
当记录的分割符指定为某符号时，从字符开始至记录分隔符的字符串为一条记录，记录的条数和行数无关。  

示例1：创建一个文本指定;为分隔符查看记录情况。

```bash
[root@centos7 ~]# cat a.txt
a,b,c;1,2,3,4,;aa,bb,cc
zz,yy,xxx;122,444,2322;AA
BB,CC
```
以上面文件内容为例，以分号为记录的分隔符，此文件一共有5条记录。
```bash
[root@centos7 ~]# awk -v RS=";" '{print $0}' a.txt
a,b,c
1,2,3,4,
aa,bb,cc            #此行与下一行为一条记录中间隐藏了一个换行符
zz,yy,xxx
122,444,2322
AA                  #此行与下一行为一条记录中间隐藏了一个换行符
BB,CC
```
##### 4.`ORS`:设置输出时记录分隔符
ORS变量指定输出时的记录分隔符为什么符号。

示例：

依旧为`a.tx`t文件，设置分隔符为,号，纪录分隔符为;号，输出时设置记录分分隔符为___，取出第一字段。

```bash
[root@centos7 ~]# cat a.txt
a,b,c;1,2,3,4,;aa,bb,cc
zz,yy,xxx;122,444,2322;AA
BB,CC

[root@centos7 ~]# awk -F, -v RS=";" -v ORS="___" '{print $1}' a.txt
a___1___aa___122___AA       
BB___
```
##### 5.`NF`:域数
NF可以查看一条记录内有多少个域,也可以用来查看倒数第N个字符为什么

示例1：

以`/etc/passwd`为例，以:号为分隔符查看每行的字段数量

```bash
[root@centos7 ~]# awk -F: '{print NF}' /etc/passwd
7
7
7
...以下省略...
```
示例2：

查看`/etc/passwd`文件内，以:为分隔符，倒数第2个字段的内容

```bash
[root@centos7 ~]# awk -F: '{print $(NF-1)}' /etc/passwd
/root
/bin
/sbin
...以下省略...
```
##### 6.`NR`:记录数
`NR`变量为已读文件的记录数
示例1：
以;为记录符，在`a.txt`的文件的每条记录前加上记录行。

```bash
[root@centos7 ~]# cat a.txt
a,b,c;1,2,3,4,;aa,bb,cc
zz,yy,xxx;122,444,2322;AA
BB,CC
[root@centos7 ~]# awk -v RS=";" '{print NR,$0}' a.txt 
1 a,b,c
2 1,2,3,4,
3 aa,bb,cc
zz,yy,xxx
4 122,444,2322
5 AA
BB,CC
```
`NR`变量还可以合并计数多个文件的记录

示例2：查看`/etc/fstab`和`/etc/issue`总共有多少记录号

```bash
[root@centos7 ~]# awk '{print NR}' /etc/fstab /etc/issue
1
2
...中间省略...
14
15
```
##### 7.`FNR`：文件的记录数
`FNR`变量为每个文件分别记录记录数。  

示例：查看`/etc/fstab`和`/etc/issue`的记录数

```bash
[root@centos7 ~]# awk '{print FNR}' /etc/fstab /etc/issue
1
2
...中间省略...
11
12
1
2
3
```
##### 8.`FILENAME`:文件名
FILENAME变量为输出文件名  

示例：在匹配到的行后面加上文件名

```bash
[root@centos7 ~]# awk '{print $0,FILENAME}' /etc/passwd
root:x:0:0:root:/root:/bin/bash /etc/passwd
bin:x:1:1:bin:/bin:/sbin/nologin /etc/passwd
daemon:x:2:2:daemon:/sbin:/sbin/nologin /etc/passwd
...以下省略...
```
##### 9.`ARGC`:命令行参数个数
`ARGC`变量存放的为参数个数。  

示例：

```bash
[root@centos7 ~]# awk '{print ARGC}' /etc/issue
2
2
2           
```
显示为2个参数，是哪两个参数呢？看下面那个函数
##### 10.`ARGV`:查看命令行参数
`ARGV`变量为一个数组里面存放的为`awk`的每个参数

示例：

打印每一个参数

```bash
[root@centos7 ~]# awk '{print ARGV[1]}' /etc/issue
/etc/issue
/etc/issue
/etc/issue
[root@centos7 ~]# awk '{print ARGV[0]}' /etc/issue
awk
awk
awk
```
##### 自定义变量

自定义变量的赋值有2种方法  

###### 1. `-v var=value`
示例1：输出变量在每行
```bash
[root@centos7 ~]# awk -v title=ceo -F: '{print title":"$1}' /etc/passwd
ceo:root
ceo:bin
ceo:daemon
ceo:adm
ceo:lp
```
###### 2. 可以直接在program中直接定义   
把变量放在{}内赋值时，变量值必须要加双引号，变量必须先赋值再引用，次序错误会导致第一次匹配到的行没有值   

示例1：在每个用户前加上`ceo`

```bash
[root@centos7 ~]# awk -F: '{title="ceo"; print title":"$1}' /etc/passwd
ceo:root
ceo:bin
ceo:daemon
ceo:adm
ceo:lp
ceo:sync
```
示例2：调用脚本文件  

`awk '{action}'`单引号内的脚本可以存放在文件内被调用

```bash
[root@centos7 ~]# cat 2.txt
{title="ceo";print title":"$1}
[root@centos7 ~]# awk -F: -f 2.txt /etc/passwd
ceo:root
ceo:bin
ceo:daemon
```
#### 三、`printf`
`printf`格式化输出：

```bash
printf "FORMAT" ,item1,item2,...
```
使用`printf`时，需要注意以下3点：

1. 使用printf时必须指定FORMAT

2. printf不会自动换行，需要显式给出换行控制符`\n`

3. FORMAT中需要分别为后面每个item指定格式符

   printf的格式符有以下：

|格式符|说明|
|:-|:-|
|%c|显示字符的ASCII码|
|%d,%i|显示十进制整数|
|%e，%E|显示科学计数法数值|
|%f|显示浮点数|
|%g|以科学计数法或浮点形式显示数值|
|%s|显示字符串|
|%u|无符号整数|
|%%|显示%自身|
printf中除了格式符还有修饰符：
|修饰符|说明|
|:-|:-|
|#[.#]|第一个数字控制显示的宽度；第二个#标识小数点后精度，%3.1f|
|-|左对齐（默认为右对齐）%-15s|
|+|显示数值的正负值符号%+d|

示例1：
%d打印整数
```bash
[root@centos7 ~]# echo 100.222 | awk '{printf "this is %d" ,$0"\n"}'
this is 100
```
示例2：
格式符有几个对应的item项就要就几个否则语法错误
```bash
[root@centos7 ~]# echo "123.455 221.245" | awk '{printf "this is %d %d" ,$1,$2}'
this is 123 221
```
示例3：
%f可以打印小数，默认输出的为6位。也可以指定小数位。
```bash
[root@centos7 ~]# echo "123.455 221.245" | awk '{printf "this is %f" ,$1}'
this is 123.455000
#指定小数位用法
[root@centos7 ~]# echo "123.455 221.245" | awk '{printf "this is %.3f" ,$1}'
this is 123.455
```
示例4：
.之前的数字为长度（默认右对齐）
```bash
[root@centos7 ~]# echo "123.455 221.245" | awk '{printf "this is %10.3f %10.3f" ,$1,$2}'        
this is    123.455    221.245
```
示例5：
指定左对齐，在%后面加上-
```bash
[root@centos7 ~]# echo "123.222 345.23" |awk '{printf "this is %-10.3f %-10.3f",$1,$2}'
this is 123.222    345.230   
```


### 四、操作符
awk还支持各种操作符如算数操作符、字符串操作符、赋值操作符、比较操作符、模式匹配操作符
#### 1.算数操作符
算数操作符有+、-、*、/、^、%，如：
x+y,x-y,x*y,x/y,x^y,x%y
示例1：
简单的算术运算

```bash
[root@centos7 ~]# awk 'BEGIN{I=20;J=10;M=I+J;print M}'
30
```
示例2：
-x:转换为负数
```bash
[root@centos7 ~]# awk 'BEGIN{I=20;I=-I;print I}'
-20
```
#### 2.赋值操作符：
赋值操作符有：=，+=，-=，*=，/=，%=，^=,++,--
示例1：
```bash
[root@centos7 ~]# awk 'BEGIN{I=10;I+=2;print I}'
12
[root@centos7 ~]# awk 'BEGIN{I=10;I-=2;print I}'
8
[root@centos7 ~]# awk 'BEGIN{I=10;I*=2;print I}'
20
```
#### 3.比较操作符和模式匹配符
比较操作符和模式匹配符常用在awk的行过滤pattern中，Pattern为空时所有都符合条件

pattern中可以添加比较符号和模式匹配

比较符号有：==(等于)，!=(不等于)，>(大于)，>=(大于等于)，<(小于)，<=(小于等于)

模式匹配符：~(符号左边内容是否右边匹配内容，包含内容)，!~(符号左边内容的是否不匹配右边内容)

模式匹配时可以使用正则表达式！

3.1比较操作符示例

示例1:

取出连接主机的IP

```bash
[root@centos7 ~]# ss -tn | awk -F" +|:" 'NR>1{print $(NF-2)}' 
192.168.172.1
```
示例2:

取出用户列表中UID大于1000的用户和ID

```bash
[root@centos7 ~]# awk -F: '$3>1000{print $1,$3}' /etc/passwd
nfsnobody 65534
```
示例3:

取出ip地址

```bash
[root@centos7 ~]# ifconfig | awk 'NR==2{print $2}'
192.168.172.129
```
示例4:

取出分区利用率大于10的设备

```bash
[root@centos7 ~]# df | awk -F" +|%" '/\/dev\/sd/ && $5>10{print $1,$5}'
/dev/sda1 17
```
3.2模式匹配示例  

示例1：匹配第一列为/dev/sd开头行

```bash
[root@centos7 ~]# df | awk '/\/dev\/sd/{print $1}'
/dev/sda2
/dev/sda3
/dev/sda1
```
示例2：匹配/etc/fstab非#开头的行
```bash
[root@centos7 ~]# awk '!/^#/' /etc/fstab 

UUID=45490aa4-cf29-420d-a606-af32688b6707 /                       xfs     defaults        0 0
UUID=15dcd896-b7cf-48d0-b8bd-4c0b0f2c62b2 /boot                   xfs     defaults        0 0
UUID=4b6e1813-2c46-402a-869a-02cbbcb76ade /data                   xfs     defaults        0 0
UUID=0995b444-48c1-4423-92bc-2deda0d3c082 swap                    swap    defaults        0 0
```
#### 4.逻辑操作符  

逻辑操作符：与&&，或||，非！  

示例1：  

取出/etc/passwd非nologin结尾的行

```bash
[root@centos7 ~]# awk '!/nologin$/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
masuri:x:1000:1000:masuri:/home/masuri:/bin/bash
```
示例2：  

取出分区利用率大于10的设备名和分区利用率

```bash
[root@centos7 ~]# df | awk -F" +|%" '/\/dev\/sd/ && $5>10{print $1,$5}'
/dev/sda1 17
```
示例3：  

取出连接数前3的IP地址  

```bash
[root@centos7 ~]# netstat -tn | awk -F" +|:" '/ESTABLISHED/{print $6}' | sort | uniq -c | sort -nr | head -3
```

#### 5.三目表达式  
三目表达式格式：  

selector?if-ture-expression:if-false-expression  

判断selector是否为真，如果为真则执行if-ture语句，不为真则执行if-false语句。  

示例1：  

在Uid大于等于1000的用户前添加commom user，小于1000的添加system user  

```bash
[root@centos7 ~]# awk -F: '$3>=1000?USER="COMMOM USER:":USER="SYSTEM USER:"{print USER$1}' /etc/passwd
SYSTEM USER:root
SYSTEM USER:bin
...中间省略...
SYSTEM USER:ntp
SYSTEM USER:tcpdump
COMMOM USER:masuri
```
### 五、PATTERN部分总结
PATTEN:awk在执行时会根据pattern条件，过滤匹配的行，再做处理。  

pattern的条件可以有以下几种：  

#### 1.空  
如果pattern部分为空，则默认匹配每一行    

示例：打印所有行  

```bash
[root@centos7 ~]# awk '{print $0}' /etc/fstab

#
# /etc/fstab
# Created by anaconda on Tue Mar  5 21:07:19 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=45490aa4-cf29-420d-a606-af32688b6707 /                       xfs     defaults        0 0
UUID=15dcd896-b7cf-48d0-b8bd-4c0b0f2c62b2 /boot                   xfs     defaults        0 0
UUID=4b6e1813-2c46-402a-869a-02cbbcb76ade /data                   xfs     defaults        0 0
UUID=0995b444-48c1-4423-92bc-2deda0d3c082 swap                    swap    defaults        0 0
```
#### 2.正则表达式    
/regular expression/:仅处理能够被模式匹配到的行，需要用//括起来  

示例：找出/etc/passwd中以g开头的行的第1字段和第3字段  

```bash
[root@centos7 ~]# awk -F: '/^g/{print $1,$3}' /etc/passwd
games 12
gluster 993
geoclue 992
gdm 42
gnome-initial-setup 989
```
#### 3.关系表达式  
relational expression: 关系表达式，结果为“真”才会被处理  

真和假的定义：  

3.1真：结果为非0值，非空字符串    

示例1: 当有值时为真，输出所有。

```bash
[root@centos7 ~]# seq 3 | awk '2'
1
2
3
```
示例2：当数字为负数时，输出也为真
```bash
[root@centos7 ~]# seq 3 | awk '"-1"'
1
2
3
```
示例3：当中间的数字为空格时也为真
```bash
[root@centos7 ~]# seq 3 | awk '" "'
1
2
3
```
3.2假：结果为空字符串或0值，假则不会被处理  

示例1：当值为空和0时不做输出  

```bash
[root@centos7 ~]# seq 5 | awk '""'
[root@centos7 ~]# seq 5 | awk '0'
[root@centos7 ~]# 
```
示例2：在awk中不加字符不添加双引号表示为变量，变量为空和0时也为假，变量中有值时为真
```bash
[root@centos7 ~]# seq 5 | awk 'i'               #变量中没有值，假
[root@centos7 ~]# seq 5 | awk -v i=1 'i'        #变量中有值，真，输出结果
1
2
3
4
5
```
#### 4.行范围  
pattern可以使用行范围进行匹配  

/pat1/,/pat2/ 不支持直接给出数字格式   

示例：打印b开头的行到f开头的行

```bash
[root@centos7 ~]# awk '/^b/,/^f/' /etc/passwd
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
```
#### 5.BEGIN/END模式  
BEGIN{}：仅在开始处理文件中的文本之前执行一次  

END{}：仅在文本处理完成之后执行一  

示例1：使用BEGIN来添加表头，做格式化输出。

```bash
[root@centos7 ~]# awk -F: 'BEGIN{print "|USERNAME                       |UID\n------------------------------------"}{printf "|%-30s |%-10d\n--------------------------------------\n",$1,$3}' /etc/passwd 
|USERNAME                       |UID
------------------------------------
|root                           |0         
--------------------------------------
|bin                            |1         
--------------------------------------
|daemon                         |2         
--------------------------------------
|adm                            |3         
--------------------------------------
|lp                             |4         
--------------------------------------
|sync                           |5         
--------------------------------------
|shutdown                       |6         
```
### 六、awk控制语句
流程控制语句是任何程序设计语言都不能缺少的部分。任何好的语言都有一些执行流程控制的语句。awk提供的完备的流程控制语句类似于C语言，这给我们编程带来了极大的方便。  
awk控制语句有：if-else，while循环，do-while循环，for循环，break，continue，delete array[index]，delete array,exit
#### 1.if-else
 使用场景：对awk取得的整行或某个字段做条件判断   
 语法：
```bash
 if(condition){statement;…}[else statement]    
 if(condition1){statement1}else if(condition2){statement2}else{statement3} 
```
示例：1.打印出/etc/passwd下uid大于1000的行
```bash
[root@centos7 ~]# awk -F: '{if($3>1000)print $0}' /etc/passwd
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
```
示例2：如果Uid为偶数则打印出uid和用户名
```bash
[root@centos7 ~]# awk -F: '{if($3%2==0)print $1,$3}' /etc/passwd
root 0
daemon 2
lp 4
shutdown 6
mail 8
games 12
ftp 14
systemd-network 192
libstoragemgmt 998
rpc 32
saslauth 996
rtkit 172
nfsnobody 65534
unbound 994
geoclue 992
saned 990
gdm 42
sshd 74
avahi 70
ntp 38
tcpdump 72
masuri 1000
```
示例3：考试成绩判断，60及格，80分还行，80分以上真棒，60以下不及格
```bash
[root@centos7 ~]# awk -v score=80 'BEGIN{if(score<60){print "no pass"}else if(score<=80){print "just so so"}else if(score>80){print "good"}}'
just so so
```
#### 2.while循环
使用场景：  
1.对一行内的多个字段逐一类似处理时使用  
2.对数组中的各元素逐一处理时使用    
语法：
```bash
while(condition){statement;...}
```
条件为真进入循环，条件为假退出循环。  
示例1：打印/etc/password第一行每一字段的长度
```bash
[root@centos7 ~]# awk -F: 'NR==1{i=1;while(i<=NF){print $i,length($i);i++}}' etc/passwd
root 4
x 1
0 1
0 1
root 4
/root 5
/bin/bash 9
```
示例2：取出最大值和最小值
```bash
[root@centos7 ~]# awk -F, '{max=$1;mix=$1;i=1;while(i<=NF){if($i>max){max=$i};if($i<mix){mix=$i};i++};print mix,max}' test1 
207 31976
```
示例3：1+2+3+..+100
```bash
[root@centos7 ~]# awk 'BEGIN{i=1;while(i<=100){sum+=i;i++}print sum}'
5050
```
#### 3.for循环
常用语法：
```bash
for(expr1;expr2;expr3){statement;...}
```
特殊用法：便利数组中的元素
语法：
```bash
    for(i in arrary){for body}
```
示例1：for循环1+到100
```bash
[root@centos7 ~]# awk 'BEGIN{for(i=1;i<=100;i++){sum+=i}print sum}'
5050
```
#### 4.break和continue
break为提前结束循环
continue为提前结束本次循环，进入下一次循环  
示例：将1到100的偶数相加
```bash
[root@centos7 ~]# awk 'BEGIN{for(i=1;i<=100;i++){if(i%2==0){sum+=i}}print sum}'
2550
```
1-100的偶数相加，当i=50时跳出不加，继续执行后续循环

```bash
[root@centos7 ~]# awk 'BEGIN{for(i=1;i<=100;i++){if(i==50){continue};if(i%2==0){sum+=i}}print sum}'
2500
```
1-100的偶数相加，当i=50时跳出所有循环
```bash
[root@centos7 ~]# awk 'BEGIN{for(i=1;i<=100;i++){if(i==50){break};if(i%2==0){sum+=i}}print sum}'
600
```
#### 5.next
next是提前结束对本行的处理直接进入下一行的处理，break跳出的是awk自身的循环。
示例：
打印Uid为偶数的行

```bash
[root@centos7 ~]# awk -F: '{if($3%2==!0){next}print $1,$3}' /etc/passwd
root 0
daemon 2
lp 4
shutdown 6
mail 8
games 12
ftp 14
systemd-network 192
libstoragemgmt 998
rpc 32
saslauth 996
rtkit 172
nfsnobody 65534
unbound 994
geoclue 992
saned 990
gdm 42
sshd 74
avahi 70
ntp 38
tcpdump 72
masuri 1000
```

### 七、awk数组
awk数组为关联数组：array[index-expression]  
index-expression为数组下标  
数组在使用时需注意以下几点：  
1.数组下标可使用任意字符串;字符串要使用双引号括起来  
2.如果某数组元素事先不存在，在引用时，awk会自动创建此元素，并将其初始化为“空串”  
3.若要判断数组中是否存在某元素，要时用“index in array”格式进行遍历

示例：给数组赋值，和输出数组
```bash
[root@centos7 ~]# awk 'BEGIN{arr["ceo"]="mage";arr["cto"]="laowang";print arr["ceo"]}'
mage
[root@centos7 ~]# awk 'BEGIN{arr["ceo"]="mage";arr["cto"]="laowang";print arr["cto"]}'
laowang
```
若要遍历数组中的每个元素，要使用for循环 
```bash
for(var in array){for-body}  
```
注意：var会遍历arry的每个索引  
示例：遍历数组，可以先将数组下标赋给i
```bash
[root@centos7 ~]# awk 'BEGIN{arr["ceo"]="mage";arr["cto"]="laowang";for(i in arr){print arr[i]}}'
mage
laowang
```

示例2：数组的其他用法：去重  
创建一个文件输入以下内容，然后执行awk命令
```bash
[root@centos7 ~]# cat f1.txt 
aa
bb
cc
aa
11
22
11
[root@centos7 ~]# awk '!line[$0]++' f1.txt
aa
bb
cc
11
22
```
此方法比较绕，读入第一行aa作为数组line的下标，此时数组内的值为空“假”取反后为真打印此行，然后再执行++，此时line内的值为1，当再次遇到aa的行时，数组line[aa]内有值1，取反后为假不输出，然后再++此时[aa]内值为2
验证：
```bash
[root@centos7 ~]# awk '{!line[$0]++;print $0,line[$0]}' f1.txt
aa 1
bb 1
cc 1
aa 2
11 1
22 1
11 2
```
示例：数组的高级用法  
取连接状态数
```bash
[root@centos7 ~]# netstat -tna | awk '/^tcp/{state[$NF]++}END{for(i in state){print i,state[i]}}'       #state[$NF]++表示将最后一个字段作为数组的下标，然后对里面的值+1，当再次遇到相同的下标时，再次加1
LISTEN 11
ESTABLISHED 1
```
统计每个ip的连接次数
```bash
[root@centos7 ~]# awk '{ip[$1]++}END{for(i in ip){print i,ip[i]}}' access_log 
172.20.0.200 1482
172.20.21.121 2
172.20.30.91 29
172.16.102.29 864
172.20.0.76 1565
172.20.9.9 15
172.20.1.125 463
172.20.61.11 2
172.20.73.73 198
172.20.107.222 3
172.20.0.222 2834
172.20.111.240 4
```
取出连接数排名前10的ip
```bash
[root@centos7 ~]# awk '{ip[$1]++}END{for(i in ip){print ip[i],i}}' access_log | sort -nr | head
4870 172.20.116.228
3429 172.20.116.208
2834 172.20.0.222
2613 172.20.112.14
2267 172.20.0.227
2262 172.20.116.179
2259 172.20.65.65
1565 172.20.0.76
1482 172.20.0.200
1110 172.20.28.145
```
小练习：
统计男女生的平均分数
|name|score|gender|
|:-|:-|:-|
|a|100|m|
|b|99|f|
|c|80|m|
|d|98|f|

```bash
[root@centos7 ~]# awk -F" +" 'NR>1{num[$3]++;sum[$3]+=$2}END{for(i in sum)print i,sum[i]/num[i]}' score 
m 90
f 98.5
```
### 八、字符串处理
1.length([s]):返回字符串的长度
示例：
```bash
[root@centos7 ~]# awk 'BEGIN{print length("abc")}'
3
```
2.sub(r,s,[t]):对t字符串搜索r表示的模式匹配的内容，并替换为s所表示的内容(只替换第一次匹配到的)
示例：

```bash
[root@centos7 ~]# echo "2008:08:08 08:08:08" | awk '{sub(/:/,"-",$1);print $0}'
2008-08:08 08:08:08
```
3.gsub(r,s,[t]):对t字符串搜索r表示的模式匹配的内容，并全部替换为s所表示的内容
示例：
```bash
[root@centos7 ~]# echo "2008:08:08 08:08:08" | awk '{gsub(/:/,"-",$1);print $0}'
2008-08-08 08:08:08
```
4.splits(s,arry,[r]):以r为分隔符，切割字符串，并将切割后的字符串保存至array做表示的数组中，第一个索引值为1，第二个索引值为2，...
示例：

```bash
[root@centos7 ~]# echo "2008:08:08 08:08:08" | awk '{split($0,arr,":");}END{for(i in arr){print i,arr[i]}}'
4 08
5 08
1 2008
2 08
3 08 08
```
用函数来实现ip连接次数
```bash
[root@centos7 ~]# awk '/^ESTAB/{split($NF,ip,":");count[ip[1]]++}END{for(i in count){print i,count[i]}}' ss.log 
192.168.172.1 465
```
### 九、自定义函数

语法：  
```bash
function name (parameter,parameter,...){
            statements
            return experssion
}
```
示例：
awk函数定义方法
```bash
[root@centos7 ~]# cat fun.awk 
#!/bin/awk -f
function max(x,y){      #x,y为行参
	x>y?var=x:var=y
	return var
}
BEGIN{print max(a,b)}   #a,b为实参
```
awk函数调用
```bash
[root@centos7 ~]# awk -v a=40 -v b=50 -f fun.awk
50
[root@centos7 ~]# 
```

### 十、awk中调用shell命令
system命令：空格是awk中的字符串连接符，如果system中需要使用awk中的变量可以使用空格分隔，或者说除了awk的变量外其他一律用“”引用起来  
示例1：调用系统命令
```bash
[root@centos7 ~]# awk 'BEGIN{system(hostname)}'
[root@centos7 ~]# awk 'BEGIN{system("hostname")}'
centos7.localdomain
[root@centos7 ~]# awk 'BEGIN{system("echo hello")}'
hello
[root@centos7 ~]# awk 'BEGIN{system("ls")}'
1.txt	    access.log	     f1.txt   initial-setup-ks.cfg  ss.log
access_log  anaconda-ks.cfg  fun.awk  ss.lg		    test1
[root@centos7 ~]# 
```
示例2：输出变量的方法  
调用双引号输出变量时，变量必须放在双引号外，如果放在双引号内，变量只会被当作字符输出
```bash
[root@centos7 ~]# awk -v var=hello 'BEGIN{system("echo var")}'
var
[root@centos7 ~]# awk -v var=hello 'BEGIN{system("echo"var)}'   #echo后必须有空格
sh: echohello: command not found
[root@centos7 ~]# awk -v var=hello 'BEGIN{system("echo" var)}'  #并且空格必须再双引号内
sh: echohello: command not found
[root@centos7 ~]# awk -v var=hello 'BEGIN{system("echo "var)}'
hello
```
### 十一、awk脚本  
awk脚本格式：  
1.脚本后缀为.awk  
2.脚本首行加上#!/bin/awk -f  
3.给脚本添加执行权限  
示例： 

```bash
[root@centos7 ~]# cat test.awk 查看脚本内容
#!/bin/awk -f
#this is test awk script
{if($3>1000)print $1,$3}
[root@centos7 ~]# chmod +x test.awk 
[root@centos7 ~]# ./test.awk -F: /etc/passwd
nfsnobody 65534
```
### 十二、向awk脚本传递参数
使用格式：  
```bash
awkfile var=value var2=value2 ... inputfile
```
注意：在BEGIN过程中不可用。直到首行输入完成以后，变量才可用。可以通过-v参数，让awk在执行BIGIN之前得到变量的值。命令行中每一个指定的变量都需要一个-v参数。  
示例：
创建一个脚本，此处以上一个脚本为例进行修改。

```bash
[root@centos7 ~]# cat test.awk      #查看脚本内容
#!/bin/awk -f
#this is test awk script
$3>=min && $3<=max{print $1,$3}   ，j   #其中min和max为位置参数，$1,$3表示awk所要扫描的文件中的字段。此处以/etc/passwd为例，就是对应的用户名和UID
 
[root@centos7 ~]# ./test.awk -F: min=10 max=20 /etc/passwd  #找出uid再10-20之间的行
operator 11
games 12
ftp 14

```