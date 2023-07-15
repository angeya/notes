# SpringBoot

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

   

   

   

## 扩展

### SpringBoot静态路径

SpringBoot默认的静态路径是`resources/static`。在此路径下放入媒体文件是可以直接访问的。



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

购买证书：可以在腾讯云等云平台上购买，然后下载，一般包含证书文件和密码文件

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

   出参可以在类的`writeInternal`方法打断点查看，该方法中的`Object object`参数即是返回的对象。序列化时将object转化为json字符串，然后将字符串写入到字节流`ByteArrayOutputStream`中，最后通过Http输出流输出。

2. 使用默认的序列化框架



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

   



