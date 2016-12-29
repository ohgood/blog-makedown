---
title: Mysql性能优化方法概述
date: 2016-11-24 22:08:05
categories: ["Mysql"]
tags: ["Mysql", "数据库性能"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Mysql.jpg)
<!--more-->

一：优化sql和索引,我之前写了一篇博客，可以看下--->>[传送门](https://ohgood.github.io/2016/11/09/Mysql%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96-%E7%B4%A2%E5%BC%95/
)

二：使用memcached,redis来当缓存，之前写了两篇缓存入门博客，可以看下--->>[传送门](https://ohgood.github.io/2016/11/27/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8redis%E4%BD%9C%E4%B8%BAMysql%E7%9A%84%E7%BC%93%E5%AD%98/)和[传送门](http://tonywu.me/2016/12/02/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8memcached%E4%BD%9C%E4%B8%BAMysql%E7%9A%84%E7%BC%93%E5%AD%98%E5%85%A5%E9%97%A8%E7%AF%87/)

三：做主从复制或主主复制，读写分离

四：创建分区表

五：垂直拆分

六：水平拆分


