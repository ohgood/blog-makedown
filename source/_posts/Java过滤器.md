---
title: Java过滤器
date: 2016-12-26 23:13:31
tags: ["Java", "过滤器"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java.jpg)
<!-- more -->
### 执行流程
Filter也称之为过滤器，它是Servlet技术中最实用的技术，Web开发人员通过Filter技术，对web服务器管理的所有web资源：例如Jsp, Servlet, 静态图片文件或静态 html 文件等进行拦截，从而实现一些特殊的功能。例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息、字符串编码转换、跨域请求处理等一些高级功能。

它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理。使用Filter的完整流程：Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。通俗点讲：执行第一个过滤器的chain.doFilter()之前的代码——>第二个过滤器的chain.doFilter()之前的代码——>……——>第n个过滤器的chain.doFilter()之前的代码——>所请求servlet的service()方法中的代码——>所请求servlet的doGet()或doPost()方法中的代码——>第n个过滤器的chain.doFilter()之后的代码——>……——>第二个过滤器的chain.doFilter()之后的代码——>第一个过滤器的chain.doFilter()之后的代码。

### FilterChain
在一个web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链。请求会先执行第一个Filter的doFilter，如果次过滤器还有下一个过滤器，那么执行第二个过滤器的doFilter，否则返回响应。
注意：chain.doFilter()代码必须写在doFilter方法的最后面。

### 生命周期
1、实例化：init() 容器在创建当前过滤器的时候自动调用;
3、过滤：doFilter() 主要的代码工作;
4、销毁：destory() 容器在销毁当前过滤器的时候自动调用。

### 配置
1.在web.xml容器注册：
```bash
<filter>
  <display-name>PageEncodingFilter</display-name>
  <filter-name>PageEncodingFilter</filter-name>
  <filter-class>com.xxx.xxx.filter.PageEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>PageEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
2.Servlet 3.0 新特性@WebFilter
```bash
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;

@WebFilter(filterName="pageEncodingFilter", urlPatterns="/*")
public class PageEncodingFilter implements Filter {

    private String encoding = "UTF-8";
    protected FilterConfig filterConfig;

    public void init(FilterConfig filterConfig) throws ServletException {
        this.filterConfig = filterConfig;
        if(filterConfig.getInitParameter("encoding") != null) {
            encoding = filterConfig.getInitParameter("encoding");
        }
    }

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        request.setCharacterEncoding(encoding);
        filterChain.doFilter(request, servletResponse);
    }

    public void destroy() {
        this.encoding = null;
    }

}
```
| 属性名   | 类型  |  描述       |
| :----:     | :----: | :-------:  |
| filterName     | String  |  指定过滤器的 name 属性，等价于 <filter-name>     |
| value          | String[]|  该属性等价于 urlPatterns 属性。但是两者不应该同时使用。  |
| urlPatterns    | String[]|  指定一组过滤器的 URL 匹配模式。等价于 <url-pattern> 标签。  |
| servletNames   | String[]|  指定过滤器将应用于哪些 Servlet。取值是 @WebServlet 中的 name 属性的取值，或者是 web.xml 中 <servlet-name> 的取值。 |
| dispatcherTypes| String  |  指定过滤器的转发模式。具体取值包括：ASYNC、ERROR、FORWARD、INCLUDE、REQUEST。 |
| initParams     | String[]|  指定一组过滤器初始化参数，等价于 <init-param> 标签。  |
| asyncSupported | String  |  声明过滤器是否支持异步操作模式，等价于 <async-supported> 标签。 |
| description    | String  |  该过滤器的描述信息，等价于 <description> 标签。  |
| displayName    | String  |  该过滤器的显示名，通常配合工具使用，等价于 <display-name> 标签。  |
以上两种方法等价。那么问题来了，因为当我们使用@WebFilter注解的时候发现注解里面没有提供可以控制执行顺序的参数,使用注解方式怎么控制多个Filter的执行顺序呢？
想要控制filer的执行顺序可以通过控制filter的文件名来控制，会按照过滤器的首字母从A-Z的顺序执行。
### 再写一个例子
```bash
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;

public class CrossOriginFilter implements Filter {

  private FilterConfig config = null;
	
  public CrossOriginFilter() {
  }

  public void destroy() {
     this.config = null;
  }

  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletResponse httpResponse = (HttpServletResponse) response;
    httpResponse.setHeader("Access-Control-Allow-Origin", config.getInitParameter("AccessControlAllowOrigin"));
    httpResponse.setHeader("Access-Control-Allow-Methods", config.getInitParameter("AccessControlAllowMethods"));
    httpResponse.setHeader("Access-Control-Max-Age", config.getInitParameter("AccessControlMaxAge"));
    httpResponse.setHeader("Access-Control-Allow-Headers", config.getInitParameter("AccessControlAllowHeaders"));
    chain.doFilter(request, response);
  }

  public void init(FilterConfig config) throws ServletException {
      this.config = config;
  }

}


```

