---
title: Shell自动化脚本
date:  2015-10-20
categories: "Shell"
tags: ["shell script", "Shell自动化脚本"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Shell.png)
<!-- more -->
### Shell以及Shell脚本语言（Shell Script）简介
Shell 是一个用C语言编写的程序，是系统跟计算机硬件交互时使用的中间介质在shell和计算机硬件之间还有一层东西那就是系统内核了。打个比方，如果把计算机硬件比作一个人的躯体，而系统内核则是人的大脑，至于shell，把它比作人的五官似乎更加贴切些。回到计算机上来，用户直接面对的不是计算机硬件而是shell，用户把指令告诉shell，然后shell再传输给系统内核，接着内核再去支配计算机硬件去执行各种操作。。Shell既是一种命令语言，又是一种程序设计语言。

Shell 脚本（shell script），是一种为shell编写的脚本程序。业界所说的shell通常都是指shell脚本，所以注意的是shell和shell script是两个不同的概念。
### 一个简单的列子

``` bash
[root@hadoop-namenode01 nginxlogs]# vi import_mairuanlog.sh 

# !/bin/sh
# upload www.mairuan.com.log to hive table

yestoday=`date --date='1 days ago' +%Y%m%d`
today_m=`date --date='0 days ago' +%Y%m`
today_d=`date --date='0 days ago' +%d`

hive -S -e "
use nginxlogs;
create table if not exists nginxlogs_temp(
loginfo string
)
row format delimited
fields terminated by '\001';
load data inpath '/flume/nginxlogs/www.mairuan.com/$today_m/$today_d' overwrite into table nginxlogs_temp;
"

# parse logs and upload data into table nginxlogs_src

hive -S -e "
use nginxlogs;
create table if not exists nginxlogs_mairuan_src(
timestamp string,
source string,
client string,
size string,
responsetime string,
upstreamtime string,
upstreamaddr string,
request_method string,
domain string,
url string,
http_user_agent string,
status string,
x_forwarded_for string
)
partitioned by (date string)
row format delimited
fields terminated by '\001'
stored as textfile;

```

### 设置自动执行的时间
#### 打开系统文件
``` bash
$ vim /etc/crontab
```
#### 文件信息显示如下：

``` bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59) 
# |  .------------- hour (0 - 23)   
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```
 
#### 设置每天自动执行的时间（分别对应相应的时间）

``` bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)  --指定自动执行时的分钟
# |  .------------- hour (0 - 23)  --指定自动执行时的小时，如果参数有两个用","隔开
# |  |  .---------- day of month (1 - 31)   --指定每个月几号执行，如果是每天则为"*"
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ... --指定哪些月份执行
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
  0  3  *  *  * root sh /shell/nginxlogs/mairuan.com.sh
  0  4  *  *  * root sh /shell/nginxlogs/mairuan.copylog.sh
 10  4  *  *  * root sh /shell/nginxlogs/import_mairuanlog.sh

```
#### 立即生效

``` bash
source /etc/crontab
```
这样，脚本在时间到了就可以自动了。
