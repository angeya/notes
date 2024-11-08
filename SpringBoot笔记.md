# SpringBoot

## 常用注解

#### @PostConstruct

@postContruct全限定类名是javax.annotation.PostConstruct，可以看出来其本身不是Spring定义的注解，但是Spring提供了具体的实现。

**作用**：@PostConstruct修饰的方法只会被调用一次，因为它只会在Bean初始化时执行一次。通常情况下，@PostConstruct方法被用于执行一些初始化操作，例如资源初始化、数据加载、连接建立等。

**执行时机**：@PostConstruct修饰的方法会在依赖注入完成后立即被调用。

从@PostConstruct注解的注释上看，可以了解到以下内容：

1. 在依赖加载后，对象使用前执行，并且只执行一次；

2. 所有支持依赖注入的类都需要支持此方法。即使类没有请求注入任何的资源，也必须调用被@PostConstruct注解标记的方法；

3. 一个类中在一个方法上使用@PostConstruct注解；

4. 使用@PostConstruct注解标记的方法不能有参数，除非是拦截器，可以采用拦截器规范定义的InvocationContext对象。

5. 使用@PostConstruct注解标记的方法不能有返回值，实际上如果有返回值，也不会报错，但是会忽略掉；

6. 使用@PostConstruct注解标记的方法的权限，public、private、protected都可以；

7. 使用@PostConstruct注解标记的方法不能被static修饰，但是final是可以的；

## IOC

## AOP

### 原理



### 使用

1. 引入依赖

   ```xml
   <!-- 引入aop支持 -->
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 编写切面类

   

3. 增强方法

   

### 切点PointCut

切点是指哪些方法需要被增强，PointCut表达式可以有一下几种方式：

1. execution

   execution([可见性]返回类型[声明类型].方法名(参数)[异常])，其中[]内的是可选的，其他的还支持通配符的使用：

   - *: 匹配所有

   - …: 匹配多个包或多个参数

   - +: 表示类及其子类

   - 运算符：&&、||、！

2. within

   是用来指定类型的，指定类型中的所有方法将被拦截是用来指定类型的，指定类型中的所有方法将被拦截。

   ```java
   within(com.demo.service.impl.UserServiceImpl); // 匹配UserServiceImpl类对应对象的所有方法调用，并且只能是UserServiceImpl对象，不能是它的子对象
   within(com.demo…*); // 匹配com.demo包及其子包下面的所有类的所有方法的外部调用
   ```

   

3. this

   SpringAOP是基于代理的，this就代表代理对象，语法是this(type)，当生成的代理对象可以转化为type指定的类型时表示匹配。

   ```java
   this(com.demo.service.IUserService); // 匹配生成的代理对象是IUserService类型的所有方法的外部调用
   ```

   

4. target

   SpringAOP是基于代理的，target表示被代理的目标对象，当被代理的目标对象可以转换为指定的类型时则表示匹配。

   ```java
   target(com.demo.service.IUserService); // 匹配所有被代理的目标对象能够转化成IuserService类型的所有方法的外部调用。
   
   ```

5. args

   args用来匹配方法参数。

   ```java
   args() // 匹配不带参数的方法
   args(java.lang.String) // 匹配方法参数是String类型的
   args(…) //带任意参数的方法
   args(java.lang.String,…) // 匹配第一个参数是String类型的，其他参数任意。最后一个参数是String的同理。
   ```

6. @within 和 @target

   带有相应标注的所有类的任意方法，比如@Transactional

   ```java
   @within(org.springframework.transaction.annotation.Transactional)
   @target(org.springframework.transaction.annotation.Transactional)
   ```

7. @annotation

   带有相应标注的任意方法，比如@Transactional或其他自定义注解

   ```java
   @annotation(org.springframework.transaction.annotation.Transactional)
   ```

8. @args

   参数带有相应标注的任意方法

### AOP支持的通知类型

1. 前置通知@Before：在某连接点之前执行的通知

   ```java
   @Before(value = "pointCut()")
   public void before(JoinPoint joinPoint){
       ...
   }
   ```

   

2. 环绕通知@Around

   ```java
   /** 
    * 环绕通知： 环绕通知非常强大，可以决定目标方法是否执行，什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值。 
    *   环绕通知第一个参数必须是org.aspectj.lang.ProceedingJoinPoint类型 
    */  
   @Around(value = "pointCut()")  
   public Object doAroundAdvice(ProceedingJoinPoint proceedingJoinPoint){  
       logger.info("环绕通知的目标方法名：" + proceedingJoinPoint.getSignature().getName());  
       try {  
           Object obj = proceedingJoinPoint.proceed();  
           return obj;  
       } catch (Throwable throwable) {  
           throwable.printStackTrace();  
       }  
       return null;  
   }  
   ```

3. xxxxxxxxxx // 简单处理数字的自定义拦截器，需要实现Interceptor接口public class HandleNum implements Interceptor {    private static Logger logger = LoggerFactory.getLogger(HandleNum.class);    /**     * 初始化操作，可以用于打开资源     */    public void initialize() {}​    /**     * 拦截单个event     */    public Event intercept(Event event) {        //输出20-60之间的数，大于60，则×100        int num = Integer.parseInt(new String(event.getBody()));        if (num < 20) {            logger.warn("num {} is too small, abandon!", num);            return null;        } else if (num > 60){            num = num * 100;        }        event.setBody(Integer.toString(num).getBytes());        return event;    }​    /**     * 拦截多个event（可以直接调用单个 event 的拦截器）     */    public List<Event> intercept(List<Event> list) {        return list.stream()                .map(this::intercept)                .filter(Objects::nonNull)                .collect(Collectors.toList());    }​    /**     * 拦截器关闭，可以用于关闭资源     */    public void close() {}​    /**     * 通过内部类继承Builder类，并实现build方法，建立拦截器     */    public static class Builder implements Interceptor.Builder {        public Interceptor build() {            return new HandleNum(); // 创建拦截器实例        }​        public void configure(Context context) {            // 上下文环境，可以获取配置信息        }    }}java

   ```java
   /** 
    * 后置返回通知 
    * 这里需要注意的是: 
    *      如果参数中的第一个参数为JoinPoint，则第二个参数为返回值的信息 
    *      如果参数中的第一个参数不为JoinPoint，则第一个参数为returning中对应的参数 
    *      returning：限定了只有目标方法返回值与通知方法相应参数类型时才能执行后置返回通知，否则不执行，
    *      对于returning对应的通知方法参数为Object类型将匹配任何目标返回值 
    */  
   @AfterReturning(value = "pointCut()",returning = "keys")  
   public void doAfterReturningAdvice1(JoinPoint joinPoint,Object keys){  
       logger.info("第一个后置返回通知的返回值："+keys);  
   }  
   
   @AfterReturning(value = "pointCut()",returning = "keys",argNames = "keys") 
   public void doAfterReturningAdvice2(String keys){  
       logger.info("第二个后置返回通知的返回值："+keys);  
   
   ```

   

4. 后置返回通知@AfterReturning：在某连接点之后执行的通知，通常在一个匹配的方法返回值的时候执行（可以在后置通知中绑定返回值）

   ```java
   /** 
    * 后置最终通知（目标方法只要执行完了就会执行后置通知方法） 
    */  
   @After(value = "pointCut()")  
   public void doAfterAdvice(JoinPoint joinPoint){ 
       logger.info("后置最终通知执行了!!!!");  
   }  
   ```

5. 后置异常通知@AfterThrowing

   ```java
   /** 
    * 后置异常通知 
    *  定义一个名字，该名字用于匹配通知实现方法的一个参数名，当目标方法抛出异常返回后，将把目标方法抛出的异常传给通知方法； 
    *  throwing:限定了只有目标方法抛出的异常与通知方法相应参数异常类型时才能执行后置异常通知，否则不执行， 
    *   对于throwing对应的通知方法参数为Throwable类型将匹配任何异常。 
    */  
   @AfterThrowing(value = "pointCut()",throwing = "exception")  
   public void doAfterThrowingAdvice(JoinPoint joinPoint,Throwable exception){ 
       //目标方法名
       logger.info(joinPoint.getSignature().getName());  
       if(exception instanceof NullPointerException){  
           logger.info("发生了空指针异常!!!!!");  
       }  
   }
   ```

### JoinPoint和ProceedingJoinPoint使用详解

#### JoinPoint可获取的信息

```java
//返回目标对象，即被代理的对象
Object getTarget();
//返回切入点的参数
Object[] getArgs();
//返回切入点的Signature
Signature getSignature();
//返回切入的类型，比如method-call，field-get等等
String getKind();
```

#### ProceedingJoinPoint

Proceedingjoinpoint 继承了 JoinPoint。是在JoinPoint的基础上暴露出 proceed 这个方法。proceed很重要，这个是aop代理链执行的方法。

#### JoinPoint使用

```java
// 获取切入点所在目标对象
Object targetObj =joinPoint.getTarget();
// 可以发挥反射的功能获取关于类的任何信息，例如获取类名如下
String className = joinPoint.getTarget().getClass().getName();
// 获取参数列表
Object[] args = joinPoint.getArgs();
// 获取切入点方法的名字: getSignature())是获取到这样的信息 :修饰符+ 包名+组件名(类名) +方法名
Signature signature = joinPoint.getSignature();
```

### AOP失效的场景

1. 方法不是public，或者是static等

2. 增强方法在同一个类内被调用

   如果在一个类内，有A和B两个方法，A调用B，切面对B进行增强，调用A方法的时候，B方法不会被增强。因为调用A方法的时候，不是使用代理对象，A直接调用B的时候，也没有使用代理对象。

   这时候可以在A方法中使用`AopContext.currentProxy()`方法获取当前类的代理对象。进而调用增强方法。使用方法如下：

   ```java
   ((AClass)AopContext.currentProxy()).B();
   ```

3. 使用 AspectJ 注解，但是启动类中没有使用`@EnableAspectJAutoProxy`注解开启代理。



## 扩展接口

### ApplicationContextAware

`ApplicationContextAware` 接口是 Spring 框架提供的一个接口，用于获取 Spring 应用上下文（ApplicationContext）。通过实现 `ApplicationContextAware` 接口，可以在 Spring 容器启动时获取到应用的上下文，并对其进行操作。

要实现 `ApplicationContextAware` 接口，需要重写其中的 `setApplicationContext` 方法，该方法会在 Spring 容器初始化时被调用，传入应用的上下文对象。

示例代码如下：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class MyBean implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    // Spring 容器初始化时被调用
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
        // 这里可以使用 applicationContext 对象来获取 Spring 容器中的 Bean
        // 或者把 applicationContext 赋值给到其他类的静态属性中
    }
}

```

### SmartInitializingSingleton

`SmartInitializingSingleton` 接口是 Spring 框架提供的一个特殊的初始化接口，用于在所有单例 Bean 初始化完成之后执行自定义的初始化逻辑。它允许在 Spring 容器初始化阶段结束后执行一些操作。

要使用 `SmartInitializingSingleton` 接口，需要实现其中的 `afterSingletonsInstantiated` 方法。这个方法会在 Spring 容器中的所有单例 Bean 实例化完成后被调用。

示例代码如下：

```java
import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.stereotype.Component;

@Component
public class MyBeen implements SmartInitializingSingleton {

    @Override
    public void afterSingletonsInstantiated() {
        // 所有单例 Bean 初始化完成后执行的逻辑
        System.out.println("All singletons have been instantiated.");
        // 可以在这里进行额外的初始化操作
    }
}
```

通过实现 `SmartInitializingSingleton` 接口，可以在 Spring 容器初始化阶段结束后执行一些定制化的初始化操作，例如对 Bean 进行进一步的配置、数据加载、缓存预热等。这个接口提供了一种方便的扩展点，可以在应用启动时执行额外的初始化逻辑。



### DisposableBean

`DisposableBean` 是 Spring 框架提供的一个接口，用于定义 bean 销毁之前的回调逻辑。实现了 `DisposableBean` 接口的类可以在 Spring 容器关闭时，或者具体的 bean 被销毁前执行自定义的清理逻辑。

要使用 `DisposableBean` 接口，需要实现其中的 `destroy` 方法。这个方法会在 bean 销毁之前被 Spring 容器调用，从而允许开发者执行如资源释放、连接关闭等清理操作。

示例代码如下：

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.stereotype.Component;

@Component
public class MyResourceHandler implements DisposableBean {

    @Override
    public void destroy() throws Exception {
        // 在这里实现清理逻辑
        System.out.println("Cleaning up resources...");
        // 例如，关闭文件流、数据库连接等
    }
}
```

在这个示例中，`MyResourceHandler` 类实现了 `DisposableBean` 接口，并在 `destroy` 方法中定义了资源清理的逻辑。当应用停止或者容器中的 `MyResourceHandler` bean 被销毁时，`destroy` 方法会被自动调用。

`DisposableBean` 接口提供了一种标准的方式来进行资源清理，有助于防止资源泄漏。在实际开发中，除了直接实现 `DisposableBean` 接口之外，也可以使用 `@PreDestroy` 注解或者在 bean 的定义中指定 `destroy-method` 属性来指定销毁回调，这些方法也都可以用于定义销毁前的清理逻辑。选择哪种方式取决于具体的场景和个人偏好。

### ApplicationContextInitializer

允许在 Spring 应用程序上下文（ApplicationContext）创建之前对其进行修改。可以通过实现此接口来自定义 ApplicationContext 的初始化逻辑。

### ApplicationListener

用于监听 Spring 应用程序中特定事件的发生。通过实现此接口，可以在应用程序启动、关闭、上下文刷新等事件发生时执行相应的逻辑。

### CommandLineRunner 和 ApplicationRunner

用于在 Spring Boot 应用程序启动后立即执行一些任务或命令行操作。`CommandLineRunner` 接口的 `run` 方法会在 Spring Boot 应用程序启动时被调用，而 `ApplicationRunner` 接口则提供了更丰富的参数信息。使用详情[启动监听器](#AppLaunchListener)。

### EnvironmentPostProcessor

用于在 Spring 应用程序上下文创建之前对环境进行修改。可以通过实现此接口来动态地修改应用程序的配置信息。

### BeanPostProcessor

允许在 Spring 容器中的 bean 实例化、依赖注入等过程中对 bean 进行定制化处理。通过实现此接口，可以在 bean 初始化之前和之后执行自定义的逻辑。

### HandlerInterceptor

用于拦截和处理 Web 请求。通过实现此接口，可以在请求处理之前、之后以及完成后执行自定义的逻辑，例如身份验证、日志记录等。

### EmbeddedServletContainerCustomizer 和 WebServerFactoryCustomizer

用于自定义嵌入式 Servlet 容器的配置。通过实现这些接口，可以对嵌入式 Servlet 容器进行各种配置，如端口号、上下文路径、SSL 配置等。

### HandlerMethodArgumentResolver 和 HandlerMethodReturnValueHandler

用于扩展 Spring MVC 的处理器方法参数解析和返回值处理。通过实现这些接口，可以自定义处理器方法的参数解析逻辑和返回值处理逻辑。



## Spring MVC

### 基础

#### Spring MVC的设计与流程

1. MVC思想

    在处理HTTP请求时，请求先到达控制器（Controller），控制器的作用是进行请求分发；

    这样它会根据请求的内容去访问模型层（Model； 在现今互联网系统中，数据主要从数据库和 NoSQL 中来，而且对于数据库而言往往还存在事务的机制，设计者会把模型层再细分为两层，即服务层（Service）和数据访问层（DAO）；

    当控制器取到由模型层返回数据后 ，就将数据渲染到视图（View）中，这样就能够展现给用户了。

    TODO图片

    

    在日常开发中，我们最主要的工作是控制器的开发。

2. Spring MVC的处理流程

    流程和组件是 SpringMVC 的核心， SpringMVC的流程是围绕 DispatcherServlet 而工作的，所以 pring MVC DispatcherServlet 是其最重要的内容，在DIspatcherServlet 的基础上，还存在其他的组件， 掌握流程和组件就是 SpringMVC 开发的基础。

    TODO图片

    

    DispatcherServlet：请求的分发中心，收到请求之后转发给HandlerMapping

    HandlerMapping：处理器映射，通过Http请求的信息，找出对应的控制器。控制器中@Controller注解表明这是一个控制器，@RequestMapping代表请求路径和控制器的映射关系，这个映射关系会在服务器启动SpringMVC时扫描到HandlerMapping的机制中存储。等收到请求之后通过URI和其他条件就能找到对应的控制器方法进行响应。只不过通过处理器映射返回的是一个HandlerExecutionChain对象。

    HandlerExecutionChain：处理器执行链，内部包含处理器（对控制器的包装）和拦截器。

    HandlerAdapter：处理器适配器。除了Http请求，还有按BeanName的请求或者是WebSocket的请求，所以得到了控制器之后不能直接执行控制器方法，因此需要一个适配器去运行HandlerExecutionChain对象的处理器。

    ModelAndView：模型与视图。运行控制器方法（一般会调用Service和Dao层）后，得到数据，将数据和视图（如Jsp文件）路径封装成ModeAndView对象返回给DispatcherServlet。

    ViewResolver：视图解析器。通过ModelAndView对象（逻辑视图），定位视图资源。
    
    View：视图渲染（结果可能是Jsp）。

#### 处理器映射

请求路径与控制器方法的映射，在方法上使用注解@RequestMapping。如

```java
@RequestMapping(value = "sayHi",method = RequestMethod.POST)
public void sayHi(){
    // do some thing
}
```

如果@RequestMapping用在控制器上，则控制器的所有方法都控制器配置的请求路径下。

在 Spring 4.3 的版本之后，为了简 method配置项的配置新增了几个注解，如`＠GetMapping` 、`＠PostMapping` 、`＠PatchMapping` 、`＠PutMapping、@DeleteMapping` 。

从名称可以看出，＠GetMapping 对应的是GET 方法， @PostMapping 对应的是POST 方法。

#### 获取控制器方法参数

处理器是对控制器的包装 ，在处理器运行的过程中会调度控制器方法，只是它在进入控制器方法之前会对 HTTP 参数和上下文进行解析，将它们转换为控制器所参数。开发者也可以自定义转换规则。

1. 无注解时获取参数

    控制器方法中没有注解，则表示http请求参数名称与方法参数名称一致。

    如：

    http://localhost:8080l/test?id=20&price=50与方法

    ```java
    @PostMapping (”/test”) 
    public void test(Integer id, Float price, String name){
    	// do some thing
    }
    ```

    对应，name默认允许为空。

    Spring MVC支持使用逗号分隔的数组参数，如Integer[] ids对应参数?id=15,16

2. 使用＠RequestParam 获取参数

    不使用参数注解，前后端参数命名必须一致。使用注解＠RequestParam可以做参数名称映射

    如：

    http://localhost:8080l/test?orderId=20&price=50与方法

    ```java
    @PostMapping (”/test”)
    public void test(@RequestParam(value="orderId”)Integer id,
                     @RequestParam(value="price”)Float price, 
                     @RequestParam(value="name”, required = false) String name){
    	// do song thing
    }
    ```

    对应，设置required = false，表示参数name允许为空。

3. 传递Json

    当前前后端分离趋势下 ，使用 JSON 已经十分普遍。

    将参数使用@RequestBody标注，意味着接收前端的Json请求体。

    ```java
    @PostMapping (”/insert”) 
    @ResponseBody
    public User insert (@RequestBody User user) {
        // do some thing
        return user; 
    }
    ```

    控制器方法标注@Response注解，会将方法返回值转换为Json数据格式返回给前端。

4. 通过URL传递参数

    REST风格的接口，参数往往通过URL进行传递。例如id为1的用户，URL一般为/user/1。Spring MVC通过注解@PathVariable和处理器映射器的组合获取URL参数。首先通过处理器映射可以定位参数的位置和名称，而＠PathVariable 则可以通过名称来获取参数。

    ```java
    @GetMapping (”/user/{id}”)  //｛...} 代表占位 ，还可以配置参数名称
    @PathVariable // 通过名称获取参数
    public User getUser (@PathVariable (”id” ) Long id) {
        // do some thing
        return user;
    }
    ```

5. 获取格式化参数

    Spring MVC支持还支持对参数的格式化，如日期(yyyy-MM-dd)和数值(1,000,000.23)等。

    ```java
    PostMapping("/format/commit"）
    public void formatTest(@DateTimeFormat(iso＝ISO.DATE) Date date , 
    						@NumberFormat(pattern =”#,###.##”) Double number) {
        // do some thing
    }
    ```

    Spring Boot 中， 日期参数的格式化也可以不使用＠DateTimeFormat 注解，而只在配置文件application.properties 中加入如下配置项即可：`spring.mvc.date-format=yyyy-MM-dd `

#### 文件上传

SpringMVC 对文件上传的支持：DispatcherServlet 会使用适配器模式，将 HttpServletRequest 对象转换为 MultipartHttpServletRequest 象。 MultipartHttpServletRequest 接口扩展了HttpServletRequest 接口的所有方法，而且定义了一些操作文件的方法 ，这样通过这些方法就可以实现对上传件的操作。

在SpringBoot的机制内，自动配置的机制会为我们自动创建 MultipartResover 对象来实现文件上传，而且支持如下关于文件上传的配置。

```properties
spring.servlet.multipart.enabled: true #是否启用springMVC multipart(http请求中将文件分为多个部分)上传功能
spring.servlet.multipart.file-size-threshold=0 ＃将文件写入磁盘的阀值 值可以使用后缀“MB ”或“ KB 来表示兆字节或字节大小
spring.servlet.multipart.max-file-size: 500MB #单个文件进最大值
spring.servlet.multipart.max-request-size: 1000MB #所有文件最大值
spring.servlet.multipart.location: "files" # 文件保存目录，该目录在tomcat下
```

在控制器方法中，可以接收文件的参数有三种类型。每一种类型的处理方法都不一样。推荐使用第二、三种。

```java
// 1、使用 HttpServletRequest 类作为参数
PostMapping(”/upload/request”) 
public void uploadRequest(HttpServletRequest request) { 
    // 强制转化为 MultipartHttpServletRequest 接口对象
    MultipartHttpServletRequest mreq = (MultipartHttpServletRequest) request; 
    MultipartFile mf = mreq.getFile(”file ”);
    String fileName = mf.getOriginalFilename(); // 获取源文件名称
    File file= new File(fileName) ; 
    mf.transferTo (file ); // 保存文件
}
                     
// 2、使用 Spring MVC MultipartFile 类作为参数
PostMapping(”/upload/multipart”) 
public void uploadMultipartFile (MultipartFile file) { 
    String fileName = file.getOriginalFilename(); // 获取源文件名称
    File dest =new File(fileName) ; 
    file.transferTo(dest); // 保存文件
}

// 3、使用 Part 类作为参数
@PostMapping(”/upload/part") 
public void uploadPart (Part file ) { 
    String fileName = file.getSubmittedFileName(); // 获取源文件名称
    filele.write(fileName); // 保存文件
}

```

#### 拦截器

当请求来到 DispatcherServlet 时， 它会根据 HandlerMapping 的机制找到处理器 这样就会返回一个 HandlerExecutionChain对象，这个对象包含处理器和拦截器。这里的拦截器会对处理器进行拦截，这样通过拦截器就可以增强处理器的功能。（比如上传文件时，如果文件过大，只有在文件全部接收完成之后，才会将文件作为参数传送给控制器，使用拦截器在请求刚到达时就做一些处理）。

拦截器的开发分为两步：

第一步是编写拦截器（实现HandlerInterceptor接口，重写相关方法）：

```java
// 自定义拦截器，（实现接口方法参数以省略）
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 在请求处理器之前进行调用（Controller方法调用之前）
        System.out.println("Pre Handle");
        return true; // 如果返回false，则停止流程，api不会被调用
    }
 
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 请求处理器之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
        System.out.println("Post Handle");
    }
 
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception{
        // 在整个请求结束之后调用，也就是在DispatcherServlet渲染了视图执行
        System.out.println("After Completion");
    }
}
```

第二步是在SpringMVC中注册拦截器：

```java
// 注册拦截器
@Component
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 将自定义拦截器对象注册拦截器到 Spring MVC，然后会返回一个拦截器注册
        InterceptorRegistration ir = registry.addInterceptor(new MyInterceptor() );
        //指定拦截匹配模式，限制拦截器拦截请求
        ir.addPathPatterns("/*/upload*"); // 这里可以配置拦截的路径，该模式可拦截请求/test/uploadFile
        //.excludePathPatterns("/login", "/error"); // 这里可以配置不拦截的路径
    }
}
```











### Controller最佳实践

#### 统一状态码

1. 返回格式

   我们通常需要对后端返回的数据进行包装一下，增加一下`状态码`，`状态信息`，这样前端接收到数据就可以根据不同的`状态码`，判断`响应数据状态`，是否成功是否异常进行不同的显示。假设我们约定了`1000`就是成功的意思如果你不封装，那么返回的数据是这样子的。

   ```json
   {
       "name": "sunny",
       "age": 20
   }
   ```

   经过封装以后时这样子的

   ```json
   {
     "code": 1000,
     "msg": "请求成功",
     "data": {
       "name": "sunny",
       "age": 20
     }
   }
   ```

2. 封装ResultVO

   这些状态码肯定都是要预先编好的，怎么编呢？写个常量`1000`？还是直接写死`1000`？要这么写就真的书白读的了，写`状态码`当然是用枚举啦

   首先先定义一个`状态码`的接口，所有`状态码`都需要实现它，有了标准才好做事；

   ```java
   public interface StatusCode {
       public int getCode();
       public String getMsg();
   }
   ```

   然后去找前端跟他约定好状态码，枚举类嘛，当然不能有`setter`方法了，因此我们不能在用`@Data`注解了，我们要用`@Getter`；

   ```java
   @Getter
   public enum ResultCode implements StatusCode{
      
       SUCCESS(1000, "请求成功"),
       FAILED(1001, "请求失败"),
       VALIDATE_ERROR(1002, "参数校验失败"),
       RESPONSE_PACK_ERROR(1003, "response返回包装失败");
   
       private int code;
       private String msg;
   
       ResultCode(int code, String msg) {
           this.code = code;
           this.msg = msg;
       }
   }
   ```

   写好枚举类，就开始写`ResultVO`包装类了，我们预设了几种默认的方法，比如成功的话就默认传入`object`就可以了，我们自动包装成`success`；

   ```java
   @Data
   public class ResultVO {
       // 状态码
       private int code;
   
       // 状态信息
       private String msg;
   
       // 返回对象
       private Object data;
   
       // 手动设置返回vo
       public ResultVO(int code, String msg, Object data) {
           this.code = code;
           this.msg = msg;
           this.data = data;
       }
   
       // 默认返回成功状态码，数据对象
       public ResultVO(Object data) {
           this.code = ResultCode.SUCCESS.getCode();
           this.msg = ResultCode.SUCCESS.getMsg();
           this.data = data;
       }
   
       // 返回指定状态码，数据对象
       public ResultVO(StatusCode statusCode, Object data) {
           this.code = statusCode.getCode();
           this.msg = statusCode.getMsg();
           this.data = data;
       }
   
       // 只返回状态码
       public ResultVO(StatusCode statusCode) {
           this.code = statusCode.getCode();
           this.msg = statusCode.getMsg();
           this.data = null;
       }
   }
   ```

   使用，现在的返回肯定就不是`return data;`这么简单了，而是需要`new ResultVO(data);`；

   ```java
   @PostMapping("/getUser")
   public ResultVO getUser(@Validated @RequestBody user) {
      return new ResultVO(user);
   }
   ```


   最后返回就会是上面带了状态码的数据了

#### 统一校验

1. 原始做法

   假设有一个添加`User`的接口，在没有统一校验时，我们需要在代码里去添加名称是否为空，年龄是否为负数等信息，这样很不优雅，而且不好维护。

2. @Validated参数校验

   好在有`@Validated`注解，又是一个校验参数必备良药了。有了`@Validated`我们只需要在`User`上面加一点小小的注解，便可以完成校验功能

   字段的校验注解（具体内容可以学习`spring-boot-starter-validation`的使用）需要添加如下maven依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-validation</artifactId>
   </dependency>
   ```

   User实体类

   ```java
   @Data
   public class User {
       @NotNull
       private String name;
   
       @Min(value=0, message="年龄不允许为负数")
       private Integer age;
   }
   ```

   controller

   ```java
   @PostMapping("getUser")
   public ResultVO getUser(@Validated @RequestBody User user) {
       return new ResultVO(user);
   }
   ```

   运行看看，如果参数不对会发生什么？
   我们故意传一个年龄为`-1`的参数过去，接口报错了

   ```json
   {
   	"timestamp": "2023-09-17T16:05:40.303+00:00",
   	"status": 400,
   	"error": "Bad Request",
   	"message": "",
   	"path": "/demo/getUser"
   }
   ```

   后台`WARN`日志如下：

   ```bash
   Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public top.angeya.vo.ResultVO top.angeya.controller.DemoController.getUser(top.angeya.entity.User)
   ```

   所以需要优化一下

3. 优化异常处理

   首先我们先看看校验参数抛出的异常是：`MethodArgumentNotValidException`

   因此我们的思路就是`AOP`拦截所有`controller`，然后异常的时候统一拦截起来，进行封装！完美！

   `spring mvc`当然知道拉，所以给我们提供了一个`@RestControllerAdvice`来增强所有`@RestController`，然后使用`@ExceptionHandler`注解，就可以拦截到对应的异常，可以对异常处理并给接口设置新的返回值。

   这里我们就拦截`MethodArgumentNotValidException.class`就好了。最后在返回之前，我们对异常信息进行包装一下，包装成`ResultVO`，当然要跟上`ResultCode.VALIDATE_ERROR`的异常状态码。这样前端看到`VALIDATE_ERROR`的状态码，就会调用数据校验异常的弹窗提示用户哪里没填好

   ```java
   @RestControllerAdvice
   public class ControllerExceptionAdvice {
       
       @ExceptionHandler({MethodArgumentNotValidException.class})
       public ResultVO methodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e) {
   
           // 从异常对象中拿到ObjectError对象
           ObjectError objectError = e.getBindingResult().getAllErrors().get(0);
           return new ResultVO(ResultCode.VALIDATE_ERROR, objectError.getDefaultMessage());
       }
   }
   ```

   来康康效果，完美。`1002`与前端约定好的状态码

   ```json
   {
   	"code": 1002,
   	"msg": "参数校验失败",
   	"data": "年龄不允许为负数"
   }
   ```

#### 统一响应

1. 统一包装响应

   再回头看一下`controller`层的返回

   ```java
   return new ResultVO(user);
   ```

   开发肯定不乐意了，谁有空天天写`new ResultVO(data)`啊，我就想返回一个实体！怎么实现我不管！SpringMVC当然知道这个操作。需要使用`@RestControllerAdvice`注解和`ResponseBodyAdvice`接口。

   `ResponseBodyAdvice`接口的supports()方法可以设置支持哪些方法或者哪些类进行通知操作。beforeBodyWrite()方法则是对需要通知的方法进行设置返回值。如果supports()方法返回false，则不会执行beforeBodyWrite()方法。
   
   ```java
   // basePackages可以指定扫描哪些包下面的controller,在Response时进行统一处理
   @RestControllerAdvice(basePackages = {"top.angeya"})
   public class ControllerResponseAdvice implements ResponseBodyAdvice<Object> {
      
       @Override
       public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
           // response是ResultVO类型
           return !methodParameter.getParameterType().isAssignableFrom(ResultVO.class);
       }
   
       @Override
       public Object beforeBodyWrite(Object data, MethodParameter returnType, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest request, ServerHttpResponse response) {
           // String类型不能直接包装
           if (returnType.getGenericParameterType().equals(String.class)) {
               ObjectMapper objectMapper = new ObjectMapper();
               try {
                   // 将数据包装在ResultVO里后转换为json串进行返回
                   return objectMapper.writeValueAsString(new ResultVO(data));
               } catch (JsonProcessingException e) {
                   throw new APIException(ResultCode.RESPONSE_PACK_ERROR, e.getMessage());
               }
           }
           // 否则直接包装成ResultVO返回
           return new ResultVO(data);
       }
   }
   ```
   
   使用过滤器过滤器的效果是一样的；
   
   打完收工，康康效果
   
   ```java
   @PostMapping("/getUser")
   public User getUser(@Validated @RequestBody user) {
      return user;
   }
   ```
   
   此时就算我们返回的是`user`，接收到的返回就是标准格式了。
   
   ```json
   {
     "code": 1000,
     "msg": "请求成功",
     "data": {
       "name": "sunny",
       "age": 20
     }
   }
   ```
   
   

2. NOT统一响应

   不开启统一响应原因

   开发现在是开心了，可是其他系统就不开心了。举个例子：我们项目中集成了一个`健康检测`的功能，也就是这货

   ```java
   @RestController
   public class HealthController {
       @GetMapping("/health")
       public String health() {
           return "success";
       }
   }
   ```

   公司部署了一套校验所有系统存活状态的工具，这工具就定时发送`get`请求给我们系统

   > “兄弟，你死了吗？”
   > “我没死，滚”
   > “兄弟，你死了吗？”
   > “我没死，滚”

   

   ​	人家要的返回不是

   ```json
   {
   
     "code": 1000,
     "msg": "请求成功",
     "data": "success"
   }
   ```

   人家要的返回只要一个`success`，人家定的标准不可能因为你一个系统改。俗话说的好，如果你改变不了环境，那你就只能我新增不进行封装注解,因为百分之99的请求还是需要包装的，只有个别不需要，写在包装的过滤器吧？又不是很好维护，那就加个注解好了。所有不需要包装的就加上这个注解。

   ```java
   @Target({ElementType.METHOD})
   @Retention(RetentionPolicy.RUNTIME)
   public @interface NotControllerResponseAdvice {}
   ```

   然后在我们的增强过滤方法上过滤包含这个注解的方法

   ```java
   // basePackages可以指定扫描哪些包下面的controller,在Response时进行统一处理
   @RestControllerAdvice(basePackages = {"top.angeya"})
   public class ControllerResponseAdvice implements ResponseBodyAdvice<Object> {
      
       @Override
       public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
           // response是ResultVO类型，或者注释了NotControllerResponseAdvice都不进行包装
           return !(methodParameter.getParameterType().isAssignableFrom(ResultVO.class)
                   || methodParameter.hasMethodAnnotation(NotControllerResponseAdvice.class));
       }
       ...
   ```

   最后就在不需要包装的方法上加上注解

   ```java
   @RestController
   public class HealthController {
       @GetMapping("/health")
       @NotControllerResponseAdvice
       public String health() {
           return "success";
       }
   }
   ```

   这时候就不会自动封装了，而其他没加注解的则依旧自动包装。

**四、统一异常**

每个系统都会有自己的`业务异常`，比如`年龄不能小于0`之类的，这种异常并非程序异常，而是业务操作引发的异常，我们也需要进行规范的编排业务`异常状态码`，并且写一个专门处理的`异常类`，最后通过刚刚学习过的`异常拦截`统一进行处理，以及打`日志`

1. 异常状态码枚举，既然是状态码，那就肯定要实现我们的标准接口`StatusCode`；

   ```java
   @Getter
   public enum  AppCode implements StatusCode {
    
       APP_ERROR(2000, "业务异常"),
       AGE_ERROR(2001, "年龄异常");
   
       private int code;
       private String msg;
   
       AppCode(int code, String msg) {
           this.code = code;
           this.msg = msg;
       }
   }
   ```

2. 异常类

   这里需要强调一下，`code`代表`AppCode`的异常状态码，也就是2000；`msg`代表`业务异常`，这只是一个大类，一般前端会放到弹窗`title`上；最后`super(message);`这才是抛出的详细信息，在前端显示在`弹窗体`中，在`ResultVO`则保存在`data`中；

   ```java
   @Getter
   public class APIException extends RuntimeException {
       private int code;
       private String msg;
   
       // 手动设置异常
       public APIException(StatusCode statusCode, String message) {
           // message用于用户设置抛出错误详情，例如：当前年龄-5，小于0
           super(message);
           // 状态码
           this.code = statusCode.getCode();
           // 状态码配套的msg
           this.msg = statusCode.getMsg();
       }
   
       // 默认异常使用APP_ERROR状态码
       public APIException(String message) {
           super(message);
           this.code = AppCode.APP_ERROR.getCode();
           this.msg = AppCode.APP_ERROR.getMsg();
       }
   
   }
   ```

3. 最后进行统一异常的拦截，这样无论在`service`层还是`controller`层，开发人员只管抛出`API异常`，不需要关系怎么返回给前端，更不需要关心`日志`的打印；

   ```java
   @RestControllerAdvice
   public class ControllerExceptionAdvice {
       @ExceptionHandler({BindException.class})
       public ResultVO MethodArgumentNotValidExceptionHandler(BindException e) {
           // 从异常对象中拿到ObjectError对象
           ObjectError objectError = e.getBindingResult().getAllErrors().get(0);
           return new ResultVO(ResultCode.VALIDATE_ERROR, objectError.getDefaultMessage());
       }
   
       @ExceptionHandler(APIException.class)
       public ResultVO APIExceptionHandler(APIException e) {
        	log.error(e.getMessage(), e);
           // 封装成统一响应结构
           return new ResultVO(e.getCode(), e.getMsg(), e.getMessage());
       }
   }
   ```

4. 最后使用，我们的代码只需要这么写

   ```java
   if (null == user) {
       throw new APIException(AppCode.USER_IS_NULL, "用户为空" );
   }
   ```

   如果参数为空，返回值如下：

   ```json
   {
     "code": 2003,
     "msg": "用户为空",
     "data": "用户为空"
   }
   ```

   就会自动抛出`AppCode.USER_IS_NULL`状态码的响应，并且带上异常详细信息`用户为空`。



### 后端校验

在 Spring Boot 中，后端校验通常使用 Java Bean Validation（如 `javax.validation.constraints` 中的注解）和 Spring 的校验功能（如 `@Valid` 和 `@Validated`）。它可以帮助确保传入的数据符合指定规则，并简化了校验逻辑的实现。

#### 1. 添加依赖

确保你的项目中包含以下依赖（如果使用的是 Spring Boot Starter，则通常会自动包含）：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

`javax.validation.constraints` 包中包含许多常用的注解，用于对 Java Bean 的字段进行约束验证。Spring Boot 和其他框架常用这些注解进行数据校验。以下是常用的约束注解及其用法示例：

#### 1. @NotNull

用于验证字段不能为 `null`。

```java
import javax.validation.constraints.NotNull;

public class User {
    @NotNull(message = "用户名不能为空")
    private String username;

    // getters and setters
}
```

**说明**：`@NotNull` 只要求字段不能为 `null`，但可以是空字符串 `""` 或空集合 `[]`。

------

#### 2. @NotEmpty

用于验证字段不能为空。适用于字符串、集合等类型，且不能为空字符串或空集合。

```java
import javax.validation.constraints.NotEmpty;

public class User {
    @NotEmpty(message = "用户名不能为空且不能为空字符串")
    private String username;

    // getters and setters
}
```

**说明**：`@NotEmpty` 要求字段既不能为 `null`，也不能为空字符串或空集合。

------

#### 3. @NotBlank

用于验证字符串不能为空且不能是空白（包含空格的字符串也算空白）。

```java
import javax.validation.constraints.NotBlank;

public class User {
    @NotBlank(message = "用户名不能为空，且不能全是空格")
    private String username;

    // getters and setters
}
```

**说明**：`@NotBlank` 用于字符串字段，要求既不能为 `null`，也不能为 `" "`。

------

#### 4. @Size

用于验证集合、字符串、数组等的长度或大小。

```java
import javax.validation.constraints.Size;

public class User {
    @Size(min = 3, max = 15, message = "用户名长度必须在3到15个字符之间")
    private String username;

    // getters and setters
}
```

**说明**：`@Size` 常用于字符串或集合类型，验证其长度在指定范围内。

------

#### 5. @Min 和 @Max

用于验证数值类型字段的最小值和最大值。

```java
import javax.validation.constraints.Min;
import javax.validation.constraints.Max;

public class Product {
    @Min(value = 1, message = "库存数量不能小于1")
    private int stock;

    @Max(value = 100, message = "折扣率不能超过100%")
    private int discount;

    // getters and setters
}
```

**说明**：`@Min` 和 `@Max` 用于数值类型字段，`@Min` 验证字段值不能小于指定值，`@Max` 验证字段值不能超过指定值。

------

#### 6. @DecimalMin 和 @DecimalMax

用于验证 `BigDecimal` 或 `Double` 等带小数的数值的最小值和最大值。

```java
import javax.validation.constraints.DecimalMin;
import javax.validation.constraints.DecimalMax;
import java.math.BigDecimal;

public class Product {
    @DecimalMin(value = "0.0", inclusive = false, message = "价格必须大于0")
    private BigDecimal price;

    @DecimalMax(value = "1000.0", message = "价格不能超过1000")
    private BigDecimal maxPrice;

    // getters and setters
}
```

**说明**：`inclusive` 属性可以控制是否包含边界值。

------

#### 7. @Pattern

用于验证字符串是否匹配指定的正则表达式。

```java
import javax.validation.constraints.Pattern;

public class User {
    @Pattern(regexp = "^[A-Za-z0-9]+$", message = "用户名只能包含字母和数字")
    private String username;

    // getters and setters
}
```

**说明**：`@Pattern` 适用于字符串类型，常用于验证特定格式的输入（如用户名、电话号码等）。

------

#### 8. @Email

用于验证邮箱格式是否正确。

```java
import javax.validation.constraints.Email;

public class User {
    @Email(message = "邮箱格式不正确")
    private String email;

    // getters and setters
}
```

**说明**：`@Email` 适用于字符串类型，验证邮箱的格式是否符合标准。

------

#### 9. @Future 和 @Past

用于验证日期必须是未来或过去的日期。

```java
import javax.validation.constraints.Future;
import javax.validation.constraints.Past;
import java.time.LocalDate;

public class Event {
    @Future(message = "活动日期必须是未来的日期")
    private LocalDate eventDate;

    @Past(message = "生日必须是过去的日期")
    private LocalDate birthday;

    // getters and setters
}
```

**说明**：`@Future` 和 `@Past` 适用于日期类型，`@Future` 要求日期必须是未来，`@Past` 要求日期必须是过去。

------

#### 10. @FutureOrPresent 和 @PastOrPresent

用于验证日期必须是未来或当前、过去或当前的日期。

```java
import javax.validation.constraints.FutureOrPresent;
import javax.validation.constraints.PastOrPresent;
import java.time.LocalDate;

public class Event {
    @FutureOrPresent(message = "创建日期不能是过去的日期")
    private LocalDate createdDate;

    @PastOrPresent(message = "开始日期不能是未来的日期")
    private LocalDate startDate;

    // getters and setters
}
```

**说明**：`@FutureOrPresent` 要求日期必须是将来或当前的日期，`@PastOrPresent` 要求日期是过去或当前的日期。

------

#### 11. @Positive 和 @Negative

用于验证字段值必须为正或负的数。

```java
import javax.validation.constraints.Positive;
import javax.validation.constraints.Negative;

public class Account {
    @Positive(message = "账户余额必须为正数")
    private double balance;

    @Negative(message = "欠款金额必须为负数")
    private double debt;

    // getters and setters
}
```

**说明**：`@Positive` 和 `@Negative` 用于数值类型字段，要求字段值为正数或负数。

------

#### 12. @PositiveOrZero 和 @NegativeOrZero

用于验证字段值必须为正或0、负或0。

```java
import javax.validation.constraints.PositiveOrZero;
import javax.validation.constraints.NegativeOrZero;

public class Account {
    @PositiveOrZero(message = "账户余额不能为负数")
    private double balance;

    @NegativeOrZero(message = "最大债务金额不能为正数")
    private double maxDebt;

    // getters and setters
}
```

**说明**：`@PositiveOrZero` 要求字段为正或零，`@NegativeOrZero` 要求字段为负或零。

------

#### 13. @Digits

用于验证数字的整数位和小数位的位数限制。

```java
import javax.validation.constraints.Digits;
import java.math.BigDecimal;

public class Product {
    @Digits(integer = 5, fraction = 2, message = "金额整数部分最多5位，小数部分最多2位")
    private BigDecimal price;

    // getters and setters
}
```

**说明**：`@Digits` 常用于验证货币等带小数的数值。

#### 14 @AssertFalse

用于验证字段的值必须为 `false`。

```java
import javax.validation.constraints.AssertFalse;

public class Settings {
    @AssertFalse(message = "该选项不能启用")
    private boolean enabled;

    // getters and setters
}
```

**说明**：`@AssertFalse` 用于布尔类型字段，要求值为 `false`。

------

#### 15. @AssertTrue

用于验证字段的值必须为 `true`。

```java
import javax.validation.constraints.AssertTrue;

public class TermsAgreement {
    @AssertTrue(message = "您必须同意条款和条件")
    private boolean agreed;

    // getters and setters
}
```

**说明**：`@AssertTrue` 用于布尔类型字段，要求值为 `true`。

------

#### 示例：结合多个注解

```java
public class User {
    @NotNull(message = "用户名不能为空")
    @Size(min = 3, max = 15, message = "用户名长度必须在3到15个字符之间")
    private String username;

    @Email(message = "邮箱格式不正确")
    private String email;

    @Min(value = 18, message = "年龄不能小于18岁")
    @Max(value = 65, message = "年龄不能超过65岁")
    private Integer age;

    // getters and setters
}
```

以上代码中，`username` 字段需要满足 `@NotNull` 和 `@Size` 的条件，`email` 字段需要满足邮箱格式，`age` 字段需要满足年龄范围。使用这些注解可以轻松实现数据验证逻辑。

------

#### 使用说明

在 Spring Boot 项目中，可以通过 `@Valid` 注解结合控制器方法参数实现校验，例如：

```java
import org.springframework.web.bind.annotation.*;
import javax.validation.Valid;

@RestController
public class UserController {

    @PostMapping("/users")
    public String createUser(@Valid @RequestBody User user) {
        // 如果校验失败，会自动返回错误信息
        return "用户创建成功";
    }
}
```

当请求中数据不符合约束条件时，Spring Boot 会自动返回校验错误信息。



## SpringBoot3.0 版本新特性

1. JDK要求最低版本Java17
2. SpringBoot3底层默认依赖Spring6
3. 支持 Jakarta EE 10，由于 Java EE 已经变更为 Jakarta EE，包名以 javax开头的需要相应地变更为jakarta
4. Tomcat10版本
5. 支持 GraalVM 原生镜像 ，GraalVM 是 Oracle 在 2018 年发布的`一个全新的通用全栈虚拟机`，并具有高性能、跨语言交互等逆天特性，支持云原生，官网：https://www.graalvm.org/
6. 改进监控功能Micrometer和Micrometer Tracing



## 延伸

### 一、SpringBoot静态路径

SpringBoot默认的静态路径是`resources/static`。在此路径下放入媒体文件是可以直接访问的。



### 二、获取当前请求的相关参数

当前请求的参数存放在`HttpServletRequest`中，当我们需要使用请求参数或者响应对象的时候，需要在controller中的方法携带参数过来。

但是当我们需要处理的业务逻辑调用栈很深的时候，就需要一直携带着`HttpServletRequest`和`HttpServletResponse`参数，这样比较麻烦。

其实SpringMvc通过ThreadLocal实现了通过当前线程获取请求参数的方法。关键类是`org.springframework.web.context.request.RequestContextHolder`，其中的方法是`getRequestAttributes()`。

xxxxxxxxxx Object currentProxy();java

```java
// 请求对象
HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();
// 响应对象
HttpServletResponse response = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getResponse();
```



### 三、SpringBoot 支持 https

想要支持https，首先需要获取证书，证书的颁发机构（AC）越权威越好，安全性也越高（访问https网站，可以点击地址栏的锁查看证书相关的信息）。

正式的证书我们可以在腾讯云上买，有免费的和付费的。但是如果我们只是测试使用的话，可以通过java的keytool制作证书。

自制证书（生成的证书放在用户目录下）：

```bash
keytool -genkey -alias tomcat -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN" -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 365

# keytool：Java Keytool工具的名称。
# -genkey：指定创建新密钥对的命令。
# -alias tomcat：设置密钥对的别名为“tomcat”。
# -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN"：设置密钥对的持有者信息。在此示例中，持有者是“Andy”，属于“kfit”组织，位于“HaiDian”地区的“BeiJing”省，国家/地区代码为“CN”。
# -storetype PKCS12：指定密钥库类型为PKCS12格式。
# -keyalg RSA：指定使用RSA算法生成密钥对。
# -keysize 2048：指定密钥长度为2048位。
# -keystore keystore.p12：指定要创建的密钥库文件名为“keystore.p12”。
# -validity 365：指定密钥对的有效期为365天。
```

将自签或者购买的证书放在SpringBoot项目resources目录下，然后在配置文件中添加以下配置（以自签证书为例）：

```yaml
server:
  port: 443
  ssl:
    key-store: keystore.p12 # 证书路径和文件名
    key-store-password: tomcat # 证书密码
    enabled-protocols: TLSv1.1,TLSv1.2 # 启用SSL协议
    ciphers: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_RC4_128_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,SSL_RSA_WITH_RC4_128_SHA # 支持的SSL密码
```

然后启动项目就可以了。

注意：1，通过配置文件的方式，无法让项目既支持https又支持https。2，https在地址栏不输入端口的话，默认是443



### 四、拦截controller方法入参和出参

1. 使用Fastjson作为序列化框架

   入参可以在`com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter`类的`readType`方法中打断点查看。反序列化过程会读取Http的输入流内容，然后转化为字节数组，再使用字节数组通过`JSON.parseObject()`方法进行解析成控制方法对应的类型对象。

   可以使用如下代码查看请求参数：

   ```java
   new String(bytes);
   ```

   出参可以在`FastJsonHttpMessageConverter`类的`writeInternal`方法打断点查看，该方法中的`Object object`参数即是返回的对象。序列化时将object转化为json字符串，然后将字符串写入到字节流`ByteArrayOutputStream`中，最后通过Http输出流输出。

2. 使用默认的（Jackson）序列化框架

   入参可以在`org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter`的`readJavaType`方法中打断点拦截。拦截到`inputMesage`对象，只需要获取其内容输入流，然后执行一下代码即可：

   ````java
   InputStream inputStream = inputMessage.getBody();
   byte[] bytes = new byte[inputStream.available()];
   // 将会消费输入流
   inputStream.read(bytes);
   new String(bytes);
   ````

   出参在`AbstractJackson2HttpMessageConverter`类的`writeInternal`方法查看，该方法中的`Object object`参数即是返回的对象。



### 五、序列化框架

Gson、Fastjson、和Jackson是Java生态中最常用的三种JSON库，用于在 Java 对象和 JSON 数据之间进行转换。在中国使用Fastjson框架可能会更多。以下是这三个库的主要对比点：

#### Gson、Fastjson、和Jackson

##### 性能

**Fastjson**：

- 通常被认为是性能最快的JSON库之一，特别是在解析和生成JSON数据方面的速度表现优异。
- 有资料显示其在某些基准测试中，Fastjson在序列化和反序列化方面的性能优于其他两个库。

**Jackson**：

- 性能也非常出色，虽然在个别基准测试中可能略逊于Fastjson，但整体上仍属于高性能库。
- 特别是在向其中添加数据（put操作）时，Jackson的表现可能比Fastjson更好。此外，如果预先实例化`ObjectMapper`并重用，可以进一步提高Jackson的效率。

**Gson**：

- 在一些性能对比测试中，Gson的序列化和反序列化速度与Fastjson相当或略慢，但差距并不显著。
- 虽然在绝对性能上可能不及Fastjson和Jackson，但其性能对于大多数应用场景来说已经足够高效。

##### **易用性**

**Fastjson**：

- 使用API简洁，提供了一些便利的方法如`JSON.parseObj`，可以直接解析JSON字符串为对象，无需额外的包装类。
- 提供了一定程度的定制能力，如通过注解`@JSONField`来控制序列化和反序列化行为。

**Jackson**：

- 使用`ObjectMapper`进行序列化和反序列化，虽然方法相对复杂，但提供了丰富的配置选项和高度的灵活性。
- 缺乏类似于Fastjson的直接解析方法，可能需要配合其他库（如`org.json`）或手动创建对象结构来处理JSON数据。
- 需要关注线程安全问题，虽然`ObjectMapper`本身是线程安全的，但如果使用不当（如动态修改配置），可能会引入并发问题。

**Gson**：

- 提供了简单直观的API，如`Gson.fromJson`和`Gson.toJson`方法，易于上手。
- 支持泛型类型自动解析，不需要像Jackson那样显式指定类型信息。
- 通过注解（如`@SerializedName`）可以灵活地控制JSON字段名与Java属性之间的映射。

##### **功能特性与兼容性**

**Fastjson**：

- 侧重性能优化，可能在高级特性支持上不如Jackson丰富。
- 曾经存在一些安全漏洞，尤其是与`autoType`相关的高危漏洞，但已通过升级版本和关闭相关功能得到解决。

**Jackson**：

- 功能强大，支持高级特性如 polymorphic deserialization、custom serializers/deserializers、模块化扩展（如对Joda-Time、Guava、Commons Lang等库的支持）。
- 社区活跃，更新频繁，对新的Java特性和标准支持及时，具有良好的兼容性。
- 与Spring框架深度集成，是Spring默认的JSON处理工具。

**Gson**：

- 提供了基本的序列化和反序列化功能，对于简单和常见的用例足够。
- 对于复杂的数据结构、特定类型的序列化需求（如日期、枚举等）可能需要更多的自定义工作。

##### **文档与稳定性**

**Fastjson**：

- 文档缺失较多，部分特性缺乏详细说明，可能导致开发者在使用时遇到困难。
- 存在过CVE漏洞报告较少的问题，可能存在潜在的安全风险监控不足。

**Jackson**：

- 文档齐全，社区维护良好，提供了详细的用户指南和API参考。
- 安全性方面受到广泛关注，有健全的CVE追踪和及时的漏洞修复机制，稳定性较高。

**Gson**：

- 作为Google的产品，其文档相对完整，提供了清晰的使用示例和API说明。
- 虽然没有Fastjson那样的严重安全问题记录，但仍需关注官方发布的安全更新。

##### **总结**

选择哪个库主要取决于具体项目的需求：

- 如果对性能有极致要求，且愿意牺牲一部分高级特性支持和文档完备性，Fastjson可能是一个合适的选择，尤其是在处理大量数据或对响应时间敏感的场景。
- 如果追求功能全面、易用性好、文档完善、安全性高，并且需要与Spring框架无缝集成，或者处理复杂的JSON结构和定制需求，Jackson通常是最佳选择。
- 对于简单项目或对Google技术栈有一定依赖，希望快速上手且满足基本JSON处理需求，Gson以其简洁的API和良好的兼容性可以成为不错的选择。



#### 这些框架的使用

**Gson**

maven依赖

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
```

java示例代码

```java
// 创建用户对象
User user = new User();
user.setName("angeya");
user.setAge(18);
user.setFriendList(Arrays.asList("lili", "tony", "mary"));

// 序列化
Gson gson = new Gson();
String json = gson.toJson(user);
System.out.println(json);

// 反序列化
User user2 =gson.fromJson(json, User.class);
System.out.println(user2);
```

但是，将Json字符串转换为List对象的时候，就有一点不同了。

由于List接口带泛型，如果还调用   fromJson(String, Class)方法，那么返回的虽然还是个List集合，但是集合里面的数据却不是User对象，而是Map对象，并将User的属性以键值对的形式存放在Map的实例中。类似的，Map和Set等带泛型的接口也是如此。
```java
// Gson包提供TypeToken<>类获取Type对象，不需要重写方法，只需获取其对象并调用getType()方法即可
List<User> userList = gson.fromJson(json, new TypeToken<List<User>>(){}.getType());
System.out.println(userList);
```



**Jackson**

Jackson 有三个核包，分别是 Streaming、Databid、Annotations，通过这些包可以方便的对 JSON 进行操作。

> Streaming 在 jackson-core 模块。 定义了一些流处理相关的 API 以及特定的 JSON 实现。
> Annotations 在 jackson-annotations 模块，包含了 Jackson 中的注解。
> Databind 在 jackson-databind 模块， 在 Streaming 包的基础上实现了数据绑定，依赖于 Streaming 和 Annotations 包，因此大多数情况下我们只需要添加 jackson-databind 依赖项，就可以使用 Jackson 功能了

maven依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

java示例代码

```java
// import com.fasterxml.jackson.databind.ObjectMapper;
// 创建映射对象
ObjectMapper objectMapper = new ObjectMapper();
// java对象序列化为json
String jsonStr = objectMapper.writeValueAsString(demoData);
System.out.println(jsonStr);

// json字符串解析为java对象
DemoData demoData = objectMapper.readValue(jsonStr, DemoData.class);
System.out.println(demoData);
```

 **Fastjson**

maven依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.83</version>
</dependency>
```

java示例代码

```java
// java对象序列化为json
String jsonStr = JSONObject.toJSONString(DemoData);
// 输出包含类型的json字符串，注意Double的数值后面会加上D，如 12.0D
// String jsonStr = JSONObject.toJSONString(apiOperationBean, SerializerFeature.WriteClassName);
System.out.println(jsonStr);

// json字符串解析为java对象
DemoData demoData = JSONObject.parseObject(jsonStr, DemoData.class);
System.out.println(demoData);
```



#### SpringBoot中使用Fastjson

在Spring Boot中默认使用Jackson作为JSON序列化和反序列化框架。

1. 引入fastjson依赖

在项目的pom.xml文件中引入fastjson依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.83</version>
</dependency>
```

2. 自定义FastJsonHttpMessageConverter

自己定义一个FastJsonHttpMessageConverter来替换掉Jackson，使得Spring Boot使用fastjson进行处理。代码如下：

```java
@Configuration
public class FastJsonHttpMessageConverterConfig {

    @Bean
    public HttpMessageConverters fastJsonHttpMessageConverters() {
        // 1.创建FastJson信息转换对象
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();

        // 2.创建FastJson配置对象
        FastJsonConfig config = new FastJsonConfig();
        config.setSerializerFeatures(
            SerializerFeature.PrettyFormat,
            // List为null输出为空数组
            SerializerFeature.WriteNullListAsEmpty,
            // 禁用循环引用检测
            SerializerFeature.DisableCircularReferenceDetect,
            // 日期使用格式化输出（默认输出时间戳）
            SerializerFeature.WriteDateUseDateFormat);
        	// 可以通过设置SerializerFeature.WriteClassName特性，全局设置序列化输出类型
        
        // 设置日期格式
        config.setDateFormat("yyyy-MM-dd HH:mm:ss");

        // 3.处理中文乱码问题
        List<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        converter.setSupportedMediaTypes(mediaTypes);

        // 4.将FastJson配置给到转换器，并添加到HttpMessageConverter转换器列表中
        converter.setFastJsonConfig(config);
        return new HttpMessageConverters(converter);
    }
}
```

以上代码通过@Bean注解将`FastJsonHttpMessageConverter`注册为Bean，并添加到`HttpMessageConverter`转换器列表中。在`FastJsonConfig`中设置了一些fastjson的配置参数，如是否格式化输出、序列化null值的规则及禁用循环引用检测等。

3. 配置Spring Boot使用FastJson替代Jackson

在Spring Boot的配置文件application.yml（或application.properties）中增加配置项，使得Spring Boot使用FastJson进行JSON处理而非默认的Jackson。

```yaml
# 方法一 可能已经过时
spring:
  http:
    converters:
      preferred-json-mapper: fastjson
# 方法二
spring:
  mvc:
    converters:
       preferred-json-mapper: fastjson
```

4. 测试

在Controller中使用@ResponseBody注解将返回结果序列化成JSON格式并返回给客户端，如下所示：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/info")
    public User getUser(){
        User user = new User();
        user.setId("123");
        user.setName("小明");
        user.setAge(24);
        user.setAddress("江苏南京");
        user.setBirthday(new Date());
        return user;
    }
}
```

完成以上操作后，就可以使用fastjson作为Spring Boot的JSON序列化和反序列化框架了。



### 六、读取resources目录下的文件

1. 方式1（项目打成 jar 包后，路径有问题，导致读取不到文件内容，因此只能在开发环境使用）

   ```java
   // 注意getResource("")里面是空字符串
   String path = this.getClass().getClassLoader().getResource("").getPath();
   // 如果路径中带有中文会被URLEncoder,因此这里需要解码
   String filePath = URLDecoder.decode(path, "UTF-8");
   ```

   

2. 方式2（获取 jar 包中文件的输入流，可以在生产环境使用）

   ```java
   /* 可以使用以下几种方式获取输入流 */
   
   // 获取类加载器后使用getResourceAsStream方法获取流
   InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream(fileName);
   // 直接使用getResourceAsStream方法获取流
   InputStream inputStream = this.getClass().getResourceAsStream("/" + fileName);
   // 通过ClassPathResource的方式获取
   ClassPathResource classPathResource = new ClassPathResource(fileName);
   InputStream inputStream = classPathResource.getInputStream();
   ```



### 七、SpringBoot启动完成监听器<a id="AppLaunchListener"></a>

如果需要在SpringApplication启动后运行某些特定代码，可以实现`ApplicationRunner`或`CommandLineRunner`接口。两个接口以相同的方式工作。

**实现ApplicationRunner接口**

```java
@Component
public class AppLaunchListener implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // do some thing
    }
}
```

**实现CommandLineRunner接口**

```java
@Component
public class AppLaunchListener implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        // do some thing
    }
}
```

但是要注意，上面的这两个`run`方法执行的时候，SpringBoot项目已经启动成功了，可以对外提供服务了（比如接口可以接收请求了）。是在由于`run`方法是在主线程中运行的，一旦抛出异常没有处理，将会导致主线程崩溃，进而导致整个SpringBoot应用崩溃，所以应该在`run`方法中捕获异常，除非是必要的步骤执行失败了就停止启动。



### 八、SpringBoot自定义Starter

Spring框架也使用了SPI机制来实现扩展点的加载和扩展。在Spring中，常见的SPI机制的使用包括：

1. **Spring Bean加载**：Spring的ApplicationContext接口提供了多种实现，例如ClassPathXmlApplicationContext、AnnotationConfigApplicationContext等。这些实现类通过SPI机制加载，Spring在classpath下查找所有的META-INF/spring.factories文件，并加载其中指定的ApplicationContext实现类。
2. **Spring扩展点**：Spring提供了多种扩展点，例如BeanPostProcessor、BeanFactoryPostProcessor等。开发者可以实现这些接口，并在META-INF/spring.factories文件中指定实现类，从而让Spring在启动时加载并执行这些扩展点。
3. **Spring Boot自动配置**：Spring Boot通过SPI机制实现了自动配置的功能。在classpath下的META-INF/spring.factories文件中，Spring Boot会列出各种自动配置类，当启动Spring Boot应用程序时，它会根据当前的环境和依赖项自动配置应用程序所需的Bean。



其中自定义`starter`就是基于`SpringBoot`自动状态实现的。

#### 1 编写自己的Starter

这里以一个简单打招呼的简单Starter作为例子，理解其流程即可。

如：配置一个打招呼语 Hello 和一个名字 July。调用Starter的处理方法时出入名字 sunny，可以返回 “Hello Sunny， I am July”

1. pom.xml中引入必要依赖

   ```xml
   <!--  定义starter的依赖坐标  -->
   <groupId>top.angeya</groupId>
   <artifactId>simple-spring-starter</artifactId>
   <version>1.0-SNAPSHOT</version>
   
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.5.6</version>
   </parent>
   
   <dependencies>
       <!--  SpringBoot  -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
       </dependency>
   
       <!--  用于配置文件的提示，方便使用者编写配置信息  -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-configuration-processor</artifactId>
           <optional>true</optional>
       </dependency>
   </dependencies>
   ```

   

2. 编写自动装配配置类，SpringBoot装配此Starter时，从这个类开始

   ```java
   @Configuration
   @EnableConfigurationProperties(SimpleProperty.class) // 定义配置类SimpleProperty
   public class AutoConfiguration {
   
       // 装配业务逻辑Bean，这一步可以没有（这就需要在使用的项目中手动声明为Bean），也可以直接在类中使用Component等注解
       @Bean
       public SimpleHandler simpleHandler() {
           return new SimpleHandler();
       }
   
   }
   ```

3. 配置信息类，simple为配置的前缀

   ```java
   @ConfigurationProperties("simple")
   public class SimpleProperty {
   
       private String greet;
   
       private String name;
       
       // getter,setter...
   }
   ```

   

4. 编写Starter业务逻辑

   ```java
   public class SimpleHandler {
   
       // 获取配置信息
       @Autowired
       private SimpleProperty simpleProperty;
   
       // 业务逻辑方法
       public String handleGreet(String yourName) {
           return simpleProperty.getGreet() + " " + yourName + ", I am " + simpleProperty.getName();
       }
   }
   ```

5. 接下来是实现自动注入的关键，SpringBoot会去扫描依赖Jar包中META-INF/spring.factories中的内容，因此在resources目录下新建META-INF/spring.factories，写下配置信息：

   ```
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   top.angeya.sss.AutoConfiguration
   ```

   

#### 2 使用Starter

经过以上的步骤，就可以实现一个简单的自定义Starter了。一下是使用方法

1. pom.xml引入Starter依赖

   ```xml
    <dependency>
        <groupId>top.angeya</groupId>
        <artifactId>simple-spring-starter</artifactId>
        <version>1.0-SNAPSHOT</version>
   </dependency>
   ```

2. 编写配置文件 application.yml

   ```yaml
   simple:
     greet: Hello
     name: July
   ```

3. 使用自定义Starter的Bean完成业务逻辑

   ```java
   @Service
   public class TestService {
       @Autowired
       SimpleHandler simpleHandler;
   
       @PostConstruct
       public void test() {
           String sentence = simpleHandler.handleGreet("sunny");
           System.out.println(sentence);
       }
   }
   ```

启动项目后输入结果如下：

```
Hello sunny, I am July
```



### 九、SSE服务端推送

服务器向浏览器推送信息，除了 WebSocket，还有一种方法：Server-Sent Events（以下简称 SSE）。

**SSE的本质**

严格地说，HTTP 协议无法做到服务器主动推送信息。但是，有一种变通方法，就是服务器向客户端声明，接下来要发送的是流信息（streaming）。

也就是说，发送的不是一次性的数据包，而是一个数据流，会连续不断地发送过来。这时，客户端不会关闭连接，会一直等着服务器发过来的新的数据流，视频播放就是这样的例子。本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE/Edge，其他浏览器都支持。
SSE请求之后，在浏览器开发者工具网络模板中查看时，响应头的Content-Type类型为: `text/event-stream`

**SSE的特点**

SSE 与 WebSocket 作用相似，都是建立浏览器与服务器之间的通信渠道，然后服务器向浏览器推送信息。

总体来说，WebSocket 更强大和灵活。因为它是全双工通道，可以双向通信；SSE 是单向通道，只能服务器向浏览器发送，因为流信息本质上就是下载。如果浏览器向服务器发送信息，就变成了另一次 HTTP 请求。
但是，SSE 也有自己的优点。

- SSE 使用 HTTP 协议，现有的服务器软件都支持。WebSocket 是一个独立协议。
- SSE 属于轻量级，使用简单；WebSocket 协议相对复杂。
- SSE 默认支持断线重连，WebSocket 需要自己实现。
- SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据。
- SSE 支持自定义发送的消息类型。
因此，两者各有特点，适合不同的场合。

**Springboot中使用SSE **

SpringMVC框架就支持SSE，不需要引入额外依赖，服务端使用和普通接口差不多。

1. 后端代码
```java
@RestController
public class DemoController {
    
    /**
     * 可以保留链接，随时异步发送
     */
    public static final ConcurrentHashMap<Long, SseEmitter> SSE_EMITTER_MAP = new ConcurrentHashMap<>();

    /**
     * 和接收普通的http请求没有区别，区别这是只是返回的值为SseEmitter对象而已
     * @param userId 用户id，可以用于标识当前连接
     * @return SseEmitter 对象，告诉浏览器，这是长连接
     */
    @GetMapping("/test")
    public SseEmitter test(Long userId) {
        //设置默认的超时时间60秒，超时之后服务端主动关闭连接。
        SseEmitter emitter = new SseEmitter(60 * 1000L);
        SSE_EMITTER_MAP.put(userId,emitter);
        emitter.onTimeout(() -> SSE_EMITTER_MAP.remove(userId));
        new Thread(() -> {
            try {
                emitter.send("消息来喽1");
                TimeUnit.SECONDS.sleep(1);
                emitter.send("消息来喽2");
                TimeUnit.SECONDS.sleep(1);
                emitter.send("消息来喽3");
                TimeUnit.SECONDS.sleep(2);
                // 自定义事件，可以用于通知客户端关闭连接（否则断连可客户端会一直重连）
                SseEventBuilder sseEventBuilder = SseEmitter.event().name("finish").id("6666").data("结束啦");
                emitter.send(sseEventBuilder);
                emitter.complete();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
        return emitter;
    }
}
```
2. 前端代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SSE-Demo</title>
</head>
<body>
<div id="message">
    这里展示消息
</div>
<script>
    // 判断浏览器是否支持SSE
    if (!window.EventSource) {
        console.log("你的浏览器不支持SSE")
    } else {
        // 主动进行建立长连接
        let source = new EventSource('http://127.0.0.1:8086/test?userId=1')
        let innerHTML = ''

        // 监听服务器端发来的事件：open
        source.onopen = function (e) {
            innerHTML += "onopen：准备就绪，可以开始接收服务器数据" + "<br/>"
            document.getElementById("message").innerHTML = innerHTML
        }
        // 监听服务器端发来的事件：message
        source.onmessage = function (e) {
            innerHTML += "onmessage:" + e.data + "<br/>"
            document.getElementById("message").innerHTML = innerHTML
        }
        // 自定义事件，发送数据不会触发onmessage方法
        source.addEventListener('finish', function (e) {
            // 主动关闭EventSource，否则服务端单方关闭客户端会自动重连
            source.close()
            innerHTML += "长连接事件处理完毕，通知服务端关闭EventSource" + "<br/>"
            document.getElementById("message").innerHTML = innerHTML
        }, false)
        // 监听服务器端发来的事件：error
        source.onerror = function (e) {
            if (e.readyState === EventSource.CLOSED) {
                innerHTML += "sse连接已关闭" + "<br/>"
            } else {
                console.log(e)
            }
        }
    }
</script>
</body>
</html>
```



### 十、数据库连接池

#### 常见连接池

在Spring Boot项目中，常用的数据库连接池有以下几种：

1. HikariCP：HikariCP是目前性能最好的Java连接池之一，具有快速、高效和轻量级等特点。它在处理高并发连接请求时表现出色，可以提供更快的响应时间和更低的资源占用率。
2. Apache Commons DBCP2：Apache Commons DBCP2是一个基于Apache软件基金会的通用连接池。它提供了一些高级功能，比如自动回收空闲连接、连接泄漏检测等。同时，它也是一个轻量级的连接池，适用于小型和中型应用程序。
3. Tomcat JDBC Pool：Tomcat JDBC Pool是Apache Tomcat Web服务器的官方JDBC连接池。它具有高性能、高可靠性、安全性和可扩展性等优点。同时，它还支持异步查询、连接泄漏检测和生命周期监听等高级功能。
4. Druid：Druid是阿里巴巴开源的一个高性能、可扩展的JDBC连接池。它可以监控SQL执行情况、统计执行时间和频率等信息，并提供了一些可视化的监控界面。同时，它也支持连接泄漏检测、密码加密、多数据源等高级功能。

其中HikariCP是SpringBoot默认数据库连接池。

#### HikariCP连接池的常见配置

```yaml
# 连接池中允许的最小连接数。缺省值和maximum-pool-size一样
spring.datasource.hikari.minimum-idle=10
# 连接池中允许的最大连接数。缺省值：10
spring.datasource.hikari.maximum-pool-size=100
# 自动提交，缺省值true
spring.datasource.hikari.auto-commit=true
# 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟
spring.datasource.hikari.idle-timeout=30000
# 连接池名字
spring.datasource.hikari.pool-name=HikariCP
# 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒
spring.datasource.hikari.max-lifetime=1800000
# 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒
spring.datasource.hikari.connection-timeout=30000
# 数据库连接测试语句
spring.datasource.hikari.connection-test-query=SELECT 1
```

#### 两种连接池的区别

mysql服务中的数据库连接池，和SpringBoot项目中的Hikari数据库连接池的区别和联系是什么？

区别：

1. 层级不同：MySQL服务中的数据库连接池是在MySQL数据库服务器上实现的，用于管理和提供与MySQL数据库的连接。而Spring Boot项目中的Hikari数据库连接池是在应用程序层级上实现的，用于管理与应用程序中的数据源（如MySQL）的连接。
2. 功能范围不同：MySQL服务中的数据库连接池主要负责管理和复用与MySQL数据库的连接，以提高性能和效率。它通常包含了一些针对MySQL数据库的特定优化和功能。而Hikari数据库连接池是一个通用的Java连接池，支持多种数据库，包括MySQL。它提供了一套通用的连接池管理功能，不局限于某个具体的数据库。

联系：

1. 都是用于连接管理：无论是MySQL服务中的数据库连接池还是Spring Boot项目中的Hikari数据库连接池，它们都用于管理和复用数据库连接，以提高应用程序的性能和资源利用率。
2. 都可以配置参数：两者都可以通过配置一些参数来调整连接池的行为，比如最大连接数、最小空闲连接数、连接超时等。这些参数的设置可以根据应用程序的需求进行调整。

#### 获取HikariCP连接池的使用情况

```java
@Resource
DataSource dataSource;

@GetMapping("test")
public String test() {
    // 确保数据连接池是hikari
	HikariPoolMXBean hikariPoolMXBean = ((HikariDataSource) dataSource).getHikariPoolMXBean();
	int total = hikariPoolMXBean.getTotalConnections();
	int wait = hikariPoolMXBean.getThreadsAwaitingConnection();
	int active = hikariPoolMXBean.getActiveConnections();
    // 并发数大于最大连接数时候，线程无法获取连接，则需要等待，正在使用时活跃
	return "连接池总数：" + total + " 等待："+ wait + " 活跃：" + active;
}
```

> 注意：DataSource 是 javax.sql 包下面的，一般容易导错类。



### 十一、Transactional 事务支持

#### 事务失效的11种情况

**1，访问权限问题**

如果我们自定义的事务方法（即目标方法），它的访问权限不是`public`，而是private、default或protected的话，spring则不会提供事务功能。

在`AbstractFallbackTransactionAttributeSource`类的`computeTransactionAttribute`方法中有个判断，如果目标方法不是public，则`TransactionAttribute`返回null，即不支持事务。

但是事务方法中调用的各个数据库操作的方法可以是private的，这些内部的方法使用同一个事务。

**2，方法用final修饰**

spring事务底层使用了aop，也就是通过jdk动态代理或者cglib，帮我们生成了代理类，在代理类中实现的事务功能。

如果某个方法用final修饰了，那么在它的代理类中，就无法重写该方法，而添加事务功能。

如果某个方法是static的，同样无法通过动态代理，支持事务。

**3，方法内部调用**

方法内部调用，使用的是this对象，而不是代理对象，所以无法增强事务。

这时候可以把需要事务方法抽取到一个新的Bean中。

或者在当前类中注入自己，然后就可以通过代理对象调用事务方法了。如下：

```java
@Servcie
public class ServiceA {
   @Autowired
   prvate ServiceA serviceA;

   public void save() {
       // 这样可以用代理对象
       serviceA.doSomeThing();
   }

   @Transactional(rollbackFor=Exception.class)
   public void doSomeThing() {
       // do some thing, such as save data
    }
 }
```

当然还可以在该Service类中使用AopContext.currentProxy()获取代理对象，这个可以参考AOP部分。

**4，未被spring管理**

使用spring事务的前提是：对象要被spring管理，需要创建bean实例。

**5，多线程调用**

虽然在事务方法内部，通过多线程来并发操作数据库，可以提高效率，但是这样就会使得事务部分失效。

spring的事务是通过数据库连接来实现的。当前线程中保存了一个map，key是数据源，value是数据库连接。我们说的同一个事务，其实是指同一个数据库连接，只有拥有同一个数据库连接才能同时提交和回滚。如果在不同的线程，拿到的数据库连接肯定是不一样的，所以是不同的事务。

**6，表不支持事务**

周所周知，在mysql5之前，默认的数据库引擎是`myisam`。

它的好处就不用多说了：索引文件和数据文件是分开存储的，对于查多写少的单表操作，性能比innodb更好。在创建表的时候，只需要把`ENGINE`参数设置成`MyISAM`即可。

所以在实际业务场景中，myisam使用的并不多。在mysql5以后，myisam已经逐渐退出了历史的舞台，取而代之的是innodb。

> 有时候我们在开发的过程中，发现某张表的事务一直都没有生效，那不一定是spring事务的锅，最好确认一下你使用的那张表，是否支持事务。

**7，错误的传播特性**

我们在使用`@Transactional`注解时，是可以指定`propagation`参数的。

该参数的作用是指定事务的传播特性，spring目前支持7种传播特性：

- `REQUIRED` 如果当前上下文中存在事务，那么加入该事务，如果不存在事务，创建一个事务，这是默认的传播属性值。
- `SUPPORTS` 如果当前上下文存在事务，则支持事务加入事务，如果不存在事务，则使用非事务的方式执行。
- `MANDATORY` 如果当前上下文中存在事务，否则抛出异常。
- `REQUIRES_NEW` 每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。
- `NOT_SUPPORTED` 如果当前上下文中存在事务，则挂起当前事务，然后新的方法在没有事务的环境中执行。
- `NEVER` 如果当前上下文中存在事务，则抛出异常，否则在无事务环境上执行代码。
- `NESTED` 如果当前上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务。

如果我们在手动设置propagation参数的时候，把传播特性设置错了，比如：

```java
@Transactional(propagation = Propagation.NEVER)
public void add(UserModel userModel) {
    saveData(userModel);
    updateData(userModel);
}
```

我们可以看到add方法的事务传播特性定义成了 Propagation.NEVER，这种类型的传播特性不支持事务，如果有事务则会抛异常。

目前只有这三种传播特性才会创建新事务：REQUIRED，REQUIRES_NEW，NESTED。

**8，自己吞了异常**

如果想要spring事务能够正常回滚，必须抛出它能够处理的异常。如果没有抛异常，则spring认为程序是正常的。

所以，可以捕获异常，但是做了一些操作之后还需要把异常抛出来，否则事务不会回滚。

**9，异常类型不对**

默认情况下只会回滚`RuntimeException`（运行时异常）和`Error`（错误），对于普通的Exception（非运行时异常），它不会回滚。

所以适当的加上`rollbackFor=Exception.class`配置，或者确认好回滚异常。也可以直接设置成`Throwable`

**10，嵌套事务回滚多了**

看如下代码：

```java
public class UserService {
    @Autowired
    private RoleService roleService;

    @Transactional
    public void saveData() throws Exception {
        // some DB operate
        roleService.doOtherThing();
    }
}

@Service
public class RoleService {

    @Transactional(propagation = Propagation.NESTED)
    public void doOtherThing() {
        // some DB operate
    }
}
```

这种情况使用了嵌套的内部事务，原本是希望调用roleService.doOtherThing方法时，如果出现了异常，只回滚doOtherThing方法里的内容，不回滚 saveData 里的内容，但事实是，saveData 也回滚了。

为什么呢？因为 doOtherThing 方法出现了异常，没有手动捕获，会继续往上抛，到外层 saveData 方法的代理方法中捕获了异常。所以，这种情况是直接回滚了整个事务，不只回滚单个保存点。

所以 saveData  方法应该改为如下：

```java
@Transactional
public void saveData() throws Exception {
    // some DB operate
    try {
        roleService.doOtherThing();
    } catch (Exception e) {
        log.error(e.getMessage(), e);
    }
}
```

**11，未开启事务**

如果你使用的是springboot项目，那么你很幸运。因为springboot通过`DataSourceTransactionManagerAutoConfiguration`类，已经默默的帮你开启了事务。

你所要做的事情很简单，只需要配置`spring.datasource`相关参数即可。

但如果你使用的还是传统的spring项目，则需要在applicationContext.xml文件中，手动配置事务相关参数。如果忘了配置，事务肯定是不会生效的。



### 十二、加载其他jar包中的Bean

在SpringBoot项目中引入其他jar包，如果jar包中包含Spring Bean的定义，默认情况下在当前SpringBoot项目中是不能加载到这些Bean的，因为SpringBoot默认值扫描当前启动类所在的包的Bean、以及jar包中META-INF/spring.factories并配置为SpringBoot Starter的包。

如果想要在当前项目中加载到到普通jar包的定义Spring Bean，需要再启动类上加入注解`@ComponentScan("com.example.test")`来指定Spring需要扫描的包，但是指定了扫描特定包之后，Spring不会扫描当前项目下的Bean了，所以需要同事指定当前项目的包。如下

```java
package com.example.demo;

// import ...

// 指定当前包名和jar包中需要扫描的包
@ComponentScan({"com.example.demo", "com.example.test"})
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpmApplication.class, args);
}
```


当然，你也可以选择定义SpringBoot Starter啦。

### 十三、jar包使用外部配置文件

每次打完jar包部署到服务器的时候，都要进去改配置文件application.yaml，很麻烦而且容易出错，如果可以统一读取外部配置文件，就不用每次都去修改了（如果配置项没有调整的话）。

其实Spring Boot 允许你在运行应用程序时通过指定 `--spring.config.location` 选项来指定外部配置文件的路径。比如，我们可以将 `application.yml` 放置在应用程序 Jar 包所在的目录，然后在启动命令中指定 `--spring.config.location=file:./` 来加载外部的配置文件，当然也可以使用完整路径来指定文件。

完整启动命令示例：

```shell
java -jar --spring.config.location=file:application.yml xxx.jar
```

> 理论上，在SpringBoot的配置文件中指定 `spring.config.location`配置项也是可以的，但是目前实践并没有生效。



### 十四、Spring事件机制

#### 介绍

Spring的事件机制是Spring框架中的一个重要特性，基于观察者模式实现，它可以实现应用程序中的解耦，提高代码的可维护性和可扩展性。Spring的事件机制包括事件、事件发布、事件监听器等几个基本概念。

事件有其便利的一面，但是用多了也容易导致混乱，所以在实际项目中，我们还是要谨慎选择是否使用 Spring 事件。

事件发布流程中，有三个核心概念：

- 事件源（ApplicationEvent）：这个就是你要发布的事件对象。
- 事件发布器（ApplicationEventPublisher）：这是事件的发布工具。
- 事件监听器（ApplicationListener）：这个相当于是事件的消费者。

以上三个要素，事件源和事件监听器都可以有多个，事件发布器（通常是由容器来扮演）一般来说只有一个。

#### 使用

1. 定义事件

   ```java
   public class MyEvent extends ApplicationEvent {
       private String name;
       public MyEvent(Object source, String name) {
           super(source);
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "MyEvent{" +
                   "name='" + name + '\'' +
                   "} " + super.toString();
       }
   }
   ```

   在具体实践中，事件源并非一定要继承自 ApplicationEvent，事件源也可以是一个普通的 Java 类，如果是普通的 Java 类，系统会自动将之封装为一个 PayloadApplicationEvent 对象去发送。

2. 发布事件

   接下来通过事件发布器将事件发布出去。Spring 中事件发布器有专门的接口 ApplicationEventPublisher：

   ```java
   @FunctionalInterface
   public interface ApplicationEventPublisher {
       // 继承自ApplicationEvent的事件
       default void publishEvent(ApplicationEvent event) {
       	publishEvent((Object) event);
       }
       // 所有事件
       void publishEvent(Object event);
   }
   ```

   AbstractApplicationContext 实现了该接口并重写了接口中的方法，所以我们平时使用的 AnnotationConfigApplicationContext 或者 ClassPathXmlApplicationContext，里边都是可以直接调用事件发布方法的。

   ```java
   @Autowired
   ApplicationContext context;
   
   public void sendEvent() {
       this.context.publishEvent(new MyEvent(this, "hello"));
   }
   ```

3. 事件监听

   事件发布之后，我们还需要事件监听器去监听并处理事件。事件监听器有两种定义方式。

   第一种是自定义类实现 ApplicationListener 接口：

   ```java
   @Component
   public class MyEventListener implements ApplicationListener<MyEvent> {
       @Override
       public void onApplicationEvent(MyEvent event) {
           System.out.println("event = " + event);
       }
   }
   ```

   第二种方式则是通过注解去标记事件消费方法：

   ```java
   @Component
   public class MyService {
       @EventListener(value = MyEvent.class)
       public void hello(MyEvent event) {
           System.out.println("event = " + event);
       }
   }
   ```

#### 异步事件处理

默认事件的发布与处理是同步的，如果想要一步处理事件，只需要在监听器处理方法上面添加`@Async`注解即可。同时别忘记了在启动类上增加`@EnableAsync`注解。

### 十五、通过ApplicationContext

要获取 Spring 容器中所有的 Bean 对象，可以使用 `ApplicationContext` 的 `getBeansOfType()` 方法或 `getBean()` 方法结合 `getBeanDefinitionNames()` 方法。下面是如何实现的详细步骤：

#### 1，getBeansOfType()获取所有Bean对象

如果你想获取所有特定类型的 Bean 对象，可以使用 `getBeansOfType()` 方法。这个方法返回一个包含所有符合指定类型的 Bean 名称和实例的 `Map`。

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.Map;

public class AllBeansExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        // 获取所有 Object 类型的 Bean 实例
        Map<String, Object> allBeans = context.getBeansOfType(Object.class);

        // 输出所有的 Bean 名称和对象
        for (Map.Entry<String, Object> entry : allBeans.entrySet()) {
            System.out.println("Bean Name: " + entry.getKey() + ", Bean Object: " + entry.getValue());
        }
    }
}
```

#### 2，getBeanDefinitionNames()与 getBean() 组合

你可以先获取所有的 Bean 名称，然后使用 `getBean()` 方法逐个获取每个 Bean 实例。

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AllBeansExample {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        // 获取所有 Bean 名称
        String[] beanNames = context.getBeanDefinitionNames();

        // 遍历 Bean 名称，获取每个 Bean 对象
        for (String beanName : beanNames) {
            Object bean = context.getBean(beanName);
            System.out.println("Bean Name: " + beanName + ", Bean Object: " + bean);
        }
    }
}
```

### 解释

- **`getBeanDefinitionNames()`**: 返回所有 Bean 的名称。
- **`getBean(String name)`**: 根据 Bean 名称获取对应的 Bean 对象实例。
- **`getBeansOfType(Class<T> type)`**: 获取指定类型（或所有类型）Bean 的实例映射。



### 十六、Spring Boot Actuator 

#### 1. Spring Boot Actuator 概述

Spring Boot Actuator 是 Spring Boot 提供的用于管理和监控应用的工具集。它暴露了一些预定义的端点（例如 `/actuator/health`、`/actuator/info`），通过这些端点，你可以轻松地查看应用的健康状态、配置信息、监控指标等。Actuator 还支持自定义和扩展，允许你添加自己的监控信息。

#### 2. Actuator 原理

Spring Boot Actuator 基于 Spring Framework 的核心功能，使用了 Spring 的 `@Endpoint` 和 `@WebEndpoint` 等注解来定义和暴露各种监控端点。Actuator 集成了以下几个 Spring 组件：

- **Spring MVC 或 Spring WebFlux**：处理 HTTP 请求，暴露 REST 风格的端点。
- **Spring Boot Autoconfiguration**：自动配置健康检查、指标、环境等信息。
- **Micrometer**：用于监控和度量的接口，整合了 Actuator 提供的指标功能，支持输出到 Prometheus、Graphite 等监控系统。

#### 3.Spring Boot Actuator 

Spring Boot Actuator 默认暴露了一些有用的端点，以下是一些常见的端点：

- **`/actuator/health`**：显示应用的健康状态（如数据库连接、缓存状态等）。
- **`/actuator/info`**：显示应用的自定义信息（通常来自 `application.properties` 中的配置信息）。
- **`/actuator/metrics`**：显示应用的各类性能指标（如 JVM 内存使用情况、CPU 使用情况等）。
- **`/actuator/beans`**：显示 Spring 容器中所有的 Bean。
- **`/actuator/env`**：显示应用的环境属性。
- **`/actuator/loggers`**：查看和修改日志级别。

这些端点并不是默认全部暴露的，只有 `health` 和 `info` 是默认可访问的，其余的需要通过配置文件启用。

#### 4. Actuator 使用步骤

**4.1 引入 Actuator 依赖**

首先需要在项目中添加 `spring-boot-starter-actuator` 依赖：

Maven：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Gradle：

```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

**4.2 配置 Actuator 端点**

通过配置文件启用需要的端点，例如你可以在 `application.properties` 或 `application.yml` 中指定哪些端点可以对外暴露：

application.properties：

```properties
management.endpoints.web.exposure.include=health,info,metrics,beans,env
```

application.yml：

```yaml
endpoints:
    web:
      exposure:
        include: health,info,metrics,beans,env
```

如果你想排除某些端点，可以使用：

```properties
management.endpoints.web.exposure.exclude=loggers
```

**4.3 访问 Actuator 端点**

在启动项目后，你可以通过以下方式访问暴露的端点：

- 健康检查：`http://localhost:8080/actuator/health`
- 应用信息：`http://localhost:8080/actuator/info`
- 性能指标：`http://localhost:8080/actuator/metrics`

#### 5. 健康检查与自定义健康检查

**5.1 健康检查端点**

`/actuator/health` 是一个非常常用的端点，用于显示应用的整体健康状况。默认的健康检查包括对数据库连接、Redis、内存等资源的检查。

可以通过配置来显示详细的健康信息：

```properties
management.endpoint.health.show-details=always
```

Actuator会扫描注册了的指示器，根据这些指示器提供的健康状态信息，最后汇总成当前应用的健康状态信息。如果有一个指示器状态Down，则当前应用状态为Down。如Mysql、Redis等常见组件在它们的Starter包中，都提供了自己的指示器。

如果你不想在健康状态检查的时候，跳过某个组件，可以通过下面配置实现

```yaml
management:
  # 健康状态检查跳过Sentinel
  health:
    sentinel:
      enabled: false
```

如果你不想自己暴漏原生的端点，想通过自己写接口获取健康状态，可以通过一下代码实现

首先配置关闭actuator接口

```yaml
management:
  server:
    # 修改端口
    port: -1
  endpoints:
    # 关闭所有actuator接口
    enabled-by-default: false
```

然后再代码中获取健康状态

```java
@Resource
private Optional<HealthEndpoint> healthEndpointOptional;
 /**
 * 服务健康状态检查
 *
 * @return 健康状态
 */
public boolean serviceHealthCheck() {
    // 健康状态默认为false
    boolean isHealthy = false;
    if (healthEndpointOptional.isPresent()) {
        // 通过Future异步获取健康状态，如果超过8s没有响应(down的时候检查会很慢)，则直接返回down
        Future<HealthComponent> future = CompletableFuture.supplyAsync(() -> healthEndpointOptional.get().health());
        try {
            HealthComponent healthComponent = future.get(8, TimeUnit.SECONDS);
            Status status = healthComponent.getStatus();
            // 判断状态是否为UP
            isHealthy = status == Status.UP;
            // 打印状态详情日志，方便排查问题
            Map<String, HealthComponent> healthDetail = ((SystemHealth) healthComponent).getComponents();
            log.debug("系统健康状态详细信息: {}", healthDetail);
        } catch (TimeoutException e) {
            // 超时的话需要取消任务
            future.cancel(true);
            log.error("异步获取健康检查状态超时, 返回down");
        } catch (InterruptedException | ExecutionException e) {
            log.error("异步获取健康检查状态异常", e);
        }
    } else {
        log.debug("HealthEndpoint组件没有被注入, 默认返回false");
    }
    return isHealthy;
}
```

**5.2 自定义健康指示器**

你可以通过实现 `HealthIndicator` 接口来自定义健康检查。例如，检查某个外部服务的可用性：

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyServiceHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean serviceUp = checkExternalService();
        if (serviceUp) {
            return Health.up().withDetail("service", "Available").build();
        } else {
            return Health.down().withDetail("service", "Unavailable").build();
        }
    }

    private boolean checkExternalService() {
        // 自定义检查外部服务的逻辑
        return true;  // 假设服务可用
    }
}
```

#### 6. 指标收集与监控

Actuator 通过 `metrics` 端点提供了多种系统和应用的监控指标。例如，以下是一些常见的指标：

- **JVM 指标**：`jvm.memory.used`, `jvm.gc.pause`
- **系统指标**：`system.cpu.usage`, `process.uptime`
- **HTTP 请求**：`http.server.requests`

你可以通过访问 `/actuator/metrics` 来查看可用的指标，然后通过访问 `/actuator/metrics/{metricName}` 查看具体的指标数据。

例如：

- `http://localhost:8080/actuator/metrics/jvm.memory.used` 显示当前 JVM 使用的内存。
- `http://localhost:8080/actuator/metrics/system.cpu.usage` 显示系统的 CPU 使用情况。

#### 7. 安全与访问控制

由于 Actuator 端点可能暴露敏感的应用信息，因此在生产环境中建议保护这些端点。Spring Boot Actuator 允许你配置不同的安全策略来控制端点的访问。

**7.1 启用基本身份验证**

你可以在 Spring Security 中启用基本身份验证来保护 Actuator 端点：

**application.properties**：

```properties
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=always
management.endpoints.web.base-path=/manage
```

使用 Spring Security 的默认安全配置，你可以配置用户和密码：

```properties
spring.security.user.name=admin
spring.security.user.password=admin123
```

#### 8. 总结

Spring Boot Actuator 提供了一套强大且易于扩展的管理和监控工具。通过简单的配置，开发者可以轻松获取应用的健康状态、监控指标、日志信息等。Actuator 支持自定义扩展、灵活的安全配置，并且通过 Micrometer 集成，能够输出指标到多种监控系统。



## 工具方法

### **断言**

1. 断言是一个逻辑判断，用于检查不应该发生的情况

2. Assert 关键字在 JDK1.4 中引入，可通过 JVM 参数`-enableassertions`开启

3. SpringBoot 中提供了 Assert 断言工具类，通常用于数据合法性检查

```java
// 要求参数 object 必须为非空（Not Null），否则抛出异常，不予放行
// 参数 message 参数用于定制异常信息。
void notNull(Object object, String message);
// 要求参数必须空（Null），否则抛出异常，不予『放行』。
// 和 notNull() 方法断言规则相反
void isNull(Object object, String message);
// 要求参数必须为真（True），否则抛出异常，不予『放行』。
void isTrue(boolean expression, String message);
// 要求参数（List/Set）必须非空（Not Empty），否则抛出异常，不予放行
void notEmpty(Collection collection, String message);
// 要求参数（String）必须有长度（即，Not Empty），否则抛出异常，不予放行
void hasLength(String text, String message);
// 要求参数（String）必须有内容（即，Not Blank），否则抛出异常，不予放行
void hasText(String text, String message);
// 要求参数是指定类型的实例，否则抛出异常，不予放行
void isInstanceOf(Class type, Object obj, String message);
// 要求参数 `subType` 必须是参数 superType 的子类或实现类，否则抛出异常，不予放行
void isAssignable(Class superType, Class subType, String message)
```

### **对象、数组、集合**

#### ObjectUtils

1. 获取对象的基本信息

```java
// 获取对象的类名。参数为 null 时，返回字符串："null" 
String nullSafeClassName(Object obj);
// 参数为 null 时，返回 0
int nullSafeHashCode(Object object);
// 参数为 null 时，返回字符串："null"
String nullSafeToString(boolean[] array);
// 获取对象 HashCode（十六进制形式字符串）。参数为 null 时，返回 0 
String getIdentityHexString(Object obj);
// 获取对象的类名和 HashCode。参数为 null 时，返回字符串："" 
String identityToString(Object obj);
// 相当于 toString()方法，但参数为 null 时，返回字符串：""
String getDisplayString(Object obj);
```

2. 判断工具

```java
// 判断数组是否为空
boolean isEmpty(Object[] array);
// 判断参数对象是否是数组
boolean isArray(Object obj);
// 判断数组中是否包含指定元素
boolean containsElement(Object[] array, Object element);
// 相等，或同为 null时，返回 true
boolean nullSafeEquals(Object o1, Object o2);
/*判断参数对象是否为空，判断标准为：
Optional: Optional.empty()       
Array: length == 0CharSequence: length == 0  
Collection: Collection.isEmpty()         
Map: Map.isEmpty() */
boolean isEmpty(Object obj);
```

3. 其他工具方法

```java
// 向参数数组的末尾追加新元素，并返回一个新数组
<A, O extends A> A[] addObjectToArray(A[] array, O obj);
// 原生基础类型数组 --> 包装类数组
Object[] toObjectArray(Object source);
```

#### StringUtils

1. 字符串判断工具

```java
// 判断字符串是否为 null，或 ""。注意，包含空白符的字符串为非空
boolean isEmpty(Object str);
// 判断字符串是否是以指定内容结束。忽略大小写
boolean endsWithIgnoreCase(String str, String suffix);
// 判断字符串是否已指定内容开头。忽略大小写
boolean startsWithIgnoreCase(String str, String prefix);
// 是否包含空白符
boolean containsWhitespace(String str);
// 判断字符串非空且长度不为 0，即，Not Empty
boolean hasLength(CharSequence str);
// 判断字符串是否包含实际内容，即非仅包含空白符，也就是 Not Blank
boolean hasText(CharSequence str);
// 判断字符串指定索引处是否包含一个子串。
boolean substringMatch(CharSequence str, int index, CharSequence substring);
// 计算一个字符串中指定子串的出现次数
int countOccurrencesOf(String str, String sub);
```

2. 字符串操作工具

```java
// 查找并替换指定子串
String replace(String inString, String oldPattern, String newPattern);
// 去除尾部的特定字符
String trimTrailingCharacter(String str, char trailingCharacter);
// 去除头部的特定字符
String trimLeadingCharacter(String str, char leadingCharacter);
// 去除头部的空白符
String trimLeadingWhitespace(String str);
// 去除头部的空白符
String trimTrailingWhitespace(String str);
// 去除头部和尾部的空白符
String trimWhitespace(String str);
// 删除开头、结尾和中间的空白符
String trimAllWhitespace(String str);
// 删除指定子串
String delete(String inString, String pattern);
// 删除指定字符（可以是多个）
String deleteAny(String inString, String charsToDelete);
// 对数组的每一项执行 trim() 方法
String[] trimArrayElements(String[] array);
// 将 URL 字符串进行解码
String uriDecode(String source, Charset charset);
```

3. 路径相关工具方法

```java
// 解析路径字符串，优化其中的 “..” 
String cleanPath(String path);
// 解析路径字符串，解析出文件名部分
String getFilename(String path);
// 解析路径字符串，解析出文件后缀名
String getFilenameExtension(String path);
// 比较两个两个字符串，判断是否是同一个路径。会自动处理路径中的 “..” 
boolean pathEquals(String path1, String path2);
// 删除文件路径名中的后缀部分
String stripFilenameExtension(String path); 
// 以 “. 作为分隔符，获取其最后一部分
String unqualify(String qualifiedName);
// 以指定字符作为分隔符，获取其最后一部分
String unqualify(String qualifiedName, char separator);
```

#### CollectionUtils

1. 集合判断工具

```java
// 判断 List/Set 是否为空
boolean isEmpty(Collection<?> collection);
// 判断 Map 是否为空
boolean isEmpty(Map<?,?> map);
// 判断 List/Set 中是否包含某个对象
boolean containsInstance(Collection<?> collection, Object element);
// 以迭代器的方式，判断 List/Set 中是否包含某个对象
boolean contains(Iterator<?> iterator, Object element);
// 判断 List/Set 是否包含某些对象中的任意一个
boolean containsAny(Collection<?> source, Collection<?> candidates);
// 判断 List/Set 中的每个元素是否唯一。即 List/Set 中不存在重复元素
boolean hasUniqueObject(Collection<?> collection);
```

2. 集合操作工具

```java
// 将 Array 中的元素都添加到 List/Set 中<E> 
void mergeArrayIntoCollection(Object array, Collection<E> collection);
// 将 Properties 中的键值对都添加到 Map 中<K,V> 
void mergePropertiesIntoMap(Properties props, Map<K,V> map);
// 返回 List 中最后一个元素
<T> T lastElement(List<T> list);
// 返回 Set 中最后一个元素
<T> T lastElement(Set<T> set);
// 返回参数 candidates 中第一个存在于参数 source 中的元素;
<E> E findFirstMatch(Collection<?> source, Collection<E> candidates);
// 返回 List/Set 中指定类型的元素。
<T> T findValueOfType(Collection<?> collection, Class<T> type);
// 返回 List/Set 中指定类型的元素。如果第一种类型未找到，则查找第二种类型，以此类推
Object findValueOfType(Collection<?> collection, Class<?>[] types);
// 返回 List/Set 中元素的类型
Class<?> findCommonElementType(Collection<?> collection)
```



### **文件、资源、IO 流**

#### FileCopyUtils

1. 输入

```java
// 从文件中读入到字节数组中
byte[] copyToByteArray(File in);
// 从输入流中读入到字节数组中
byte[] copyToByteArray(InputStream in);
// 从输入流中读入到字符串中String copyToString(Reader in);
```

2. 输出

```java
// 从字节数组到文件
void copy(byte[] in, File out);
// 从文件到文件int copy(File in, File out);
// 从字节数组到输出流
void copy(byte[] in, OutputStream out);
// 从输入流到输出流
int copy(InputStream in, OutputStream out);
// 从输入流到输出流
int copy(Reader in, Writer out);
// 从字符串到输出流
void copy(String in, Writer out);
```

#### ResourceUtils

1. 从资源路径获取文件

```java
// 判断字符串是否是一个合法的 URL 字符串。
static boolean isUrl(String resourceLocation);
// 获取 URL
static URL getURL(String resourceLocation);
// 获取文件（在 JAR 包内无法正常使用，需要是一个独立的文件）
static File getFile(String resourceLocation);
```

2. Resource

```java
// 文件系统资源 D:\...FileSystemResource// URL 资源，如 file://... http://...UrlResource// 类路径下的资源，classpth:...ClassPathResource
// Web 容器上下文中的资源（jar 包、war 包）ServletContextResource
// 判断资源是否存在
boolean exists();
// 从资源中获得 File 对象
File getFile();
// 从资源中获得 URI 对象
URI getURI();
// 从资源中获得 URI 对象
URL getURL();
// 获得资源的 InputStream
InputStream getInputStream();
// 获得资源的描述信息
String getDescription();
```

**StreamUtils**

1. 输入

```java
void copy(byte[] in, OutputStream out);
int copy(InputStream in, OutputStream out);
void copy(String in, Charset charset, OutputStream out);
long copyRange(InputStream in, OutputStream out, long start, long end);
```

2. 输出

```java
byte[] copyToByteArray(InputStream in);
String copyToString(InputStream in, Charset charset);
// 舍弃输入流中的内容int drain(InputStream in)
```



### **反射、AOP**

#### ReflectionUtils4

1. 获取方法

```java
// 在类中查找指定方法
Method findMethod(Class<?> clazz, String name);
// 同上，额外提供方法参数类型作查找条件
Method findMethod(Class<?> clazz, String name, Class<?>... paramTypes);
// 获得类中所有方法，包括继承而来的
Method[] getAllDeclaredMethods(Class<?> leafClass);
// 在类中查找指定构造方法
Constructor<T> accessibleConstructor(Class<T> clazz, Class<?>... parameterTypes);
// 是否是 equals() 方法
boolean isEqualsMethod(Method method);
// 是否是 hashCode() 方法 
boolean isHashCodeMethod(Method method);
// 是否是 toString() 方法
boolean isToStringMethod(Method method);
// 是否是从 Object 类继承而来的方法
boolean isObjectMethod(Method method);
// 检查一个方法是否声明抛出指定异常
boolean declaresException(Method method, Class<?> exceptionType);
```

2. 执行方法

```java
// 执行方法
Object invokeMethod(Method method, Object target);
// 同上，提供方法参数
Object invokeMethod(Method method, Object target, Object... args);
// 取消 Java 权限检查。以便后续执行该私有方法
void makeAccessible(Method method);
// 取消 Java 权限检查。以便后续执行私有构造方法
void makeAccessible(Constructor<?> ctor);
```

3. 获取字段

```java
// 在类中查找指定属性
Field findField(Class<?> clazz, String name);
// 同上，多提供了属性的类型
Field findField(Class<?> clazz, String name, Class<?> type);
// 是否为一个 "public static final" 属性
boolean isPublicStaticFinal(Field field)
```

4. 设置字段

```java
// 获取 target 对象的 field 属性值
Object getField(Field field, Object target);
// 设置 target 对象的 field 属性值，值为 value
void setField(Field field, Object target, Object value);
// 同类对象属性对等赋值
void shallowCopyFieldState(Object src, Object dest);
// 取消 Java 的权限控制检查。以便后续读写该私有属性
void makeAccessible(Field field);
// 对类的每个属性执行 callback
void doWithFields(Class<?> clazz, ReflectionUtils.FieldCallback fc);
// 同上，多了个属性过滤功能。
void doWithFields(Class<?> clazz, ReflectionUtils.FieldCallback fc,                   ReflectionUtils.FieldFilter ff);
// 同上，但不包括继承而来的属性
void doWithLocalFields(Class<?> clazz, ReflectionUtils.FieldCallback fc);
```

#### **AopUtils**

1. 判断代理类型

```java
// 判断是不是 Spring 代理对象
boolean isAopProxy();
// 判断是不是 jdk 动态代理对象
bealean isJdkDynamicProxy();
// 判断是不是 CGLIB 代理对象
boolean isCglibProxy()
```

2. 获取被代理对象的 class

```java
// 获取被代理的目标
classClass<?> getTargetClass()
```

#### AopContext

1. 获取当前对象的代理对象

```java
Object currentProxy();
```



## 经验之谈

### Redis集群拓扑刷新

当使用`spring-boot-starer-redis`连接Redis集群的时候，如果一个主节点或者从节点挂了，集群的结构会变化，但是SpringBoot的Redis客户端没有获取到更新之后的Reids节点拓扑信息，还是用之前的节点信息进行数据操作，就会报错。

SpringBoot 2.3.x以上版本，可以增加以下配置，使Redis客户端能够自动刷新Redis集群拓扑结构。

```yaml
redis:
  lettuce:
    cluster:
      refresh:
        adaptive: true # 开启集群拓扑自动刷新
        period: 10s # 刷新周期，会有短时间还是使用之前的节点信息
```













