Java Web有三大组件，分别是Servlet、Filter和Listener，以下对这三大组件做一个简单的使用使用介绍。

## Servlet

### 重定向

重定向，请求端会请求两次。

```java
@RestController
@RequestMapping("web")
public class IndexController {
    @GetMapping("index")
    public String index() {
        LocalDateTime localDateTime = LocalDateTime.now();
        return "Hello, now is " + localDateTime;
    }
    @GetMapping("/test/redirect")
    public void redirect(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("---");
        response.sendRedirect("index");
    }
}
```

默认地址只能修改一级。如果前端访问："web/test/redirect"，则redirect方法会被执行，然后前端地址改为“web/text/index”，再请求一次。

如果需要重新指定地址，应该使用`response.sendRedirect(request.getContextPath() + "web/index");`则会访问到“web/index”接口。

## Filter

### 简介

Filter是一个接口，可以用来实现对请求的一个过滤。

init 方法在初始化时执行，主要用来做一些初始化的工作，比如通过 filterConfig 对象获取配置参数，连接数据库等操作。

doFilter 是过滤的核心方法，可以获取请求和响应对象，可以得到请求路径，请求和响应参数等等。filterChain 是过滤器链，如果需要继续请求操作，则需要调用 filterChain.doFilter() 方法并传入请求和响应参数。否则请求不会往下执行，前端返回200状态空白。如果有多个过滤器，则过滤器的执行顺序是按照过滤器名称来排序的。

destroy 方法执行过滤器的销毁操作。

```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```

应用场景：敏感词汇过滤，登录状态判断，设置字符编码等等。



### Filter在SpringBoot项目中的使用

编写实现类 RequestFilter

```java
@WebFilter("/one/*") // 本过滤器对所有以 /one/ 开头的请求进行过滤操作
public class RequestFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("经过过滤器" + ((RequestFacade) servletRequest).getRequestURL());
        filterChain.doFilter(servletRequest, servletResponse);
    }
    
    // init 和 destory 方法根据具体业务来决定是否需要实现
}

```

在 SpringBoot 启动类上添加注解 @ServletComponentScan

```java
@SpringBootApplication
@ServletComponentScan // 也支持指定 Filter 所在的包名
public class WebApp {
    public static void main(String[] args) {
        SpringApplication.run(WebApp.class, args);
    }
}
```

这样过滤器就可以使用了。

如果将 @WebFilter 注解换成 @Component 也可以，只是不能指定路径过滤了，不过节省了 @ServletComponentScan 。



### 简单使用案例

1. 登录状态验证

   ```java
   @WebFilter("/user/*")
   public class AuthFilter implements Filter {
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
               throws IOException, ServletException {
           System.out.println("AuthFilter: check authentication");
           HttpServletRequest req = (HttpServletRequest) request;
           HttpServletResponse resp = (HttpServletResponse) response;
           if (req.getSession().getAttribute("user") == null) {
               // 未登录，自动跳转到登录页
               System.out.println("AuthFilter: not signin!");
               resp.sendRedirect("/signin");
           } else {
               // 已登录，继续处理
               chain.doFilter(request, response);
           }
       }
   }
   ```

   

2. 设置字符编码

   ```java
   public void doFilter(ServletRequest request, ServletResponse response,
                        FilterChain chain) throws IOException, ServletException {
       HttpServletRequest request = (HttpServletRequest) request;
       HttpServletResponse response = (HttpServletResponse) response;
   
       request.setCharacterEncoding("UTF-8");  
       response.setCharacterEncoding("UTF-8"); 
       
       chain.doFilter(request, response); 
   }
   ```

   

## Listener

### 简介

除了Servlet和Filter外，JavaEE的Servlet规范还提供了第三种组件：Listener。

Listener顾名思义就是监听器，有好几种Listener，每一种都不同事件下触发的，可以根据具体业务使用合适的Listener。通过Listener我们可以监听Web应用程序的生命周期，获取`HttpSession`等创建和销毁的事件。

使用方式与 Filter 类似，首先应该实现具体的 Listener 接口及其相应方法，同时实现类需要标注 @WebListener 注解或者 @Component 注解。

以下是几种 Listener 的介绍。

| 监听器类型                      | 介绍                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| ServletContextListener          | 监听Servlet上下文创建和销毁事件                              |
| HttpSessionListener             | 监听Session的创建和销毁事件                                  |
| ServletRequestListener          | 监听接收Http请求创建和销毁事件                               |
| ServletRequestAttributeListener | 监听ServletRequest请求的属性变化事件（即调用`ServletRequest.setAttribute()`方法） |
| ServletContextAttributeListener | 监听ServletContext的属性变化事件（即调用`ServletContext.setAttribute()`方法） |



### ServletContextListener

下面的`AppListener`实现了`ServletContextListener`接口，它会在整个Web应用程序初始化完成后，以及Web应用程序关闭后获得回调通知。我们可以把初始化数据库连接池等工作放到`contextInitialized()`回调方法中，把清理资源的工作放到`contextDestroyed()`回调方法中，因为Web服务器保证在`contextInitialized()`执行后，才会接受用户的HTTP请求。

```java
@WebListener
public class AppListener implements ServletContextListener {
    // 在此初始化WebApp,例如打开数据库连接池等:
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("WebApp initialized.");
    }

    // 在此清理WebApp,例如关闭数据库连接池等:
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("WebApp destroyed.");
    }
}
```

一个Web服务器可以运行一个或多个WebApp，对于每个WebApp，Web服务器都会为其创建一个全局唯一的`ServletContext`实例，我们在`AppListener`里面编写的两个回调方法实际上对应的就是`ServletContext`实例的创建和销毁。`ServletContext`是一个WebApp运行期的全局唯一实例，可用于设置和共享配置信息。

`ServletRequest`、`HttpSession`等很多对象也提供`getServletContext()`方法获取到同一个`ServletContext`实例。

```java
public void contextInitialized(ServletContextEvent sce) {
    System.out.println("WebApp initialized: ServletContext = " + sce.getServletContext());
}
```

很多第三方Web框架都会通过一个`ServletContextListener`接口初始化自己。

### HttpSessionListener

HttpSessionListener 可以监听 Session 的创建和销毁事件。可以用来统计在线人数，或者结合 WebSocket 等实现超时前端跳转到登录页面等功能。

Session 的详细介绍见相应章节。

```java
@WebListener
public class SessionListener implements HttpSessionListener {
    
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        System.out.println("Session创建" + se.getSession());
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        System.out.println("Session销毁" + se.getSession());
    }
}
```

### ServletRequestListener

ServletRequestListener 可以监听Http请求创建和销毁事件，可以做一些与拦截相关的业务。

```java
@WebListener
public class RequestListener implements ServletRequestListener {

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
        System.out.println("请求初始化了");
    }

    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
        System.out.println("请求销毁了");
    }
}
```

