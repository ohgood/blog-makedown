---
title: 跨域请求之Java后台实现
date: 2016-12-29 9:50:31
tags: ["Java", "跨域请求"]
categories: ["Java"]
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/java.jpg)
<!-- more -->
### 跨域问题
跨域请求就是一个站点中的资源去访问另外一个不同域名站点上的资源。 资源可以是一个请求,或一个操作或一个数据流等。
同一个ip、同一个网络协议、同一个端口，三者都满足就是同一个域，否则就是
跨域问题了。而为什么开发者最初不直接定为一切可跨域的呢？默认的为什么都是不可跨域呢？这就涉及到了同源策
略，为了系统的安全，由Netscape提出一个著名的安全策略。现在所有支持JavaScript的浏览器都会使用这个策略。
所谓同源是，域名，协议，端口相同。当我们在浏览器中打开百度和谷歌两个网站时，百度浏览器在执行一个脚本的
时候会检查这个脚本属于哪个页面的，即检查是否同源，只有和百度同源的脚本才会被执行，如果没有同源策略，那
随便的向百度中注入一个js脚本，弹个恶意广告，通过js窃取信息，这就很不安全了。

### 解决方法
关于解决跨域问题，有多种解决方案，Java后台解决方法如下：
Access-Control-Allow-Origin 为允许哪些Origin发起跨域请求. 这里设置为”*”表示允许所有，通常设置为所有并不安全，最好指定一下。 
Access-Control-Allow-Methods 为允许请求的方法. 
Access-Control-Max-Age 表明在多少秒内，不需要再发送预检验请求，可以缓存该结果 
Access-Control-Allow-Headers 表明它允许跨域请求包含content-type头，这里设置的x-requested-with ，表示ajax请求
```bash
@WebFilter(filterName="crossOrignFilter", urlPatterns={"/*"}, initParams={@WebInitParam(name="AccessControlAllowOrigin",value="*"),
                                      @WebInitParam(name="AccessControlAllowMethods", value="POST, GET, DELETE, PUT"),
                                      @WebInitParam(name="AccessControlMaxAge", value="3628800"),
                                      @WebInitParam(name="AccessControlAllowHeaders", value="x-requested-with")})
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


