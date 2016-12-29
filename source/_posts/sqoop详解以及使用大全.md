---
title: sqoop详解以及使用大全
date: 2016-10-30 23:12:54
tags: ["sqoop", "hadoop"]
categories: 大数据
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Hadoop.jpg)
<!-- more -->
### sqoop简介

Sqoop是一个用来将Hadoop和关系型数据库中的数据相互转移的工具，可以将一个关系型数据库（例如 ： MySQL ,Oracle ,PostgreSQL等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。利用Mapreduce分布式批处理，加快了数据传输速度，保证了容错性。百度传送门->[戳一下](http://baike.baidu.com/link?url=ew7-Ry9dCX56iq3nrqd8r5QVvP2biEwWKZcM8kFg7JAc4c9CTIUsK5H-MYP6S3fUCxthuYppDOT3bMZIrSy87_)

### sqoop原理

#### sqoop1 import原理
从传统数据库获取元数据信息(schema、table、field、field type)，把导入功能转换为只有Map的Mapreduce作业，在mapreduce中有很多map，每个map读一片数据，进而并行的完成数据的拷贝。

#### sqoop1 export原理
获取导出表的schema、meta信息，和Hadoop中的字段match；多个map only作业同时运行，完成hdfs中数据导出到关系型数据库中。

### sqoop方法

#### export
从HDFS中导出数据到关系型数据库

sqoop export --connect jdbc:mysql://192.168.113.24:3306/res --username root --password admin --table yt_daily_install --export-dir /user/hive/warehouse/result.db/yt_daily_install/dt=2016.04-08 --input-fields-terminated-by '\001' ;

#### import
将数据库表的数据导入到hive中，如果在hive中没有对应的表，则自动生成与数据库表名相同的表。
sqoop import –connect jdbc:mysql://localhost:3306/hive –username root –password

123456 –table user –split-by id –hive-import

#### create-hive-table

生成与关系数据库表的表结构对应的HIVE表

基础语句：

sqoop create-hive-table –connect jdbc:mysql://localhost:3306/hive -username root -password 123456 –table TBLS –hive-table h_tbls2


#### eval

可以快速地使用SQL语句对关系数据库进行操作，这可以使得在使用import这种工具进行数据导入的时候，可以预先了解相关的SQL语句是否正确，并能将结果显示在控制台。

查询示例：

sqoop eval –connect jdbc:mysql://localhost:3306/hive -username root -password 123456 -query “SELECT * FROM tbls LIMIT 10″

数据插入示例：

sqoop eval –connect jdbc:mysql://localhost:3306/hive -username root -password 123456 -e “INSERT INTO TBLS2

VALUES(100,1375170308,1,0,’Hadoop’,0,1,’guest’,’MANAGED_TABLE’,’abc’,’ddd’)”

-e、-query这两个参数经过测试，比如后面分别接查询和插入SQL语句，皆可运行无误，如上。


#### codegen

将关系数据库表映射为一个Java文件、Java class类、以及相关的jar包，作用主要是两方面：

1、  将数据库表映射为一个Java文件，在该Java文件中对应有表的各个字段。

2、  生成的Jar和class文件在metastore功能使用时会用到。

基础语句：

sqoop codegen –connect jdbc:MySQL://localhost:3306/Hive –username root –password 123456 –table TBLS2


#### import-all-tables

将数据库里的所有表导入到HDFS中，每个表在hdfs中都对应一个独立的目录。

sqoop import-all-tables –connect jdbc:mysql://localhost:3306/test

sqoop import-all-tables –connect jdbc:mysql://localhost:3306/test –hive-import


#### job

用来生成一个sqoop的任务，生成后，该任务并不执行，除非使用命令执行该任务。

sqoop job


#### list-tables

打印出关系数据库某一数据库的所有表名

sqoop list-tables –connect jdbc:mysql://localhost:3306/zihou -username root -password 123456


#### list-databases

打印出关系数据库所有的数据库名

sqoop list-databases –connect jdbc:mysql://localhost:3306/ -username root -password 123456


#### merge

将HDFS中不同目录下面的数据合在一起，并存放在指定的目录中，示例如：

sqoop merge –new-data /test/p1/person –onto /test/p2/person –target-dir /test/merged –jar-file /opt/data/sqoop/person/Person.jar –class-name Person –merge-key id

其中，–class-name所指定的class名是对应于Person.jar中的Person类，而Person.jar是通过Codegen生成的


#### metastore

记录sqoop job的元数据信息，如果不启动metastore实例，则默认的元数据存储目录为：~/.sqoop，如果要更改存储目录，可以在配置文件sqoop-site.xml中进行更改。

metastore实例启动：sqoop metastore


#### version

显示sqoop版本信息

语句：sqoop version


#### help

打印sqoop帮助信息

语句：sqoop help
