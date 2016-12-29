---
title: Java bug系列三
date: 2016-12-01 01:05:54
tags: ["Java", "bug"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java-bug.png)
<!--more-->
今天在Myeclipse导入本地项目时之后，启动的时候出现了一个问题，把这个问题的解决方法记录一下。
import之后出现了下面的问题：
```bash
Caused by: java.lang.UnsupportedClassVersionError: org/apache/commons/dbcp2/BasicDataSource : Unsupported major.minor version 51.0
```
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/java-bug-3/java-bug-3-1.png)

解决方法：
右击工具栏里的windows->Properties->java->Installed JRES,然后添加JDK1.7或者以上的版本。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/java-bug-3/java-bug-3-2.png)
然后再运行一下，效果如下：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/java-bug-3/java-bug-3-3.png)


原因：
dbcp2需要JDK版本至少是jdk1.7。
