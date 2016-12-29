---
title: 深入理解hadoop
date: 2016-11-06 22:25:41
categories: "大数据"
tags: ["hadoop"]
---

![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Hadoop.jpg)
<!-- more -->
发现了一篇好的博文，把它转载到这里，方便以后学习。
### 起源
Hadoop起源于Apache Nutch， 一个开源的网络搜索引擎，也是Apache的Lucene项目的一部分。
Hadoop是创始人Doung Cutting的儿子给一头大象起的名字。
Hadoop的子项目及其后续项目所以用的名称也与其本身的功能多数相关，通常以动物的名字。
一些小的组件，名称通常具有很好的描述性。
MapReduce编程模型的思想来源于函数式编程语言Lisp，由Google公司于2004年提出并首先应用于大型集群。同时，Google也发表了GFS、BigTable等底层系统以应用MapReduce模型。在2007年，Google’s MapReduce Programming Model-Revisted论文发表，进一步详细介绍了Google MapReduce模型以及Sazwall并行处理海量数据分析语言。Google公司以MapReduce作为基石，逐步发展成为全球互联网企业的领头羊。

### hadoop HDFS
HDFS 是一个具有高度容错性的分布式文件系统，它能提供高吞吐量的数据访问，HDFS 的架构如图 2-4 所示，总体上采用了 master/slave 架构，主要由以下几个组件组成 ：Client、 NameNode、 Secondary NameNode 和 DataNode。下面分别对这几个组件进行介绍。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/hadoop-1.png)
1)Client
Client（代表用户）通过与 NameNode 和 DataNode 交互访问 HDFS 中的文件。Client提供了一个类似 POSIX 的文件系统接口供用户调用。

2）NameNode
整个 Hadoop 集群中只有一个 NameNode。它是整个系统的“总管” ，负责管理 HDFS的目录树和相关的文件元数据信息。这些信息是以“fsimage” （HDFS 元数据镜像文件）和 “editlog” （HDFS 文件改动日志）两个文件 形式存放在本地磁盘，当 HDFS 重启时重新构造出来的。此外，NameNode 还负责监控各个 DataNode 的健康状态，一旦发现某个DataNode 宕掉，则将该 DataNode 移出 HDFS 并重新备份其上面的数据。

3）Secondary NameNode
Secondary NameNode 最重要的任务并不是为 NameNode 元数据进行热备份，而是定期合并 fsimage 和 edits 日志，并传输给 NameNode。这里需要注意的是，为了减小 NameNode压力，NameNode 自己并不会合并 fsimage 和 edits，并将文件存储到磁盘上，而是交由Secondary NameNode 完成。

4）DataNode
一般而言，每个 Slave 节点上安装一个 DataNode，它负责实际的数据存储，并将数据信息定期汇报给 NameNode。DataNode 以固定大小的 block 为基本单位组织文件内容，默认情况下 block 大小为 64MB。当用户上传一个大的文件到 HDFS 上时，该文件会被切分成若干个 block，分别存储到不同的 DataNode ；同时，为了保证数据可靠，会将同一个 block以流水线方式写到若干个（默认是 3，该参数可配置）不同的 DataNode 上。这种文件切割后存储的过程是对用户透明的。

### Hadoop MapReduce 
hadoop mapReduce  的主要设计思想包括简化编程接口、高扩展性、提高系统容错性等，其对外提供了5个标准的可编程接口，InputFormat，Mapper，Partitioner，Reducer，OutputFormat。
一、mapReduce 编程实例
mapReduce 能解决的问题往往都有一个共同特征，这类问题能被分解成子任务并行执行，比如top(K) 问题，比如从海量日志中统计词频最高的词，由两个mapReduce完成，一个完成完成分词，一个完成统计。

二、架构设计　　
Hadoop MapReduce 架构也采用了 Master/Slave（M/S）架构，具体如图 2-5所示。它主要由以下几个组件组成 ：Client、JobTracker、 TaskTracker 和 Task。下面分别对这几个组件进行介绍。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/hadoop-2.png)

### 生命周期
Hadoop MapReduce处理的数据一般位于底层的分布式文件系统之中，该系统往往将文件分成若干的block存储在不同的节点上。默认情况下，每个task只处理一个block。MapReduce 主要由4个组件组成，分别是Client、JobTracker、Task、TaskTracker,一个MapReduce作业的运行周期是用户通过client将作业提交到jobTracker上，jobTracker会启动若干个task进行监控和调度，而task的运行环境准备和资源消耗情况则是由taskTracker负责，比如启动单独的jvm保障task运行资源的隔离性，而task的执行进度也是由task先汇报给taskTracker 再由taskTracker 通过RPC汇报给jobTracker的。TaskTracker 周期性地通过 Heartbeat 向 JobTracker 汇报本节点的资源使用情况，一旦出现空闲资源，JobTracker 会按照一定的策略选择一个合适的任务使用该空闲资源，这由任务调度器完成。任务调度器是一个可插拔的独立模块，且为双层架构，即首先选择作业，然后从该作业中选择任务，其中，选择任务时需要重点考虑数据本地性。此外，JobTracker 跟踪作业的整个运行过程，并为作业的成功运行提供全方位的保障。首先，TaskTracker 或者Task 失败时，转移计算任务 ；其次，当某个 Task 执行进度远落后于同一作业的其他 Task 时，为之启动一个相同 Task，并选取计算快的 Task 结果作为最终结果。

### hadoop RPC
网络通信是hadoop的核心模块之一，他支撑了整个Hadoop的上层分布式应用(HBASE、HDFS、MapReduce), Hadoop RPC具有以下几个特性，透明性（用户本身不应该感觉到跨机器调用的细节）、高性能（高吞吐、高并发）、可控性（轻量级、网络链接、超时、缓冲区设计可定制可扩展）。RPC框架实现分为四个层面：

#### 序列化层
这里序列化特指将结构化的对象转为字节流以便在网络中传输或者持久化存储、hadoop 实现了自己的序列化框架（hadoop子项目Avro），良好的序列化框架应该有以下特点，压缩，序列化应该压缩数据以便在网络中传输和存储；可扩展性，理论上结构化对象的属性变化，不影响反序列化；良好的兼容性，支持多语言；高效性，序列化或者反序列化速率高效。java本身也有自己的序列化框架，但是java的序列化框架不够灵活，不能控制序列化的整个流程，序列化算法也不标准，没有做一定的压缩，java序列化首先写类名，然后再是整个类的数据，而且成员对象在序列化中只存引用，成员对象的可以出现的位置很随机，既可以在序列化的对象前，也可以在其后面，这样就对随机访问造成影响，一旦出错，整个后面的序列化就会全部错误。

Avro是个支持多语言的数据序列化框架，支持c，c++，c＃，python，java，php，ruby，java。他的诞生主要是为了弥补Writable只支持java语言的缺陷。很多人会问类似的框架还有Thrift和Protocol，那为什么不使用这些框架，而要重新建一个框架呢，或者说Avro有哪些不同。首先，Avro和其他框架一样，数据是用与语言无关的schema描述的，不同的是Avro的代码生成是可选的，schema和数据存放在一起，而schema使得整个数据的处理过程并不生成代码、静态数据类型等，为了实现这些，需要假设读取数据的时候模式是已知的，这样就会产生紧耦合的编码，不再需要用户指定字段标识。

Avro的schema是JSON格式的，而编码后的数据是二进制格式（当然还有其他可选项）的，这样对于已经拥有JSON库的语言可以容易实现。

Avro还支持扩展，写的schema和读的schema不一定要是同一个，也就是说兼容新旧schema和新旧客户端的读取，比如新的schema增加了一个字段，新旧客户端都能读旧的数据，新客户端按新的schema去写数据，当旧的客户端读到新的数据时可以忽略新增的字段。

Avro还支持datafile文件，schema写在文件开头的元数据描述符里，Avro datafile支持压缩和分割，这就意味着可以做Mapreduce的输入。

#### 函数调用层
函数调用层的主要功能是定位要调用的函数并执行该函数、Hadoop RPC 采用反射和动态代理来实现函数调用。

#### 网络传输层
网络传输层主要描述了server与client之间的消息传输方式、Hadoop RPC 采用了基于TCP/IP的socket调用。

#### 服务端处理框架
服务端处理框架可被抽象为网络I/O 模型，他的设计直接影响PRC的处理能力，常见的网路I/O模型分为，阻塞、非阻塞、事件驱动等、Hadoop RPC的服务端处理采用了基于Reactor模式的事件驱动I/O模型。

Hadoop多用户作业调度器
　　hadoop 最初是为批处理作业设计的，当时只采用了一个简单的FIFO调度机制分配任务，随着hadoop的普及以及应用的用户越来越多，基于FIFO的单用户调度机制不能很好的利用集群资源（比如机器学习和数据挖掘对处理耗时要求不高但I/O密集，生产性作业队实时要求高，如Hive查询统计CPU密集，即不同的作业类型对资源要求不一致），多用户调度器势在必行。多用户调度主要有两种思路，一种是在物理集群上虚拟出多个hadoop集群，优点是实现简单，缺点是集群管理麻烦、调度资源浪费，典型代表HOD(Hadoop on Demand);另一种是扩展Hadoop调度器，使之支持多队列多用户调度，典型代表是capacity scheduler 和fair scheduler。

### hadoop 队列管理机制
hadoop 以队列为单位管理资源，用户只能向一个或多个队列提交作业，队列管理分为两方便：用户权限管理和资源管理。管理员可以配置每个队列的用户和用户组，也可以配置每个队列的管理员，他可以kill队列，改变队列的优先级等；系统支援管理有调度器完成，管理员可以设置各个队列的资源容量参数。

#### capacity Scheduler 
capacity scheduler 主要是由yahoo实现的，主要有以下几个特点：

容量保证：管理员可以设置队列的资源使用上限
灵活性：如果一个队列的资源有剩余，可以共享给其他需要资源的队列，当此队列需要资源时由其他队列归还资源。
多重租赁：支持多用户共享集群和多作业同时运行。
支持资源密集型作业
支持作业优先级
#### hadoop 安全机制
由于hadoop 一般部署在由防火墙隔离的局域网环境之中，hadoop安全机制基本不用与考虑地域外网攻击，更多的是保障多用户在集群环境下安全高效的使用集群资源。Hadoop RPC 采用了SASL(Sample Authentication and Security Layer) 进行安全认证。

#### Kerberos 认证
kerberos 是一种网络安全认证协议，主要概念如下：

客户端（client）：请求服务的用户
服务端（server）：向用户提供服务的一方
秘钥分发中心（kerberos key distribution center,KDC）:中心化的存储了客户端密码和其他账户信息，他接收来自客户端的请求，验证合法性并授予会话凭证，分为认证服务和授权服务。
认证服务(authentication server,AS)：校验用户身份
票据授权服务（Ticket-Granting Service,TGS）:验证由AS颁发的票据，如果票据验证通过，则颁发服务许可票据
票据（Ticket）:用于服务器与用户之间安全的传输信息，同时也附加一些标识
票据授权票据（Ticket-Granting Ticket,TGT）:AS颁发的票据
服务许可票据（Service-Granting Ticket,SGT）：TGS颁发的票据

kerberos 一般采用对称加密方式，认证流程如下图：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/hadoop-3.png)

kerberos相对于SSL的优点：

- kerberos采用对称加密相比SSL非对称加密算法高效。
- 用户管理简单，kerberos基于第三方KDC统一管理，撤销用户只需移出KDC记录，而SSL需要广播给各个服务器

Kerberos协议本身并不能完全解决网络安全性问题，它是建立在一些假定之上的，只有在满足这些假定的环境中它才能正常运行：
- 不能对拒绝服务(Denial of Service)攻击进行防护。Kerberos不能解决拒绝服务攻击，在该协议的很多环节中，攻击者都可以阻断正常的认证步骤。这类攻击只能由管理员和用户来检测和解决。
- 主体必须保证他们的私钥的安全。如果一个入侵者通过某种方法窃取了主体的私钥，他就能冒充身份。
- Kerberos无法应付口令猜测攻击。如果一个用户选择了弱口令，那么攻击者就有可能成功地用口令字典破解掉，继而获得那些由源自于用户口令加密的所有消息。
- 网络上每个主机的时钟必须是松散同步的。这种同步可以减少应用服务器进行重放攻击检测时所记录的数据。松散程度可以以一个服务器为准进行配置。时钟同步协议必须保证自身的安全，才能保证时钟在网上同步。
- 主体的标识不能频繁地循环使用。由于访问控制的典型模式是使用访问控制列表(ACLs)来对主体进行授权。如果一个旧的ACL还保存着已被删除主体的入口，那么攻击者可以重新使用这些被删除的用户标识，就会获得旧ACL中所说明的访问权限。


----------------------------------------------------------------------------------原文转自[这里](http://www.cnblogs.com/LiJianBlog/p/5262170.html)
