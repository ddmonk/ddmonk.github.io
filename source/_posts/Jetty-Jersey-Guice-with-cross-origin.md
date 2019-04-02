title: Jetty Jersey Guice with cross origin
date: 2018-06-04 11:04:51
tags:
- jetty web
---

# Jetty Request URL 跨域问题 #

## 跨域问题的原因 ##
跨域是指a页面想获取b页面资源，如果a、b页面的协议、域名、端口、子域名不同，或是a页面为ip地址，b页面为域名地址，所进行的访问行动都是跨域的，而浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源。
该部分内容可以参考 [连接](https://blog.csdn.net/qq_31617637/article/details/72955239)

## 跨域问题jetty静态资源解决办法 ##
使用jetty静态资源跨域主要是解决加载js、css等资源的跨越问题。  
解决类似问题可以为servlet增加一个Filter。简单的filter代码如下：  
``` java
public class CrossOriginFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 必须的
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        response.setHeader("Access-Control-Allow-Origin", "*");
        filterChain.doFilter(servletRequest, response);
    }
    public void destroy() {
    }
}
```
类似方法也可以使用jetty-servlets.jar中的CrossOriginFilter。`web.xml`配置文件如下：
``` xml
<web-app ...>
     ...
     <filter>
         <filter-name>cross-origin</filter-name>
         <filter-class>org.eclipse.jetty.servlets.CrossOriginFilter</filter-class>
     </filter>
     <filter-mapping>
         <filter-name>cross-origin</filter-name>
         <url-pattern>/cometd/*</url-pattern>
     </filter-mapping>
     ...
 </web-app>
```
注意，maven使用的dependency为
``` xml
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-servlets</artifactId>
            <version>${jetty.version}</version>
        </dependency>
```
可以看到如图   
![response结果](Jetty-Jersey-Guice-with-cross-origin/20180604113414.png)

## 跨域问题jersey请求解决办法 ##
我使用的Guice与jersey相结合，要使通过jersey的请求能够带上对应的Response内容，需要在JerseyServletModule中增加对应的Filter。
代码如下：   
``` java
bind(CrossOriginFilter.class).in(Singleton.class);
filter("/*").through(CrossOriginFilter.class);
```
访问后获取response即可获取。  
亲测无问题。  
附上github example地址[jetty+jesery+guice example](https://github.com/ddmonk/jetty-jesery-guice)