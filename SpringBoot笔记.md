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

3. 后置最终通知@After

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

## 扩展接口

### Spring事件机制

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

## Spring MVC

### controller最佳实践

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

   

## SpringBoot3.0 版本新特性

1. JDK要求最低版本Java17
2. SpringBoot3底层默认依赖Spring6
3. 支持 Jakarta EE 10，由于 Java EE 已经变更为 Jakarta EE，包名以 javax开头的需要相应地变更为jakarta
4. Tomcat10版本
5. 支持 GraalVM 原生镜像 ，GraalVM 是 Oracle 在 2018 年发布的`一个全新的通用全栈虚拟机`，并具有高性能、跨语言交互等逆天特性，支持云原生，官网：https://www.graalvm.org/
6. 改进监控功能Micrometer和Micrometer Tracing



## 延伸

### SpringBoot静态路径

SpringBoot默认的静态路径是`resources/static`。在此路径下放入媒体文件是可以直接访问的。



### 获取当前请求的相关参数

当前请求的参数存放在`HttpServletRequest`中，当我们需要使用请求参数或者响应对象的时候，需要在controller中的方法携带参数过来。

但是当我们需要处理的业务逻辑调用栈很深的时候，就需要一直携带着`HttpServletRequest`和`HttpServletResponse`参数，这样比较麻烦。

其实SpringMvc通过ThreadLocal实现了通过当前线程获取请求参数的方法。关键类是`org.springframework.web.context.request.RequestContextHolder`，其中的方法是`getRequestAttributes()`。

xxxxxxxxxx Object currentProxy();java

```java
// 请求对象
HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();
// 响应对象
HttpServletResponse request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getResponse();
```



### SpringBoot 支持 https

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



### 拦截controller方法入参和出参

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

### 序列化框架

Jackson 和 Fastjson 都是流行的 Java JSON 库，用于在 Java 对象和 JSON 数据之间进行转换。

#### Jackson 和 Fastjson 对比

1. 性能：Fastjson 在性能方面通常比 Jackson 更快。它在处理大型数据集时表现良好，因为它具有更高的解析和序列化速度。但是，对于小型数据集，两者之间的性能差异可能并不明显。
2. 功能和灵活性：Jackson 提供了丰富的功能和灵活的配置选项。它支持 JSON-B、JSON-P 和其他标准，并且有广泛的生态系统和社区支持。Jackson 可以更好地与其他库和框架集成，例如 Spring 框架。
3. 文档和学习资源：Jackson 的文档和学习资源相对更多，社区活跃度也更高。这意味着你可以更容易地找到解决问题的答案，并获得更好的支持。
4. 安全性：Jackson 相对于 Fastjson 来说，有更好的安全记录和漏洞修复历史。Jackson 更加注重安全性，并且在过去的一些版本中修复了一些安全漏洞。

在中国使用Fastjson框架可能会更多。

#### Jackson 和 Fastjson 的使用

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



### 读取resources目录下的文件

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

### SpringBoot启动完成监听器

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





### SpringBoot自定义Starter

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

### SSE服务端推送
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

### 数据库连接池

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

### Transactional 事务支持

#### 事务失效的11种情况

##### 1，访问权限问题

如果我们自定义的事务方法（即目标方法），它的访问权限不是`public`，而是private、default或protected的话，spring则不会提供事务功能。

在`AbstractFallbackTransactionAttributeSource`类的`computeTransactionAttribute`方法中有个判断，如果目标方法不是public，则`TransactionAttribute`返回null，即不支持事务。

但是事务方法中调用的各个数据库操作的方法可以是private的，这些内部的方法使用同一个事务。

##### 2，方法用final修饰

spring事务底层使用了aop，也就是通过jdk动态代理或者cglib，帮我们生成了代理类，在代理类中实现的事务功能。

如果某个方法用final修饰了，那么在它的代理类中，就无法重写该方法，而添加事务功能。

如果某个方法是static的，同样无法通过动态代理，支持事务。

##### 3，方法内部调用

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

##### 4，未被spring管理

使用spring事务的前提是：对象要被spring管理，需要创建bean实例。

##### 5，多线程调用

虽然在事务方法内部，通过多线程来并发操作数据库，可以提高效率，但是这样就会使得事务部分失效。

spring的事务是通过数据库连接来实现的。当前线程中保存了一个map，key是数据源，value是数据库连接。我们说的同一个事务，其实是指同一个数据库连接，只有拥有同一个数据库连接才能同时提交和回滚。如果在不同的线程，拿到的数据库连接肯定是不一样的，所以是不同的事务。

##### 6，表不支持事务

周所周知，在mysql5之前，默认的数据库引擎是`myisam`。

它的好处就不用多说了：索引文件和数据文件是分开存储的，对于查多写少的单表操作，性能比innodb更好。在创建表的时候，只需要把`ENGINE`参数设置成`MyISAM`即可。

所以在实际业务场景中，myisam使用的并不多。在mysql5以后，myisam已经逐渐退出了历史的舞台，取而代之的是innodb。

> 有时候我们在开发的过程中，发现某张表的事务一直都没有生效，那不一定是spring事务的锅，最好确认一下你使用的那张表，是否支持事务。

##### 7，错误的传播特性

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

##### 8，自己吞了异常

如果想要spring事务能够正常回滚，必须抛出它能够处理的异常。如果没有抛异常，则spring认为程序是正常的。

所以，可以捕获异常，但是做了一些操作之后还需要把异常抛出来，否则事务不会回滚。

##### 9，异常类型不对

默认情况下只会回滚`RuntimeException`（运行时异常）和`Error`（错误），对于普通的Exception（非运行时异常），它不会回滚。

所以适当的加上`rollbackFor=Exception.class`配置，或者确认好回滚异常。也可以直接设置成`Throwable`

##### 10，嵌套事务回滚多了

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



##### 11，未开启事务

如果你使用的是springboot项目，那么你很幸运。因为springboot通过`DataSourceTransactionManagerAutoConfiguration`类，已经默默的帮你开启了事务。

你所要做的事情很简单，只需要配置`spring.datasource`相关参数即可。

但如果你使用的还是传统的spring项目，则需要在applicationContext.xml文件中，手动配置事务相关参数。如果忘了配置，事务肯定是不会生效的。

### 加载其他jar包中的Bean

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





















