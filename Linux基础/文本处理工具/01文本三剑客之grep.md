### 文本三剑客之`grep`

`grep`: Global search REgular expression and Print out the line

作用：文本搜索工具，根据用户指定的"模式"对目标文本逐行进行匹配检查；打印匹配到的行。

`PATTERN`(模式)：由正则表达式字符及文本字符所编写的过滤条件

```bash
grep [OPTIONS] PATTERN [FILE...]
```

`PATTERN`的简单使用：

```bash
grep root /etc/passwd 			# 从文件中挑出包含root的行
grep "$USER" /etc/passwd 		# grep中可以使用变量
grep '$USER'/etc/passwd			# 当使用单引号时则表示字符串本身，变量不生效
grep `whoami`/etc/passwd		# grep中还能使用命令的执行结果
```

`OPTIONS`选项：

* `--color=auto`: 对匹配到的文本着色显示

* `-m #`:匹配#次后停止(`grep`处理文件时是逐行处理，批配到#次后停止)

  ```bash
  [root@mylinuxops /]# grep -m1 root /etc/passwd
  root:x:0:0:root:/root:/bin/bash
  ```

* `-v`: 显示不被pattern匹配到的行

  ```bash
  [root@mylinuxops /]# grep -v root /etc/passwd
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  ...
  ```

* `-i`: 忽略字符大小写

  ```bash
  [root@mylinuxops /]# grep ROOT /etc/passwd
  [root@mylinuxops /]# grep -i ROOT /etc/passwd
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  ```

* `-n`: 显示匹配的行号

  ```bash
  [root@mylinuxops /]# grep -n root /etc/passwd
  1:root:x:0:0:root:/root:/bin/bash
  10:operator:x:11:0:operator:/root:/sbin/nologin
  ```

* `-c`: 统计匹配的行数

  ```bash
  [root@mylinuxops /]# grep -c root /etc/passwd
  2
  ```

* `-o` :仅显示匹配到的字符串

  ```bash
  [root@mylinuxops /]# grep -o root /etc/passwd
  root
  root
  root
  root
  # 此处用字符来表示没有意义，使用正则表达式作用更大
  ```

* `-q`: 静默模式，不输出任何信息

  ```bash
  # 静默模式输出时，不显示内容但是可以使用$?来判断，常用在脚本
  [root@mylinuxops /]# grep -q root /etc/passwd
  [root@mylinuxops /]# echo $?
  0
  [root@mylinuxops /]# grep -q Root /etc/passwd
  [root@mylinuxops /]# echo $?
  1
  ```

* `-A #`: after, 后#行

  ```bash
  1:root:x:0:0:root:/root:/bin/bash
  2-bin:x:1:1:bin:/bin:/sbin/nologin
  --
  10:operator:x:11:0:operator:/root:/sbin/nologin
  11-games:x:12:100:games:/usr/games:/sbin/nologin
  ```

* `-B #`: before, 前#行

  ```bash
  [root@mylinuxops /]# grep -B 1 "up" 22.log 
  Nmap scan report for 172.16.22.1
  Host is up (0.0047s latency).
  --
  Nmap scan report for 172.16.22.23
  Host is up (0.14s latency).
  ```

* `-C #`: context, 前后各#行

  ```bash
  [root@mylinuxops /]# grep -nC1 root /etc/passwd
  1:root:x:0:0:root:/root:/bin/bash
  2-bin:x:1:1:bin:/bin:/sbin/nologin
  --
  9-mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
  10:operator:x:11:0:operator:/root:/sbin/nologin
  11-games:x:12:100:games:/usr/games:/sbin/nologin
  ```

* `-e`: 实现多个选项间的逻辑or关系

  ```bash
  grep –e 'cat' -e 'dog' file
  ```

* `-w`: 匹配整个单词，字母数字下划线算一个单词

  ```bash
  # 不使用-w，rooter也能匹配
  [root@mylinuxops /]# grep root /etc/passwd
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  rooter:x:1006:1009::/home/rooter:/bin/bash
  # 使用-w后只匹配root
  [root@mylinuxops /]# grep -w root /etc/passwd
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  ```

  

* `-E`: 使用ERE

* `-F`: 相当于`fgrep`，不支持正则表达式

* `-f file`: 根据模式文件处理