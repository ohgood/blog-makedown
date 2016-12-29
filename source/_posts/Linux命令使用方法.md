---
title: Linux命令使用方法
date: 2016-11-17 06:47:15
tags: ["Linux命令"]
categories: ["Linux"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Linux.png)
<!--more-->
平时用到的，碰到一个就整理一个。
### find
查找linux系统中名字为hbase的文件:
find / -name "hbase" 
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/Linux命令-1.png)
如果你要找的这个文件是hbase.sh或者你只知道hbase 忘了后面的.sh,则需要以下命令
find / -name "hbase*"

### jps
jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有Java进程pid的命令，简单实用，非常适合在Linux/unix平台上简单察看当前java进程的一些简单情况。
jps存放在JAVA_HOME/bin/jps，使用时为了方便请将JAVA_HOME/bin/加入到Path.
比较常用的参数：
-q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数
-m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
-v 输出传递给JVM的参数

### touch 
使用指令"touch"时，如果指定的文件不存在，则将创建一个新的空白文件。例如，在当前目录下，使用该指令创建一个空白文件"file"，输入如下命令：
$ touch a.txt            #创建一个名为“file”的新的空白文件 
使用指令"touch"修改文件"testfile"的时间属性为当前系统时间，输入如下命令：
$ touch a.txt                 #修改文件的时间属性 
首先，使用ls命令查看testfile文件的属性，如下所示：
$ ls -l a.txt                 #查看文件的时间属性  
执行指令"touch"修改文件属性以后，并再次查看该文件的时间属性
$ touch a.txt                 #修改文件时间属性为当前系统时间  
$ ls -l a.txt                 #查看文件的时间属性  
查看时间对比，图片如下：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/Linxu-touch.png)

### cat 
从键盘创建一个文件。$ cat > b.txt 加入不向往文件输入内容，Ctrl+d,直接退出；让想输入，直接输入内容，Ctrl+d再退出：
```bash
[root@hadoop-namenode01 media]# cat > b.txt
1
2
3
4
```
查看文件内容
```bash
[root@hadoop-namenode01 media]# cat b.txt
1
2
3
4
```
将几个文件合并为一个文件： cat a.txt b.txt > c.txt：
```bash
[root@hadoop-namenode01 media]# cat c.txt 
1
2
3
4
[root@hadoop-namenode01 media]# 

```
### mkdir 
创建文件夹，-p 确保目录名称存在，不存在的就建一个。
```bash
[root@hadoop-namenode01 media]# ls
[root@hadoop-namenode01 media]# mkdir a.txt
[root@hadoop-namenode01 media]# mkdir -p test/a.txt
[root@hadoop-namenode01 media]# ls
a.txt  test
[root@hadoop-namenode01 media]# 

```
