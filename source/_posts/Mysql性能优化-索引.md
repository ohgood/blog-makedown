---
title: Mysql性能优化---索引
date: 2016-11-09 05:28:19
categories: "Mysql"
tags: ["数据库性能优化", "索引","Mysql"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Mysql.jpg)
<!-- more -->
### 索引的介绍
1.MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。
2.在关系数据库中，索引是一种单独的、物理的数对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。
索引提供指向存储在表的指定列中的数据值的指针，然后根据您指定的排序顺序对这些指针排序。数据库使用索引以找到特定值，然后顺指针找到包含该值的行。这样可以使对应于表的SQL语句执行得更快，可快速访问数据库表中的特定信息。
当表中有大量记录时，若要对表进行查询，第一种搜索信息方式是全表搜索，是将所有记录一一取出，和查询条件进行一一对比，然后返回满足条件的记录，这样做会消耗大量数据库系统时间，并造成大量磁盘I/O操作；第二种就是在表中建立索引，然后在索引中找到符合查询条件的索引值，最后通过保存在索引中的ROWID（相当于页码）快速找到表中对应的记录。
3.数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。
4.索引可以提高查询速度，它就相当于字典的目录，可以通过它很快查询到想要的结果，而不需要进行全表扫描。

### 索引的数据结构
详解b+树
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-4.jpg)
如上图，是一颗b+树，关于b+树的定义可以参见B+树，这里只说一些重点，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），如磁盘块1包含数据项17和35，包含指针P1、P2、P3，P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。真实的数据存在于叶子节点即3、5、9、10、13、15、28、29、36、60、75、79、90、99。非叶子节点只不存储真实的数据，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表中。

b+树的查找过程

如图所示，如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。真实的情况是，3层的b+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常高。

b+树性质：
1.通过上面的分析，我们知道IO次数取决于b+数的高度h，假设当前数据表的数据为N，每个磁盘块的数据项的数量是m，则有h=㏒(m+1)N，当数据量N一定的情况下，m越大，h越小；而m = 磁盘块的大小 / 数据项的大小，磁盘块的大小也就是一个数据页的大小，是固定的，如果数据项占的空间越小，数据项的数量越多，树的高度越低。这就是为什么每个数据项，即索引字段要尽量的小，比如int占4字节，要比bigint8字节少一半。这也是为什么b+树要求把真实的数据放到叶子节点而不是内层节点，一旦放到内层节点，磁盘块的数据项会大幅度下降，导致树增高。当数据项等于1时将会退化成线性表。
2.当b+树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+数是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，b+树会优先比较name来确定下一步的所搜方向，如果name相同再依次比较age和sex，最后得到检索的数据；但当(20,F)这样的没有name的数据来的时候，b+树就不知道下一步该查哪个节点，因为建立搜索树的时候name就是第一个比较因子，必须要先根据name来搜索才能知道下一步去哪里查询。比如当(张三,F)这样的数据来检索时，b+树可以用name来指定搜索方向，但下一个字段age的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是F的数据了， 这个是非常重要的性质，即索引的最左匹配特性。

### 建立索引的原则
1.越小的数据类型通常更好：越小的数据类型通常在磁盘、内存和CPU缓存中都需要更少的空间，处理起来更快。
2.简单的数据类型更好：整型数据比起字符，处理开销更小，因为字符串的比较更复杂。在MySQL中，应该用内置的日期和时间数据类型，而不是用字符串来存储时间；以及用整型数据类型存储IP地址。
3.尽量避免NULL：应该指定列为NOT NULL，除非你想存储NULL。在MySQL中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值。
4.尽量选择区分度高的列作为索引,区分度的公式是count(distinct contact_id)/count(contact_id)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1。
5.索引列不能参与计算，保持列“干净”。
6.尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。
7.很多时候用 exists 代替 in 是一个好的选择：select num from a where num in(select num from b)用下面的语句替换：select num from a where exists(select 1 from b where num=a.num)。
8.索引并不是越多越好，索引固然可 以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。
9.查询表时用具体的字段列表代替“*”，不要返回用不到的任何字段。

### 索引语句
ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。
CREATE INDEX可对表增加普通索引或UNIQUE索引。
查看索引
mysql> show index from tblname;
mysql> show keys from tblname;
· Table：表的名称。
· Non_unique：如果索引不能包括重复词，则为0。如果可以，则为1。
· Key_name:索引的名称。
· Seq_in_index:索引中的列序列号，从1开始。
· Column_name:列名称。
· Collation:列以什么方式存储在索引中。在MySQL中，有值‘A’（升序）或NULL（无分类）。
· Cardinality索引中唯一值的数目的估计值。通过运行ANALYZETABLE或myisamchk-a可以更新。基数根据被存储为整数的统计数据来计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL使用该索引的机 会就越大。
· Sub_part:如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL。
· Packed:指示关键字如何被压缩。如果没有被压缩，则为NULL。
· Null:如果列含有NULL，则含有YES。如果没有，则该列含有NO。
· Index_type:用过的索引方法（BTREE, FULLTEXT, HASH, RTREE）。
· Comment：

### B+树索引
数据库中B+树索引分为聚集索引（clustered index）和非聚集索引（secondary index）.这两种索引的共同点是内部都是B+树，高度都是平衡的，叶节点存放着所有数据。不同点是叶节点是否存放着一整行数据。
#### 聚集索引
聚集索引：该索引中键值的逻辑顺序决定了表中相应行的物理顺序。
聚集索引--->主键索引、唯一索引、多列索引
创建聚集索引的语法：create NONCLUSTERED INDEX idximpID ON EMP(empID)
#### 非聚集索引(辅助索引)
非聚集索引--->普通索引
非聚集索引(辅助索引)：数据存储在一个地方，索引存储在另一个地方，索引带有指针指向数据的存储位置。
创建非聚集索引的语法: create CLUSTERED INDEX idxempID on emp(empID)
#### 使用场景
动作描述              使用聚集索引        使用非聚集索引 
列经常被分组排序           应                  应 
返回某范围内的数据         应                 不应 
一个或极少不同值          不应                不应 
小数目的不同值             应                 不应 
大数目的不同值            不应                 应 
频繁更新的列              不应                 应 
外键列                     应                  应 
主键列                     应                  应 
频繁修改索引列            不应                 应

### hash索引
在My SQL中，只有Memory引擎显式支持哈希索引。这也是Memory引擎表的默认索 引类型，Memory引擎同时也支持B-Tree索引。

### B+Trees索引和hash索引比较
索引的存储类型目前只有两种（btree和hash），具体和存储引擎模式相关：
MyISAM        btree
InnoDB        btree
MEMORY/Heap   hash，btree
默认情况MEMORY/Heap存储引擎使用hash索引

MySQL的btree索引和hash索引的区别
hash 索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位，不像btree(B-Tree)索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问，所以 hash 索引的查询效率要远高于 btree(B-Tree) 索引。

### 索引分类以及性能测试
首先，我准备了一个百万级别的数据库来测试我们的索引，如果需要自取->[传送门](http://pan.baidu.com/s/1c2ig7He)，我COUNT了一下，300多万数据，已经勉强够我们测试使用。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-1.png)
我先看一下一下不建立任何索引查询一条数据需要多长时间哈。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-2.png)
虽然是10s左右，但是如果数据千万，甚至上亿级别的，估计一个简单的查询就要上分钟时间，但是很多情况下，大型网站数据不止这一点点，而且查询条件也会相当复杂，所以时间远远不止我们想象的这么小。我们平时浏览网站等个几秒就已经很不耐烦了，这样不仅仅影响用户的体验，而且会损失一大部分的用户。所以数据库优化是必须的。
#### 普通索引
1.创建索引
ALTER TABLE test ADD INDEX index_name (mac);
然后我们再看一下我们查询一条数据需要多长时间。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-3.png)
我只能说：很快= =。
#### 唯一索引
创建唯一索引的目的不是为了提高访问速度，而只是为了避免数据出现重复,所以这里就不测试了。唯一索引可以有多个但索引列的值必须唯一，索引列的值允许有空值。如果能确定某个数据列将只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该使用关键字UNIQUE，把它定义为一个唯一索引。
创建唯一索引
1.创建唯一索可以使用关键字UNIQUE随表一同创建
```BASH
mysql> CREATE TABLE `wb_blog` ( 
    ->   `id` smallint(8) unsigned NOT NULL, 
    ->   `catid` smallint(5) unsigned NOT NULL DEFAULT '0', 
    ->   `title` varchar(80) NOT NULL DEFAULT '', 
    ->   `content` text NOT NULL, 
    ->   PRIMARY KEY (`id`), 
    ->   UNIQUE KEY `catename` (`catid`) 
    -> ) ; 
```
2.在创建表之后使用CREATE命令来创建
```bash
mysql> CREATE UNIQUE INDEX indexName ON test(contact_id); 
```

#### 主键索引
其实我们这个在Mysql建表的时候都用过，创建主键就是创建主键索引。PS：主键索引是一种特殊的唯一索引，这里也不测试了。
问题来了，主键索引和唯一索引有什么区别呢？
1.主键一定是唯一性索引，唯一性索引并不一定就是主键。 
2.一个表中可以有多个唯一性索引，但只能有一个主键。 
3.主键列不允许空值，而唯一性索引列允许空值。

#### 全文索引
在MySQL5.6以下，只有MyISAM表支持全文检索。在MySQL5.6以上Innodb引擎表也提供支持全文检索。MySQL全文检索是利用查询关键字和查询列内容之间的相关度进行检索，可以利用全文索引来提高匹配的速度。注意需要添加全文索引的字段需要是char,varchar或者text类型.MySQL不支持中文全文索引，原因很简单：与英文不同，中文的文字是连着一起写的，中间没有MySQL能找到分词的地方，截至目前MySQL5.6版本是如此，但是有变通的办法，就是将整句的中文分词，并按urlencode、区位码、base64、拼音等进行编码使之以“字母+数字”的方式存储于数据库中。

--创建表的适合添加全文索引
CREATE TABLE `test` (
`contact_id` int(11) NOT NULL AUTO_INCREMENT ,
`prod_id` int(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL ,
`mac` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL ,
`qq` int(10) NULL DEFAULT NULL ,
`wexin` varchar(10) NULL DEFAULT NULL ,
`email` int(10) NULL DEFAULT NULL ,
`addtime` date(100) NULL DEFAULT NULL ,
`date` varchar(10) NULL DEFAULT NULL ,
PRIMARY KEY (`contact_id`),
FULLTEXT (mac)
);
修改表结构添加全文索引
ALTER TABLE test ADD FULLTEXT index_content(mac)
直接创建索引
CREATE FULLTEXT INDEX index_content ON article(mac)
在写SQL语句测试之前，我们需要先了解一下Mysql的全文搜索match...against的用法：

mysql全文搜索有三种模式：
一、自然语言查找。这是mysql默认的全文搜索方式，sql示例：
select  id,title FROM post WHERE MATCH(content) AGAINST ('search keyword')
或者显式声明使用自然语言搜索方式
select  id,title FROM post WHERE MATCH(content) AGAINST ('search keyword' IN NATURAL LANGUAGE MODE)
由于自然语言搜索方式是默认模式，所以可以省略声明模式的“IN NATURAL LANGUAGE MODE”部分。
自然语言搜索模式的么特点：
忽略停词（stopword），英语中频繁出现的and/or/to等词被认为是没有实际搜索的意义，搜索这些不会获得任何结果。
如果某个词在数据集中频繁出现的几率超过了50%，也会被认为是停词，所以如果数据库中只有一行数据，不管你怎么全文搜索都不能获得结果。
搜索结果都具有一个相关度的数据，返回结果自动按相关度由高到低排列。
只针对独立的单词进行检索，而不考虑单词的局部匹配，如搜索box时，就不会将boxing作为检索目标。

二、布尔查找。这种查找方式的特点是没有自然查找模式中的50%规则，即便有词语在数据集中频繁出现的几率超过50%，
也会被作为搜索目标进行检索并返回结果，而且检索时单词的局部匹配也会被作为目标进行检索。sql示例
1select  id,title FROM post WHERE MATCH(content) AGAINST ('search keyword' IN BOOLEAN MODE)

三、带子查询扩展的自然语言查找。
1select  id,title FROM post WHERE MATCH(content) AGAINST ('search keyword' IN BOOLEAN MODE WITH EXPANSION)
搜索语法规则：
 1.+   一定要有(不含有该关键词的数据条均被忽略)。 
 2.-   不可以有(排除指定关键词，含有该关键词的均被忽略)。 
 3.>   提高该条匹配数据的权重值。 
 4.<   降低该条匹配数据的权重值。
 5.~   将其相关性由正转负，表示拥有该字会降低相关性(但不像 - 将之排除)，只是排在较后面权重值降低。 
 6.*   万用字，不像其他语法放在前面，这个要接在字符串后面。 
 7." " 用双引号将一段句子包起来表示要完全相符，不可拆字。
SELECT id FROM articles WHERE MATCH (title,content) AGAINST ('+apple -banana' IN BOOLEAN MODE);
其中+ 表示AND，即必须包含。- 表示NOT，即必须不包含。即：返回记录必需包含 apple，且不能包含 banner。

SELECT * FROM articles WHERE MATCH (title,content) AGAINST ('apple banana' IN BOOLEAN MODE);
apple和banana之间是空格，空格表示OR。即：返回记录至少包含apple、banana中的一个。

SELECT * FROM articles WHERE MATCH (title,content) AGAINST ('+apple banana' IN BOOLEAN MODE);
返回记录必须包含apple，同时banana可包含也可不包含，若包含的话会获得更高的权重。

SELECT * FROM articles WHERE MATCH (title,content) AGAINST ('+apple ~banana' IN BOOLEAN MODE);
~ 是我们熟悉的异或运算符。返回记录必须包含apple，若也包含了banana会降低权重。
但是它没有 +apple -banana 严格，因为后者如果包含banana压根就不返回。

 SELECT * FROM articles WHERE MATCH (title,content) AGAINST ('+apple +(>banana <orange)' IN BOOLEAN MODE);
 返回必须同时包含“apple banana”或者必须同时包含“apple orange”的记录。
 若同时包含“apple banana”和“apple orange”的记录，则“apple banana”的权重高于“apple orange”的权重。

#### 多列索引
联合索引又叫复合索引、多列索引。对于复合索引:Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。
删除表中所有的索引，然后执行一条SQL，看一下执行时间，用来和创建多列查询之后的查询时间做对比。
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-5.png)
创建多列索引
ALTER TABLE test ADD INDEX index_name ( mac,qq );

然后执行上一条语句，查看一下执行时间：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/索引-6.png)
还是很快的哈。


