---
title: Java无界通配符
date: 2016-12-20 17:32:01
tags: ["Java"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java.jpg)
<!-- more -->
### 用之前想说的话

java中的通配符由**?**表示，**?**代表任何类型，应用场景是在方法的形参上使用，为了弥补泛型机制带来的参数传递问题，主要有三种通配符分类 无界通配：?，子类限定：? extends Object，父类限定：? super Integer。

### 本人为什么会用？
最近在用java语言写接口，和前端商量对接数据的接收格式和返回格式，当然json格式啦。项目使用的是SpringMVC框架，**@ResponseBody**注解可以让框架帮我们封装数据成json格式。因为在返回之前要对各种list对象分装成一个对象，list对象类型不唯一，所以需要写多个返回的javabean，这样写代码"很丑"。为了提高代码重用率，所以想到了通配符**?**

### 使用方法

很简单，看下代码就明白了。
javabean
```bash
public class ObjectList {
	private List<?> allParams;
	
	private List<?> partParams;

	public List<?> getAllParams() {
		return allParams;
	}

	public void setAllParams(List<?> allParams) {
		this.allParams = allParams;
	}

	public List<?> getPartParams() {
		return partParams;
	}

	public void setPartParams(List<?> partParams) {
		this.partParams = partParams;
	}

	@Override
	public String toString() {
		return "ObjectList [allParams=" + allParams + ", partParams="
				+ partParams + "]";
	}
}
```
控制层Controller:
```bash
List<BasicParams> allBasicParams = dataService.getAll();
		oList.setAllParams(allBasicParams);
```
```bash
List<WebVisit> allBasicParams = dataService.getAllIp();
		oList.setAllParams(allBasicParams);
```
