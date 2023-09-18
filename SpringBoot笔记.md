# SpringBoot

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


## Spring MVC

### controller最佳实践

**一、统一状态码**

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

**统一校验**

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

**三、统一响应**

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



## 扩展

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

   入参可以在`FastJsonHttpMessageConverter`类的`readType`方法中打断点查看。反序列化过程会读取Http的输入流内容，然后转化为字节数组，再使用字节数组通过`JSON.parseObject()`方法进行解析成控制方法对应的类型对象。

   可以使用如下代码查看请求参数：

   ```java
   new String(bytes);
   ```

   出参可以在`FastJsonHttpMessageConverter`类的`writeInternal`方法打断点查看，该方法中的`Object object`参数即是返回的对象。序列化时将object转化为json字符串，然后将字符串写入到字节流`ByteArrayOutputStream`中，最后通过Http输出流输出。

2. 使用默认的序列化框架

   入参可以在`AbstractJackson2HttpMessageConverter`的`readJavaType`方法中打断点拦截。拦截到`inputMesage`对象，只需要获取其内容输入流，然后执行一下代码即可：

   ````java
   InputStream inputStream = inputMessage.getBody();
   byte[] bytes = new byte[inputStream.available()];
   // 将会消费输入流
   inputStream.read(bytes);
   new String(bytes);
   ````

   出参在`AbstractJackson2HttpMessageConverter`类的`writeInternal`方法查看，该方法中的`Object object`参数即是返回的对象。

### 使用Fastjson作为序列化框架

1. 引入fastjson依赖

在项目的pom.xml文件中引入fastjson依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.72</version>
</dependency>
```

2. 自定义FastJsonHttpMessageConverter

在Spring Boot中默认使用Jackson作为JSON序列化和反序列化框架，需要自己定义一个FastJsonHttpMessageConverter来替换掉Jackson，使得Spring Boot使用fastjson进行处理。代码如下：

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
spring:
  http:
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

