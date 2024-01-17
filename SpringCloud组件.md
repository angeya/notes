## OpenFeign

### 概述

Feign是一个声明式的Web服务客户端（Web服务客户端就是Http客户端），让编写Web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。

Java当中常见的Http客户端有很多，除了Feign，类似的还有Apache 的 HttpClient 以及OKHttp3，还有SpringBoot自带的RestTemplate这些都是Java当中常用的HTTP 请求工具。

### OpenFeign和Feign的区别

- Feign

  Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端，Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。

  Feign是在2019就已经不再更新了，通过maven网站可以看出来，随之取代的是OpenFeign，从名字上就可以知道，他是Feign的升级版。

- OpenFeign

  OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的`@FeignClient`可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

### @FeignClient 注解

使用OpenFeign就一定会用到这个注解，@FeignClient属性如下：

- **name/value：** 指定Feign客户端的名称。用于标识客户端，并作为Spring容器中bean名称的一部分
- **serviceId: ** ~~已废弃~~。以前用于在使用Eureka等服务发现机制时指定客户端的服务ID。已被`name()`取代
- **contextId:** 指定Feign客户端的上下文ID。可用于区分同一个Feign客户端的多个实例
- **url:** url一般用于调试，可以手动指定@FeignClient调用的地址
- **decode404:** 指定是否解码HTTP 404响应。如果设置为`true`，Feign将不会为404响应抛出异常，而是通过Encoder解码返回一个正常的响应对象，否则抛出FeignException
- **configuration:** Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract
- **fallback:** 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口
- **fallbackFactory:** 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码
- **path:** 定义当前FeignClient的统一前缀，当我们项目中配置了server.context-path,server.servlet-path时使用

### 常规远程调用

1. 添加maven依赖（可以直接在SpringBoot项目中使用）

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
       <version>2.2.2.RELEASE</version>
   </dependency>
   ```

2. 启动类添加`@EnableFeignClients`注解

   ```java
   @EnableFeignClients
   @SpringBootApplication
   public class OpenFeignApp {
       public static void main(String[] args) {
           SpringApplication.run(OpenFeignApp.class, args);
       }
   }
   ```

3. 编写远程调用接口

   ```java
   @FeignClient(name = "remoteService", url = "http://localhost:8088/")
   public interface IRemoteService {
       
       // 这里相当于请求 http://localhost:8088/demo
       @GetMapping("/demo")
       String demo();
   }
   ```

4. 远程调用

   ```java
   @SpringBootTest
   class IRemoteServiceTest {
   
       // 注入OpenFeign接口
       @Autowired
       private IRemoteService remoteService;
   
       @Test
       void demo() {
           // 调用接口方法，通过生成动态代理，调用对应路径的远程接口
           String res = remoteService.demo();
           System.out.println(res);
       }
   }
   ```

   

5. xxx



## SpringCloudBus

### 概述

SpringCloudBus是微服务总线，可以实现微服务之间的通信，如消息广播。基于SpringBoot的事件发送和监听机制，可以基于RabbitMq或者kafka实现。
如果使用RabbitMQ，连接上RabbitMQ之后，可以在Exchange中看到这一行：springCloudBus topic 。也可以不需要消息队列工作。

可以通过广播消息用来解决多个微服务使用Websocket推送消息的问题（问题：某个需要发送消息的服务可能没有被前端Websocket注册）。

### 使用

1，引入maven依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

2，监听远程事件
在启动类上增加注解 `@RemoteApplicationEventScan` 开启事件监听，否则无法监听远程事件，只能监听本地事件

3，定义事件

事件需要继承自`RemoteApplicationEvent`，RemoteApplicationEvent是ApplicationEvent的子类。

```java
@Data
public class MyEvent extends RemoteApplicationEvent {
    // 要保留空构造函数，否则无法反序列化
    public MyEvent() {}

    // 需要指定source（事件源，不为空的Object即可）和originService（源微服务）才能发送远程事件（广播）
    // 也可以指定目标微服务destinationService
    public MyEvent(Object source, String originService, String data) {
        super(source, originService);
        this.data = data;
    }

    // 这里是事件的负载
    private String data;
}
```
4，发送事件

```java
/**
 * 当前微服务
 */
@AutoWired
private BusProperties busProperties

public void send() {
	MyEvent event = new MyEvent(this, busProperties.getId(), "sunny");
    eventPublisher.publishEvent(event);
}
```

5，监听事件

```java
@Component
public class MyListener {
    @EventListener
    public void onMyCustomEvent(MyEvent event) {
        // ...
        System.out.println("收到事件" + event);
    }
}
```

