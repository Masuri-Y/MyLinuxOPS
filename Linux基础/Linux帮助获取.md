---
title: Linux帮助获取
date: 2019-03-04 12:33:34
categories: Linux基础
tags:
---

Linxu帮助获取方法有许多种类,在获取帮助信息时，内部命令和外部命令的获取方式是有区别的：  

<!-- more -->

内部命令：  

```
help COMMAND
```

外部命令：有以下几种途径  

```bash
1. 通过命令自带的帮助信息  
   COMMAND --help  
   COMMAND -h
2. 使用手册(manual)  
   man COMMAND
3. 信息页
   info COMMAND 支持信息页中的超链接。
4. 程序自身的帮助文档，有README、INSTALL、Changelog等。  
   此类文档目录：/usr/share/doc
5. 程序的官方文档
6. 发行版官方文档
7. google
```

### ***man命令的使用方法***

命令格式：  

```
   man [章节] COMMAND  
```

章节共有9个每个章节代表不同的内容  

| 章节号 | 内容                                          |
| :----: | :-------------------------------------------- |
|   1    | 用户命令（使用者在shell环境中可以操作的指令） |
|   2    | 系统调用（系统核心可以调用的函数和工具）      |
|   3    | c库调用（常用的函数和函数库）                 |
|   4    | 设备文件及特殊文件                            |
|   5    | 配置文件格式                                  |
|   6    | 游戏                                          |
|   7    | 杂项                                          |
|   8    | 管理类的命令                                  |
|   9    | Linux内核API                                  |

man的内容分成几个部分   

| 名称        | 说名                               |
| :---------- | :--------------------------------- |
| NAME        | 简单的指令名称及说明               |
| SYNOPSIS    | 指令的语法                         |
| DESCRIPTION | 指令的完整说明                     |
| OPTIONS     | 指令的相关选项及说明               |
| COMMANDS    | 程序在执行时可下达的指令           |
| FILES       | 这个程序资料可以参考的其他档案     |
| SEE ALSO    | 一些可以参考的和指令相关的其他内容 |
| EXAMPLE     | 一些可以参考的范例                 |

man命令的一些简单操作方法  

| 按键  | 说明             |
| :---- | :--------------- |
| 空格  | 向文件尾部翻一屏 |
| b     | 向文件首部翻一屏 |
| d     | 向文件尾部翻半屏 |
| u     | 向文件首部翻半屏 |
| j     | 向文件尾部翻一行 |
| k     | 向文件首部翻一行 |
| q     | 退出             |
| 数字# | 跳转到第#行      |
| 1G    | 跳转到文件首部   |
| G     | 跳转到文件尾部   |

man文件内搜索关键字的方法  

| 按键     | 说明                                                        |
| :------- | :---------------------------------------------------------- |
| /KEYWORD | 在文档内从当前位置向尾部搜索KEYWORD关键字，并且不区分大小写 |
| ?KEYWORD | 在文档内从当前位置向首部搜索KEYWORD关键字，兵器不区分大小写 |
| n        | 按照与搜索关键字方向相同的方法，查找下一个关键字            |
| N        | 按照与搜索关键字方向相反的方向，查找下一个关键字            |

man命令的一些相关选项  

| 选项       | 说明                              |
| ---------- | --------------------------------- |
| -f COMMAND | 搜索系统中哪些与COMMAND相关的章节 |
| -k COMMAND | 搜索与关键字有关的帮助文件        |

示例：

```bash
[root@centos7 ~]# man -f passwd   此命令同whatis passwd
passwd (1)           - update user's authentication tokens
sslpasswd (1ssl)     - compute password hashes
passwd (5)           - password file

[root@centos7 ~]# man -k passwd    
chpasswd (8)         - update passwords in batch mode
fgetpwent_r (3)      - get passwd file entry reentrantly
getpwent_r (3)       - get passwd file entry reentrantly
gpasswd (1)          - administer /etc/group and /etc/gshadow
grub2-mkpasswd-pbkdf2 (1) - Generate a PBKDF2 password hash.
lpasswd (1)          - Change group or user password
```

注意：
使用-k,-f选项时首先需要建立资料库才行，此时需要执行mandb(Centos7),makewhatis(makewhatis)

