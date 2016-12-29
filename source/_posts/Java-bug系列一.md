---
title: Java bug系列一
date: 2016-11-15 03:55:09
tags: ["Java", "bug"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java-bug.png)
<!--more-->
今天从SVN上拉项目出现了一个问题，然后把这个问题的解决方法记录一下。
我在maven install的时候出现了下面的问题：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/java-bug1.png)

解决方法：
点击Eclipse菜单：Window->Preferences->Java->InstalledJREs,选择右侧的Edit来修改JREs,之前我的JREhome设置的是：C:\ProgramFiles\Jave\jre6，这个位置下是没有tools.jar包的，修改成：C:\ProgramFiles\Java\jdk1.6.0_25，然后点击弹出窗口的Finish按钮和主页面中的OK按钮，则问题解决。
然后再maven install一下，结果如下：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img//java-bug2.png)
原因：
关于tools.jar的作用，认为是用来生成javadoc的包，但是我们平时开发的时候是不会用到这个包的，因为我们开发不许要生成javadoc的这个功能，所以在我们eclipse中配置的jre没有添加tools.jar包，所以这个时候如果我们的项目中需要对项目生成javadoc，比如这里我们使用了maven中生成javadoc的报告，这个时候我们有必要把这个tools.jar的包加入到项目中，没有添加的时候，在运行maven的install的时候将会报错。
