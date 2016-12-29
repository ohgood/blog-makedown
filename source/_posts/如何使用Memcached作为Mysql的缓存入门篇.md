---
title: 如何使用Memcached作为Mysql的缓存入门篇
date: 2016-12-02 21:04:05
categories: "Memcached"
tags: ["Memcached","缓存工具",,"数据库性能优化"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Memcached.jpg)
<!--more-->
#### memcached简介
自己可以去百度看一下，看完就可以对memcached有一个初步的认识。[传送门](http://baike.baidu.com/item/memcached).

#### memcached安装以及测试
安装memcached很简单，我这里是在windows操作系统上安装的memcached,步骤如下：
1.可以去官网下载memcached windows64位版， 也可以从我的网盘下载, 自取–>[传送门](http://pan.baidu.com/s/1gfLrmcb).
2.解压压缩包
3.安装并启动memcached服务
按照下面的命令顺序执行
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用memcached作为Mysql的缓存入门篇/memcached-1.png)
或者是进入“服务”程序去开启，如下图：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用memcached作为Mysql的缓存入门篇/memcached-2.png)
4.连接服务
```bash
telnet 127.0.0.1 11211
```
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用memcached作为Mysql的缓存入门篇/memcached-3.png)

#### 为什么要使用memcached呢？
使用memcached作为缓存工具的原因和是用redis做缓存的原因一样，只不过该怎么选择这两个当中的一个，这和项目的具体架构、应用场景等因素有关。例如redis一般是用作分布式缓存，而memcached是用作本地缓存的居多一点；与关系型数据库同步更新的异同等等，当然它俩的区别还包括很多，这里就不多说了，以后弄明白了再拉出来单独讲。下面就是我之前写的为什么要使用redis。
```bash
以前看见过一句话，感觉比自己说得好，就写下来吧：要避免对关系型数据库进行大量查改操作，当年新浪还是哪儿，点赞的时候后台每次都要访问修改数据库，结果死的很惨。nosql key value结构改查要比关系型数据库快的多，而且单线程的redis在并发上不需要做太大的控制，个别情况可能需要用jedis加个垮jvm的线程锁。到了公司接触了项目就会发现redis用的特别多。
自己的经历：我还记得以前做电商的时候，像购物车这一块，因为用户会频繁的刷新购物车里的商品信息，如果直接操作Mysql数据库的话，大量的磁盘I/O会给Mysql数据库带来很大的压力，因为redis直接操作内存，所以读写很快，用redis做缓存的话，我们把购物车里的数据先存到redis中，等下单的时候，可以更新一下Mysql,这样是不错的选择。
```

#### memcached实践
刚开始学的时候，自己在网上找了点资源，然后我把资源整理一下，包括加注释，使用memcached缓存的优化代码等等，加深对memcached的理解。在网上下载是要虚拟货币的，如果你也想学就到我网盘免费下载吧。[传送门](http://pan.baidu.com/s/1c1Gx19u).使用的SpringMVC+Mybatis+memcached+Mysql+maven
1.Spring配置文件
```bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
    http://www.springframework.org/schema/util 
    http://www.springframework.org/schema/util/spring-util-3.2.xsd">

	<!-- 引入jdbc配置文件 -->
	<context:property-placeholder location="classpath:jdbc.properties" />

	<!--本示例采用DBCP连接池。 连接池配置如下 -->
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${jdbc_driverClassName}" />
		<property name="url" value="${jdbc_url}" />
		<property name="username" value="${jdbc_username}" />
		<property name="password" value="${jdbc_password}" />
	</bean>

	<!-- mybatis文件配置，扫描所有mapper文件 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"
		p:dataSource-ref="dataSource" p:configLocation="classpath:mybatis-config.xml"
		p:mapperLocations="classpath:com/mapper/*.xml" /><!-- configLocation为mybatis属性 
		mapperLocations为所有mapper -->

	<!-- spring与mybatis整合配置，扫描所有dao ,生成与DAO类相同名字的bean（除了首字母小写） -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"
		p:basePackage="com.dao" p:sqlSessionFactoryBeanName="sqlSessionFactory" />

	<!-- 对数据源进行事务管理 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
		p:dataSource-ref="dataSource" />
	<tx:annotation-driven mode="proxy"
		transaction-manager="transactionManager" />
</beans>
```
2.mvc配置文件
```bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:p="http://www.springframework.org/schema/p" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">

	<!-- 扫描文件（自动注入）,包括DAO层注入Service层，Service层注入Controller层 -->
	<context:component-scan base-package="com.*" />
	<mvc:annotation-driven />
	<!-- 避免IE在ajax请求时，返回json出现下载 -->
	<bean id="jacksonMessageConverter"
		class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
		<property name="supportedMediaTypes">
			<list>
				<value>text/html;charset=UTF-8</value>
			</list>
		</property>
	</bean>

	<!-- 对模型视图添加前后缀 -->
	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.InternalResourceViewResolver"
		p:prefix="/view/" p:suffix=".jsp" />

</beans>
```
3.memcached配置文件
```bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
     http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
     http://www.springframework.org/schema/context  
     http://www.springframework.org/schema/context/spring-context-2.5.xsd  
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
	
	<!-- 设置memcached缓存池 -->
	<bean id="memcachedPool" class="com.danga.MemCached.SockIOPool"
		factory-method="getInstance" init-method="initialize" lazy-init="false"
		destroy-method="shutDown">
		<constructor-arg>
			<value>memcachedPool</value>
		</constructor-arg>
		<property name="servers">
			<list>
				<value>127.0.0.1:11211</value>
			</list>
		</property>
		<property name="initConn">
			<value>20</value>
		</property>
		<property name="minConn">
			<value>20</value>
		</property>
		<property name="maxConn">
			<value>1000</value>
		</property>
		<property name="nagle">
			<value>false</value>
		</property>
		<property name="socketTO">
			<value>3000</value>
		</property>
	</bean>
	<!-- 添加memcached客户端 -->
	<bean id="memCachedClient" class="com.danga.MemCached.MemCachedClient">
		<constructor-arg>
			<value>memcachedPool</value>
		</constructor-arg>
	</bean>
</beans>  
```
4.Mybatis
```bash
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration 
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 命名空间 -->
	<typeAliases>
		<typeAlias alias="User" type="com.model.User" />
	</typeAliases>
</configuration>
```
5.web容器
```bash
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>SpringMVCMemcached</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-root.xml,classpath:spring-memcached.xml</param-value>
  </context-param>
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```
6.pom.xml
```bash
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>SpringMVCMemcached</groupId>
	<artifactId>SpringMVCMemcached</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	<properties>
		<!-- spring版本号 -->
		<spring.version>3.2.4.RELEASE</spring.version>
		<!-- mybatis版本号 -->
		<mybatis.version>3.2.4</mybatis.version>
		<!-- log4j日志文件管理包版本 -->
		<slf4j.version>1.6.6</slf4j.version>
		<log4j.version>1.2.9</log4j.version>
	</properties>
	<dependencies>
		<!-- spring核心包 -->
		<!-- springframe start -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-pool</groupId>
			<artifactId>commons-pool</artifactId>
			<version>1.6</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>com.danga</groupId>
			<artifactId>java-memcached</artifactId>
			<version>2.6.6</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<!-- springframe end -->
		<!-- mybatis核心包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>
		<!-- mybatis/spring包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.2.2</version>
		</dependency>
		<!-- mysql驱动包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.29</version>
		</dependency>
		<!-- junit测试包 -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-dbcp2</artifactId>
			<version>2.0.1</version>
		</dependency>
		<!-- json数据 -->
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>1.9.13</version>
		</dependency>
		<!-- 日志文件管理包 -->
		<!-- log start -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<!-- log end -->
	</dependencies>
	<build>
		<finalName>springmvc</finalName>
	</build>
</project>
```
7.jdbc.properties
```bash
jdbc_driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://192.168.113.24:3306/test
jdbc_username=root
jdbc_password=123456
```
讲一讲查询操作的步骤：先查询缓存，没命中，然后再查询Mysql，查找到之后返回结果，并且add memcached缓存中;插入操作的话：只插入数据到Mysql;删除操作的话，Mysql和memcached缓存都删掉就好了。
举个例子：
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/如何使用memcached作为Mysql的缓存入门篇/memcached-4.png)
