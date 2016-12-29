---
title: 如何使用redis作为Mysql的缓存
date: 2016-11-27 20:05:23
categories: "redis"
tags: ["redis","缓存工具","Nosql数据库","数据库性能优化"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Redis.jpg)
<!--more-->
### redis简介
自己可以去百度看一下，看完就可以对redis有一个初步的认识。[传送门](http://baike.baidu.com/link?url=Vm1XlosB5q092b3T2CnZWdod9uPNFGm4-R7bHxTLFtw345H9hWE7vlGpm5-UJM34hNEw_uWURyBb3XbvToYfq_)

### redis安装
安装redis很简单，我这里是在windows操作系统上安装的redis,步骤如下：
1.可以去官网下载redis windows版， 也可以从我的网盘下载, 自取-->[链接](http://pan.baidu.com/s/1qYrmy40)
2.解压压缩包
安装完成了。= = 
接下来测试一下，看下是否可以使用。
1.启动redis
```bash
redis-server.exe redis.conf
```
出现下面信息表示启动成功
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用redis作为Mysql的缓存/redis-1.png)
2.连接redis服务(往这里看------>注意：1.要重新开一个dos命令窗口; 2.注意修改自己的ip)
```bash
redis-cli.exe -h 192.168.113.23 -p 6379
```
假如出现error,
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用redis作为Mysql的缓存/redis-3.png)
请使用密码登录
```bash
redis-cli.exe -h 192.168.113.23 -p 6379 -a 123456
```
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用redis作为Mysql的缓存/redis-2.png)
3.测试一下
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用redis作为Mysql的缓存/redis-4.png)
嗯，可以了。

### 为什么使用redis呢？
以前看见过一句话，感觉比自己说得好，就写下来吧：要避免对关系型数据库进行大量查改操作，当年新浪还是哪儿，点赞的时候后台每次都要访问修改数据库，结果死的很惨。nosql key value结构改查要比关系型数据库快的多，而且单线程的redis在并发上不需要做太大的控制，个别情况可能需要用jedis加个垮jvm的线程锁。到了公司接触了项目就会发现redis用的特别多。
自己的经历：我还记得以前做电商的时候，像购物车这一块，因为用户会频繁的刷新购物车里的商品信息，如果直接操作Mysql数据库的话，大量的磁盘I/O会给Mysql数据库带来很大的压力，因为redis直接操作内存，所以读写很快，用redis做缓存的话，我们把购物车里的数据先存到redis中，等下单的时候，可以更新一下Mysql,这样是不错的选择。

### redis实践
reids的一个小例子，个人感觉很有用的。如果需要请去我的网盘自己下载,里面有项目，sql文件。--->[传送门](http://pan.baidu.com/s/1skWlKlf)
工具：Myeclipse, mysql.架构是Springmvc+ibatis+redis+mysql.自己可以先部署起来，然后回来看我下面的内容，估计就知道redis怎么使用了，这样对redis入门使用还是很有帮助的。
下面就介绍一下redis有关的配置和使用：
1.redis配置文件 redis.properties
```bash
#redis中心
redis.host=127.0.0.1
redis.port=6379
redis.password=123456
redis.maxIdle=100
redis.maxActive=300
redis.maxWait=1000
redis.testOnBorrow=true
redis.timeout=100000
 
# 不需要加入缓存的类
targetNames=xxxRecordManager,xxxSetRecordManager,xxxStatisticsIdentificationManager
# 不需要缓存的方法
methodNames=
 
#设置缓存失效时间
com.service.impl.xxxRecordManager= 60
com.service.impl.xxxSetRecordManager= 60
defaultCacheExpireTime=3600
 
fep.local.cache.capacity =10000
```
2.redisContext.xml
```bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
	  http://www.springframework.org/schema/aop
	  http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"  default-autowire="byName">
	
	<!-- 读取配置文件 -->
	<context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true" />
	<!-- jedis pool配置 -->
	<bean class="redis.clients.jedis.JedisPoolConfig" id="poolConfig" >
		<property name="maxIdle" value="${redis.maxIdle}" />
        <property name="maxWait" value="${redis.maxWait}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
	</bean>
	
	<!-- redis服务器中心 -->
   <bean class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" id="connectionFactory">
        <property name="poolConfig" ref="poolConfig" />
        <property name="port" value="${redis.port}" />
        <property name="hostName" value="${redis.host}" />
        <property name="password" value="${redis.password}" />
        <property name="timeout" value="${redis.timeout}" />
   </bean>
   <bean class="org.springframework.data.redis.core.RedisTemplate" id="redisTemplate">
         <property name="connectionFactory" ref="connectionFactory" />
         <property name="keySerializer">
             <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
         </property>
         <property name="valueSerializer">
             <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
         </property>
   </bean>
   
   <!-- cache配置 -->
   <bean class="com.tony.interceptor.MethodCacheInterceptor" id="methodCacheInterceptor">
        <property name="redisUtil" ref="redisUtil" />
   </bean>
   <bean class="com.tony.util.RedisUtil" id="redisUtil">
         <property name="redisTemplate" ref="redisTemplate" />
   </bean>
   
   <!-- cache作用域 -->  
   <aop:config>
	<aop:pointcut id="redisServiceMethods" expression="execution(* com.tony.service.*.*(..))" />
	<aop:advisor advice-ref="methodCacheInterceptor" pointcut-ref="redisServiceMethods" />
    </aop:config>
</beans>
```
3.在Spring配置文件applicationContext.xml中引入redis配置文件
```bash
<!-- 引入同文件夹下的redis属性配置文件 -->
<import resource="classpath:redisContext.xml"/>
```
4.使用
我们添加一个用户的时候把用户添加到redis,调用redisUtil工具类里的方法(实际上是redisTemplate对象操作redis数据库):
```bash
/**
 * 添加用户
 * @param user
 * @return
 */
public void addUser(User user){
       redisUtil.set(user.getUname(), user);
       userDao.addUser(user);
}
```



