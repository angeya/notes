### SpringCloudBus

SpringCloudBus是微服务总线，可以实现微服务之间的通信。基于SpringBoot的事件发送和监听机制，可以基于RabbitMq或者kafka实现。
如果使用RabbitMQ，连接上RabbitMQ之后，可以在Exchange中看到这一行：springCloudBus topic 
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

