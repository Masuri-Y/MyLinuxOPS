### 文本三剑客之`sed`

`grep`命令可以对文件进行过滤，默认是过滤行。`sed`命令也能实现过滤，不仅能过滤还可以修改文件。

在脚本中通常需要对文件进行更新和更改内容，这时候就需要使用到`sed`命令来实现这个功能。

`sed`命令也支持正则表达式，所以其可以利用正则表达式实现更加灵活的用法。

`sed`是一种流编辑器(`Stream EDitor`)，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”(pattern space)，接着用`sed`命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。然后读入下行，执行下一个循环。如果没有使诸如‘D’的特殊命令，那会在两个循环之间清空模式空间，但不会清空保留空间。这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

功能：主要用来自动编辑一个或多个文件,简化对文件的反复操作,编写转换程序等

参考： <http://www.gnu.org/software/sed/manual/sed.html>

#### `sed`工具

##### 用法：

```bash
sed [option]... 'script' inputfile...
```

##### 常用选项：

* `-n`: 不输出模式空间内容到屏幕，即不自动打印

* `-e`: 多点编辑

  ```bash
  # 使用多点编辑的方法，取出ip地址
  [root@mylinuxops data]# ifconfig eth0 | sed -rn -e '2s/.*inet//' -e '2s/.* //p'
  172.16.11.255
  ```

* `-f /PATH/SCRIPT_FILE`: 从指定文件中读取编辑脚本

* `-r`: 支持使用扩展正则表达式

* `-i.bak`: 备份文件并原处编辑

##### script: 

script指的是`sed`自己的脚本语言。脚本内的内容由`地址`和`命令`组成。

* `'地址命令'`: 

  * 地址: 对哪一行进行处理。地址可以是指定的行，也可以使用正则表达式进行匹配。

    * 地址定界：

      * 不给地址：对全文进行处理

      * 单地址：

        * `#`: 指定的行，`$`表示最后一行

          ```bash
          # 挑出第10行
          [root@mylinuxops data]# sed -n '10p' passwd
          operator:x:11:0:operator:/root:/sbin/nologin
          
          # 挑出最后一行
          [root@mylinuxops data]# sed -n '$p' passwd
          cockpit-wsinstance:x:991:988:User for cockpit-ws instances:/nonexisting:/sbin/nologin
          
          # 若没有-n选项，则会打印文件内所以的行，第10行打印2遍
          [root@mylinuxops data]# sed '10p' passwd
          
          # sed还可以接受标准输出的内容进行处理
          # 取出ifconfig的第二行
          [root@mylinuxops ~]# ifconfig | sed -n '2p'
                  inet 172.16.11.61  netmask 255.255.255.0  broadcast 172.16.11.255
          ```

        * `/pattern/`: 被此处模式所能够匹配到的每一行

          ```bash
          # 打印出#开头的行
          [root@mylinuxops ~]# sed -n '/^#/p' /etc/fstab 
          #
          # /etc/fstab
          # Created by anaconda on Mon Mar  1 05:07:57 2021
          ...
          
          # 挑出df命令输出中vd的行
          [root@mylinuxops ~]# df | sed -n '/.*vd.*/p'
          /dev/vda2      202804352 1953256 190479464   2% /
          /dev/vda1         487634  148223    309715  33% /boot
          ```

      * 地址范围：

        * `#,#`: 打印第x行到第y行

          ```bash
          # 打印第3行到第6行
          [root@mylinuxops data]# seq 10 | sed -n '3,6p'
          3
          4
          5
          6
          ```

        * `#,+#`: 从第x行往后+y行

          ```bash
          # 从第三行往后+2行
          [root@mylinuxops data]# seq 10 | sed -n '3,+2p'
          3
          4
          5
          ```

        * `/pat1/,/pat2/`或`#,/pat1/`: 查找从`/pat1/`到`/pat2/`之间的行

          ```bash
          # 匹配从a开头到s开头的行，会匹配文件内所有a开头到s开头的行，若只有a开头没有s开头则匹配剩下的全文。
          [root@mylinuxops data]# sed -n '/^a/,/^s/p' passwd
          adm:x:3:4:adm:/var/adm:/sbin/nologin
          lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
          sync:x:5:0:sync:/sbin:/bin/sync
          ```

      * `~`: 步进

        * `1~2`: 奇数行
      
          ```bash
          [root@mylinuxops data]# seq 10 | sed -n '1~2p'
          1
          3
          5
          7
          9
          ```
        
        * `2~1`: 偶数行
        
          ```bash
          [root@mylinuxops data]# seq 10 | sed -n '2~2p'
          2
          4
          6
          8
          10
          ```

  * 命令: 可以将其显示、删除或者搜索替代。

    * 编辑命令：

      * `d`: 删除模式空间匹配的行，并立即启用下一轮循环

        ```bash
        # 将文件读入模式空间，当匹配时则删除，将不匹配的行输出。
        [root@mylinuxops data]# seq 5 | sed '2d'
        1
        3
        4
        5
        
        # 将非#开头的行删除。
        [root@mylinuxops data]# sed '/^#/d' /etc/fstab 
        
        UUID=dfd00454-d3b4-475f-aacb-64e27e5377f9 /                       ext4    defaults        1 1
        UUID=cde18fcc-03b4-4c90-a235-512769ec23bc /boot                   ext4    defaults        1 2
        UUID=5596e69d-a88a-498c-b821-f0f114a368d6 none                    swap    defaults        0 0
        ```

      * `p`:打印当前模式空间内容，追加到默认输出之后

        ```bash
        # sed命令读入一行，若脚本没有指令，则自动将读入的行输出
        [root@mylinuxops data]# sed '' passwd
        root:x:0:0:root:/root:/bin/bash
        bin:x:1:1:bin:/bin:/sbin/nologin
        daemon:x:2:2:daemon:/sbin:/sbin/nologin
        ...
        
        # 若使用p命令，则会将读入的行进行打印，而系统自动会打印读如的行，则显示的内容为2行，每行内容重复显示
        [root@mylinuxops data]# sed 'p' passwd
        root:x:0:0:root:/root:/bin/bash
        root:x:0:0:root:/root:/bin/bash
        bin:x:1:1:bin:/bin:/sbin/nologin
        bin:x:1:1:bin:/bin:/sbin/nologin
        daemon:x:2:2:daemon:/sbin:/sbin/nologin
        daemon:x:2:2:daemon:/sbin:/sbin/nologin
        ...
        
        # 使用-n可以取消系统的自动打印
        [root@mylinuxops data]# sed -n 'p' /etc/passwd
        root:x:0:0:root:/root:/bin/bash
        bin:x:1:1:bin:/bin:/sbin/nologin
        daemon:x:2:2:daemon:/sbin:/sbin/nologin
        ...
        ```

      * `a [\]text`: 在指定行后面追加文本，支持使用\n实现多行追加

        ```bash
        # 在#开头的行后追加横线
        [root@mylinuxops data]# sed '/^#/a------------' /etc/fstab 
        
        #
        ------------
        # /etc/fstab
        ------------
        # Created by anaconda on Mon Mar  1 05:07:57 2021
        ------------
        ....
        
        # 使用-i修改文件，在匹配的内容后增加一行，建议使用-i.bak对原文件作备份。
        [root@mylinuxops data]# sed -i.bak '/.*aliases/aalias ls=hostname' ~/.bashrc 
        [root@mylinuxops data]# ll ~/.bashrc*
        -rw-r--r--  1 root root 194 Mar 20 18:36 /root/.bashrc
        -rw-r--r--. 1 root root 176 May 11  2019 /root/.bashrc.bak
        # 原文件已经修改
        [root@mylinuxops data]# cat ~/.bashrc
        # .bashrc
        # User specific aliases and functions
        alias ls=hostname
        
        # 插入的行中前面存在特殊字符时需要使用\转意
        [root@mylinuxops data]# seq 3 | sed 'a\ xyz'
        1
         xyz
        2
         xyz
        3
         xyz
         
        # 若要插入变量则需要使用'''三引号来包裹
        [root@mylinuxops data]# seq 3 | sed 'a\'''$USER''''
        1
        root
        2
        root
        3
        root
        ```

      * `i [\]text`: 在行前面插入文本

        ```
        [root@mylinuxops data]# seq 3 | sed 'i\'''$USER''''
        root
        1
        root
        2
        root
        3
        ```

      * `c [\]text`: 替换行为单行或多行文本

        ```bash
        [root@mylinuxops data]# seq 3 | sed 'c\'''$USER''''
        root
        root
        root
        ```

      * `w /path/file`: 保存模式匹配的行至指定文件，相当于重定向

        ```bash
        # 使用重定向
        [root@mylinuxops data]# seq 10 | sed -n '3,5p' > log1
        [root@mylinuxops data]# cat log1
        3
        4
        5
        
        # 使用w指令写入文件
        [root@mylinuxops data]# seq 10 | sed -n '3,5w ./log2'
        [root@mylinuxops data]# cat log2 
        3
        4
        5
        ```

      * `r /path/file`: 读取指定文件的文本至模式空间中匹配到的行后

        ```bash
        # 在每行后插入文件的内容
        [root@mylinuxops data]# seq 3 | sed 'r /etc/issue'
        1
        \S
        Kernel \r on an \m
        
        2
        \S
        Kernel \r on an \m
        
        3
        \S
        Kernel \r on an \m
        
        ```

      * `=`: 为模式空间中的行打印行号

        ```bash
        [root@mylinuxops data]# sed '=' /etc/issue
        1
        \S
        2
        Kernel \r on an \m
        3
        
        ```

      * `!`: 模式空间中匹配行取反处理

        ```bash
        # 打印非#开头的行
        [root@mylinuxops data]# sed -n '/^#/!p' /etc/fstab 
        
        UUID=dfd00454-d3b4-475f-aacb-64e27e5377f9 /                       ext4    defaults        1 1
        UUID=cde18fcc-03b4-4c90-a235-512769ec23bc /boot                   ext4    defaults        1 2
        UUID=5596e69d-a88a-498c-b821-f0f114a368d6 none                    swap    defaults        0 0
        ```
        
      * `s///替换标记`: 查找替换，支持使用其他分隔符，s@@@，s###

        * 替换标记：
          * `g`: 行内全局替换
          * `p`: 显示替换成功的行
          * `w /PATH/FILE`: 将替换成功的行保存到文件中

        ```bash
        # 将ab替换成xyz
        [root@mylinuxops data]# echo abc | sed 's/ab/xyz/'
        xyzc
        
        # 使用查找替换后+向引用，留下需要的内容 
        [root@mylinuxops data]# echo abc | sed -r 's/(a).*/\1/'
        a
        [root@mylinuxops data]# echo abc | sed -r 's/(a)(b)(c)/\3/'
        c
        
        # 在GRUB_CMDLINE_LINUX行最后插入" xyz"
        [root@mylinuxops data]# sed -r 's/(.*LINUX.*)(")/\1 xyz\2/' /etc/default/grub
        GRUB_TIMEOUT=5
        GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
        GRUB_DEFAULT=saved
        GRUB_DISABLE_SUBMENU=true
        GRUB_TERMINAL="serial console"
        GRUB_SERIAL_COMMAND="serial --speed=115200"
        GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 rhgb quiet crashkernel=auto resume=UUID=5596e69d-a88a-498c-b821-f0f114a368d6 console=ttyS0,115200n8 xyz"
        GRUB_DISABLE_RECOVERY="true"
        GRUB_ENABLE_BLSCFG=true
        
        # 取出基名和路径名
        [root@mylinuxops data]# echo /etc/sysconfig/network-scripts/ | sed -r 's@(.*/)([^/]+)/?$@\1@'
        /etc/sysconfig/
        [root@mylinuxops data]# echo /etc/sysconfig/network-scripts/ | sed -r 's@(.*/)([^/]+)/?$@\2@'
        network-scripts/
        
        # 取出ip地址
        [root@mylinuxops data]# ifconfig | sed -rn '2s/^.* (.+) net.*$/\1/p'
        172.16.11.61 
        [root@mylinuxops ~]# ifconfig | sed -nr '2s/[^0-9]+([0-9.]+) .*/\1/p'
        172.16.11.61
        
        # &符号的使用，用来代替搜索出的内容。
        [root@mylinuxops ~]# sed -n 's/root/&er/p' /etc/passwd
        rooter:x:0:0:root:/root:/bin/bash
        operator:x:11:0:operator:/rooter:/sbin/nologin
        ```

#### `sed`高级编辑命令

* `P`: 打印模式空间开端至\n内容，并追加到默认输出之前
* `h`: 把模式空间中的内容覆盖至保持空间中
* `H`：把模式空间中的内容追加至保持空间中
* `g`: 从保持空间取出数据覆盖至模式空间
* `G`：从保持空间取出内容追加至模式空间
* `x`: 把模式空间中的内容与保持空间中的内容进行互换
* `n`: 读取匹配到的行的下一行覆盖至模式空间
* `N`: 读取匹配到的行的下一行追加至模式空间
* `d`: 删除模式空间中的行
* `D`: 如果模式空间包含换行符，则删除直到第一个换行符的模式空间中的文本，并不会读取新的输入行，而使用合成的模式空间重新启动循环。如果模式空间不包含换行符，则会像发出d命令那样启动正常的新循环

##### 高级编辑示例

```bash
# 打印偶数行
sed -n 'n;p' FILE
# 逆序输出
sed '1!G;h;$!d' FILE
sed 'N;D' FILE
sed '$!N;$!D' FILE
sed '$!d' FILE
sed 'G' FILE
sed 'g' FILE
sed '/^$/d;G' FILE
sed 'n;d' FILE
sed -n '1!G;h;$p' FILE
```

