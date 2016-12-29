---
title: shell script语法
date: 2016-11-27 18:00:18
categories: "Shell"
tags: ["shell script"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Shell.png)
<!-- more -->
现在脚本语言应用较广的就是Shell脚本语言和Python脚本语言，我这个地方会列举一些自己在实际项目中应用的Shell script语法。（持续更新中。。。）

列举一些常用简单的Shell命令，以后用到的都整理到这里。

#### date和ago
写法一：
``` bash
date -d"1 day ago" +"%F %H:%M:%S"
2016-10-27 13:53:04

```
写法二：
``` bash
date --date="1 day ago" +"%F %H:%M:%S"
2016-10-25 14:11:52
```
