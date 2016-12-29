---
title: flume-ng
date: 2016-10-26 02:04:57
categories: "大数据"
tags: ["flume-ng", "hadoop"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Flume.png)
<!-- more -->
### 什么是flume-ng

Flume NG是Cloudera提供的一个分布式、可靠、可用的系统，它能够将不同数据源的海量日志数据进行高效收集、聚合、移动，最后存储到一个中心化数据存储系统中。由原来的Flume OG到现在的Flume NG，进行了架构重构，并且现在NG版本完全不兼容原来的OG版本。经过架构重构后，Flume NG更像是一个轻量的小工具，非常简单，容易适应各种方式日志收集，并支持failover和负载均衡。
注意：Flume 使用 java 编写，其需要运行在 Java1.6 或更高版本之上。

### flume-ng是怎么工作的

#### 几个核心概念
> * Event：一个数据单元，带有一个可选的消息头, 可以是日志、avro对象等
> * Flow：Event从源点到达目的点的迁移的抽象
> * Client：操作位于源点处的Event，将其发送到Flume Agent
> * Agent：一个独立的Flume进程，包含组件Source、Channel、Sink
> * Source：用来消费传递到该组件的Event，简单讲就是收集数据
> * Channel：中转Event的一个临时存储，保存有Source组件传递过来的Event
> * Sink：从Channel中读取并移除Event，将Event传递到Flow Pipeline中的下一个Agent（如果有的话）

#### 数据流模型
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/flume-ng-1.png)
Flume的数据流由事件(Event)贯穿始终。事件是Flume的基本数据单位，它携带日志数据(字节数组形式)并且携带有头信息，这些Event由Agent外部的Source。当Source捕获事件后会进行特定的格式化，然后Source会把事件推入(单个或多个)Channel中。你可以把Channel看作是一个缓冲区，它将保存事件直到Sink处理完该事件。Sink负责持久化日志或者把事件推向另一个Source。
很直白的设计，其中值得注意的是，Flume提供了大量内置的Source、Channel和Sink类型。不同类型的Source,Channel和Sink可以自由组合。组合方式基于用户设置的配置文件，非常灵活。比如：Channel可以把事件暂存在内存里，也可以持久化到本地硬盘上。Sink可以把日志写入HDFS, HBase，甚至是另外一个Source等等。

对于直接读取文件Source, 主要有两种方式： 

Exec source

可通过写Unix command的方式组织数据，最常用的就是tail -F [file]。
可以实现实时传输，但在flume不运行和脚本错误时，会丢数据，也不支持断点续传功能。因为没有记录上次文件读到的位置，从而没办法知道，下次再读时，从什么地方开始读。特别是在日志文件一直在增加的时候。flume的source挂了。等flume的source再次开启的这段时间内，增加的日志内容，就没办法被source读取到了。不过flume有一个execStream的扩展，可以自己写一个监控日志增加情况，把增加的日志，通过自己写的工具把增加的内容，传送给flume的node。再传送给sink的node。要是能在tail类的source中能支持，在node挂掉这段时间的内容，等下次node开启后在继续传送，那就更完美了。
Spooling Directory Source

SpoolSource:是监测配置的目录下新增的文件，并将文件中的数据读取出来，可实现准实时。需要注意两点：1、拷贝到spool目录下的文件不可以再打开编辑。2、spool目录下不可包含相应的子目录。在实际使用的过程中，可以结合log4j使用，使用log4j的时候，将log4j的文件分割机制设为1分钟一次，将文件拷贝到spool的监控目录。log4j有一个TimeRolling的插件，可以把log4j分割的文件到spool目录。基本实现了实时的监控。Flume在传完文件之后，将会修改文件的后缀，变为.COMPLETED（后缀也可以在配置文件中灵活指定） 
ExecSource，SpoolSource对比：ExecSource可以实现对日志的实时收集，但是存在Flume不运行或者指令执行出错时，将无法收集到日志数据，无法何证日志数据的完整性。SpoolSource虽然无法实现实时的收集数据，但是可以使用以分钟的方式分割文件，趋近于实时。如果应用无法实现以分钟切割日志文件的话，可以两种收集方式结合使用。 

### 安装以及配置
1.先安装JDK，这里就不介绍了，可以去看下百度[传送门](http://jingyan.baidu.com/article/d621e8dae805272865913fa7.html)。
2.去官网下载安装包，[传送门](http://flume.apache.org/download.html)
3.解压	apache-flume-1.6.0-bin.tar.gz
``` bash
	tar zxvf apache-flume-1.6.0-bin.tar.gz 
```
  解压 apache-flume-1.6.0-src.tar.gz， 然后把此文件内的内容替换掉apache-flume-1.7.0-bin里的bin文件

``` bash
	tar zxvf apache-flume-1.6.0-src.tar.gz
	cp -ri apache-flume-1.6.0-src/* apache-flume-1.6.0-bin/bin
```
4.配置环境变量
``` bash 
	export FLUME_HOME=/opt/softwares/flume/apache-flume-1.6.0-bin
	export FLUME_CONF_DIR=$FLUME_HOME/conf
	export PATH=$PATH:$FLUME_HOME/bin
```
5.配置环境变量立即生效
``` bash 
	source /etc/profile
```

6.测试是否安装成功
``` bash 
	flume-ng version
```
出现如下信息表示安装成功：
``` bash
Flume 1.7.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: 2561a23240a71ba20bf288c7c2cda88f443c2080
Compiled by hshreedharan on Mon May 11 11:15:44 PDT 2015
From source with checksum b29e416802ce9ece3269d34233baf43f
```

### 一个简单的例子(单节点flume写入HDFS文件)

``` bash 
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source
a1.sources.r1.type = spooldir
a1.sources.r1.fileSuffix = .COMPLETED
a1.sources.r1.deletePolicy = never
a1.sources.r1.channels = c1
a1.sources.r1.spoolDir = /home/data/logs --监控的文件目录
# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = hdfs://192.168.113.24:8020/test/flume/ --指定HDFS的服务器地址
a1.sinks.k1.hdfs.filePrefix = SequenceFile
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动flume agent a1 

``` bash
/home/hadoop/flume-1.6.0-bin/bin/flume-ng agent -c . -f /home/hadoop/flume-1.6.0-bin/conf/spool.conf -n a1 -Dflume.root.logger=INFO,console
```

在/home/data/logs创建一个新文件，然后查看consle信息：
``` bash 
16/10/26 20:16:00 INFO avro.ReliableSpoolingFileEventReader: Preparing to move file /home/data/logs/test_hdfs.conf to /home/data/logs/test_hdfs.conf.COMPLETED
16/10/26 20:16:00 INFO hdfs.HDFSSequenceFile: writeFormat = Writable, UseRawLocalFileSystem = false
16/10/26 20:16:01 INFO hdfs.BucketWriter: Creating hdfs://192.168.113.24:8020/test/flume//SequenceFile.1477538160587.tmp
16/10/26 20:16:01 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
16/10/26 20:16:35 INFO hdfs.BucketWriter: Closing hdfs://192.168.113.24:8020/test/flume//SequenceFile.1477538160587.tmp
16/10/26 20:16:35 INFO hdfs.BucketWriter: Renaming hdfs://192.168.113.24:8020/test/flume/SequenceFile.1477538160587.tmp to hdfs://192.168.113.24:8020/test/flume/SequenceFile.1477538160587	
```

然后我们可以在服务器上看下hfds文件是否生成：
``` bash 
[root@hadoop-namenode01 flume]# hdfs dfs -ls /test/flume
Found 1 items
-rw-r--r--   1 root root        118 2016-10-27 11:16 /test/flume/SequenceFile.1477538160587
```
OK,已经成功了。
