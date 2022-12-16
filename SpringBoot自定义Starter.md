# SpringBoot自定义Starter

##  1 编写自己的Starter

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
   
       // 装配业务逻辑Bean，这一步可以没有（这就需要在使用的项目中注入），也可以直接在类中使用Component等注解
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

   

## 2 使用Starter



