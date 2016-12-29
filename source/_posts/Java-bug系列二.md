---
title: Java bug系列二
date: 2016-11-30 23:06:51
tags: ["Java", "bug"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java-bug.png)
<!--more-->
今天在Myeclipse导入本地项目时出现了一个问题，把这个问题的解决方法记录一下。
import之后出现了下面的问题：
```bash
Java compiler level does not match the version of the installed Java project.
```

解决方法：
项目右击：Properties->Myeclipse->JavaScript->Project Facets,右侧窗口中的“Java”下拉列表中，选择相应的版本。记得点击“Apply”按钮。

原因：
项目开发使用的JDK版本和编辑器的JDK版本不一致造成的。
