Java Web有三大组件，分别是Servlet、Filter和Listener，以下对这三大组件做一个简单的使用使用介绍。

## Servlet



## Filter

### 简介

Filter是一个接口，可以用来实现对请求的一个过滤。

init 方法在初始化时执行，主要用来做一些初始化的工作，比如通过 filterConfig 对象获取配置参数，连接数据库等操作。

doFilter 是过滤的核心方法，可以获取请求和响应对象，可以得到请求路径，请求和响应参数等等。filterChain 是过滤器链，如果需要继续请求操作，则需要调用 filterChain.doFilter() 方法并传入请求和响应参数。否则请求不会往下执行。如果有多个过滤器，则过滤器的执行顺序是按照过滤器名称来排序的。

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
   public void doFilter(ServletRequest arg0, ServletResponse arg1,
                        FilterChain filterChain) throws IOException, ServletException {
   
       HttpServletRequest request=(HttpServletRequest) arg0;
       HttpServletResponse response=(HttpServletResponse) arg1;
       HttpSession session=request.getSession();
   
       String path=request.getRequestURI();
       Integer uid = (Integer)session.getAttribute("userid");
   
       if(path.indexOf("/login.jsp")>-1){ // 登录页面不过滤
           filterChain.doFilter(arg0, arg1); // 递交给下一个过滤器
           return;
       }
       if(path.indexOf("/register.jsp")>-1){ // 注册页面不过滤
           filterChain.doFilter(request, response);
           return;
       }
   
       if(uid!=null){//已经登录
           filterChain.doFilter(request, response); // 放行，递交给下一个过滤器
   
       }else{
           response.sendRedirect("login.jsp");
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

