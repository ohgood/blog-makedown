---
title: Hive使用指南
date: 2016-10-31 01:19:31
tags: Hive
categories: 大数据
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Hive.png)
<!-- more -->
最近使用Hive来处理工作当中的海量数据,把Hive相关的知识点和平时应用之后的理解整理出来，希望对以后的成长有帮助。


### Hive简介

hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。百度传送门->[戳一下](http://baike.baidu.com/link?url=O6i7dtGxeGe8MTZENt5gaa-RZiKXMovyZjZR908SVH169U57o9CeknHpknp589thHHXFYIi51gl3LT_2LOEBRq)
简单的讲一下运行流程：Hive的执行入口是Driver，执行的SQL语句首先提交到Drive驱动，然后调用compiler解释驱动，最终解释成MapReduce任务去执行。

### 服务端组件

#### 用户接口
Hive 对外提供了三种服务模式，即 Hive 命令行模式（CLI），Hive 的 Web 模式（WUI），Hive 的远程服务（Client）。

1、 Hive 命令行模式 
Hive 命令行模式启动有两种方式。执行这条命令的前提是要配置 Hive 的环境变量。 
进入 /opt//hive 目录，执行如下命令。 
```bash
./hive
```
或者
```bash
hive - -service cli
```
Hive 命令行模式用于 Linux 平台命令行查询，查询语句基本跟 MySQL 查询语句类似，运行结果如下所示:
```bash
[root@hadoop-namenode01 ~]# hive
16/10/31 17:06:17 WARN mapreduce.TableMapReduceUtil: The hbase-prefix-tree module jar containing PrefixTreeCodec is not present.  Continuing without it.

Logging initialized using configuration in jar:file:/opt/cloudera/parcels/CDH-5.8.0-1.cdh5.8.0.p0.42/jars/hive-common-1.1.0-cdh5.8.0.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> 
```

2、 Hive Web 模式(没使用过)
Hive Web 界面的启动命令如下。
```bash
hive - -service hwi
```
通过浏览器访问 Hive，默认端口为 9999。

3 、 Hive的远程服务
Hive 远程服务通过 JDBC 等访问来连接 Hive ，这是程序员最需要的方式。
```bash
hive  --service hiveserver2 &   //默认端口10000
hive --service hiveserver2 --hiveconf hive.server2.thrift.port 10002 &  //可以通过命令行直接将端口号改为10002
```
hive的远程服务端口号也可以在hive-default.xml文件中配置，修改hive.server2.thrift.port对应的值即可。
< property>
    < name>hive.server2.thrift.port< /name>
    < value>10000< /value>
    < description>Port number of HiveServer2 Thrift interface when hive.server2.transport.mode is 'binary'.< /description>
< /property>

使用Hive JDBC连接,以下是一个简单的列子
```bash
package wordcount.api;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import org.apache.log4j.Logger;
import wordcount.model.Word;

public class hiveJdbCli {

	private static String drivernName="org.apache.hive.jdbc.HiveDriver";
	private static String url="jdbc:hive2://192.168.113.100:10000/default";
	private static String user="root";
	private static String password="admin";
	private static String sql="";
	private static ResultSet res = null;
	private static final Logger log = Logger.getLogger(hiveJdbCli.class);
	public static void main(String[] args) throws Exception {

		Connection conn = null;
		Statement stmt = null;
		String tableName = "HiveTest";
		conn = getConn();
		stmt = conn.createStatement();
		getWordCount(stmt);
		//ShowTables(stmt);
//		CreateTable(stmt, tableName);
	}

	public static List<Word> getWordCount(Statement stmt) throws SQLException{

		sql = "select word, count(word) from words group by word";
		res = stmt.executeQuery(sql);
		List<Word> list = new ArrayList<Word>();
 		while (res.next()) {
     		Word word = new Word();
			word.setName(res.getString(1));
			word.setCount(res.getInt(2));
			list.add(word);
		}
		return list;
	};

	private static boolean ShowTables(Statement stmt) throws Exception {

		sql = "show tables";
		res = stmt.executeQuery(sql);
		while(res.next()) {
			String string = res.getString(1);
			System.out.println(string);
		}
		return true;
	}

	private static boolean CreateTable(Statement stmt, String tableName) throws Exception{

		sql = "create table " 
			+ tableName 
			+ "(key int, value string)"
			+ "row format delimited "
			+ "fields terminated by '\t'";
		return stmt.execute(sql);
	}

	private static Connection getConn() throws Exception{

		Class.forName(drivernName);
		Connection conn = DriverManager.getConnection(url, user, password);
		return conn;
	}
}
```

假如我们使用的是框架开发,只需修改连接数据库的配置文件即可。
```bash
jdbc.driver=org.apache.hive.jdbc.HiveDriver
jdbc.url=jdbc:hive2://192.168.113.24:10000/result
jdbc.user=root
jdbc.password=admin
```

#### MetaStore组件(元数据存储)
将元数据存储在 RDBMS 中，一般常用 MySQL 和 Derby，支持把metastore独立出来放在远程的集群上面，使得hive更加健壮。默认情况下，Hive 元数据保存在内嵌的 Derby 数据库中，只能允许一个会话连接，只适合简单的测试。实际生产环境中不适用， 为了支持多用户会话，则需要一个独立的元数据库，使用 MySQL 作为元数据库，Hive 内部对 MySQL 提供了很好的支持。
所以安装Hive时，需要先配置一个独立的元数据库，元数据主要包括了表的名称、表的列、分区和属性、表的属性（是不是外部表等等）、表的数据所在的目录。安装方法就不在这里介绍了。

#### Driver组件
该组件包括：Compiler、Optimizer、Executor,它可以将Hive的编译、解析、优化转化为MapReduce任务提交给JobTracker或者是SourceManager来进行实际的执行相应的任务。

### Hive原理
流程大致步骤为:
1.用户提交查询等任务给Driver。
2.编译器获得该用户的任务Plan。
3.编译器Compiler根据用户任务去MetaStore中获取需要的Hive的元数据信息。
4.编译器Compiler得到元数据信息，对任务进行编译，先将HiveQL转换为抽象语法树，然后将抽象语法树转换成查询块，将查询块转化为逻辑的查询计划，重写逻辑查询计划，将逻辑计划转化为物理的计划（MapReduce）, 最后选择最佳的策略。
5.将最终的计划提交给Driver。
6.Driver将计划Plan转交给ExecutionEngine去执行，获取元数据信息，提交给JobTracker或者SourceManager执行该任务，任务会直接读取HDFS中文件进行相应的操作。
7.获取执行的结果。
8.取得并返回执行结果。

1)解析器（parser）：将查询字符串转化为解析树表达式。
2)语义分析器（semantic analyzer）：将解析树表达式转换为基于块（block-based）的内部查询表达式，将输入表的模式（schema）信息从 metastore 中进行恢复。用这些信息验证列名， 展开 SELECT * 以及类型检查（固定类型转换也包含在此检查中）。
3)逻辑策略生成器（logical plan generator）：将内部查询表达式转换为逻辑策略，这些策略由逻辑操作树组成。
4)优化器（optimizer）：通过逻辑策略构造多途径并以不同方式重写。优化器的功能如下。
将多 multiple join 合并为一个 multi-way join；
对join、group-by 和自定义的 map-reduce 操作重新进行划分；
消减不必要的列；
在表扫描操作中推行使用断言（predicate）；
对于已分区的表，消减不必要的分区；
在抽样（sampling）查询中，消减不必要的桶。

### Hive安装
1.先安装Mysql，用来存储元数据。传送门->[戳一下](http://jingyan.baidu.com/article/a378c9609eb652b3282830fd.html)
2.下载Hive安装包并解压安装。传送门->[戳一下](http://mirrors.cnnic.cn/apache/hive/)
3.配置Hive：
a. 进入conf目录  cp hive-default.xml.template hive-site.xml
b. hive-site.xml配置  
```bash
<configuration>
	<property>
	    <name>javax.jdo.option.ConnectionURL</name>
	    <value>jdbc:mysql://master00:3306/hive?createDatabaseIfNotExist=true</value>
	    <description>JDBC connect string for a JDBC metastore</description>
	</property>
	<property>
	    <name>javax.jdo.option.ConnectionDriverName</name>
	     <value>com.mysql.jdbc.Driver</value>
	     <description>Driver class name for a JDBC metastore</description>
	</property>
	 <property>
	    <name>javax.jdo.option.ConnectionUserName</name>
	    <value>root</value>
	    <description>Username to use against metastore database</description>
	 </property>
	 <property>
	 	<name>javax.jdo.option.ConnectionPassword</name>
	    <value>admin</value>
	    <description>password to use against metastore database</description>
	 </property>
</configuration>
 ```
c. 拷贝驱动包到hive/lib目录下 cp mysql-connector-java-5.1.39-bin.jar  ../../softwares/hive/lib/
d. 启动hadoop
e. 将hive/lib目录下最新的jline-2.12.jar包拷贝到/hadoop/share/hadoop/yarn/lib/目录下，将原有的jline.jar备份
f. 启动hive测试运行。
```bash
[root@hadoop-namenode01 ~]# hive
16/11/01 10:13:58 WARN mapreduce.TableMapReduceUtil: The hbase-prefix-tree module jar containing PrefixTreeCodec is not present.  Continuing without it.
Logging initialized using configuration in jar:file:/opt/cloudera/parcels/CDH-5.8.0-1.cdh5.8.0.p0.42/jars/hive-common-1.1.0-cdh5.8.0.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> 
```
g. 安装完毕

### 常用的HQL(例举一些工作常用的)
#### 创建HIVE表
create table yt_daily_install(mac_int int,add_time string) partitioned by(dt string);

#### 创建临时表并该表添加数据
create temporary table rate SELECT round(c.num/d.mac_int, 2) as r, c.date as t FROM day_keep_41 AS c, yt_daily_install AS d WHERE c.date = d.add_time;

#### 插入Hive表
 insert into(overwrite) result.yt_daily_install partition (dt="2016.04-08") select count(b.mac),to_date(b.add_time) from (select * from (select mac, add_time,prod_id, ROW_NUMBER() over(partition by mac order by add_time) as new_index from yt_data.yt_mac_copy) a where a.new_index=1) b where prod_id="41" group by to_date(b.add_time);
两者的区别：
insert overwrite 会覆盖已经存在的数据，我们假设要插入的数据和已经存在的N条数据一样，那么插入后只会保留一条数据；
insert into 只是简单的copy插入，不做重复性校验，如果插入前有N条数据和要插入的数据一样，那么插入后会有N+1条数据；

#### 为表新增字段
alter table day_keep_41 add retention_rate varchar(25);
