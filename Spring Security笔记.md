## 基本使用

### 初使用

1. 引入maven依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 这个时候我们不在配置文件中做任何配置，随便写一个Controller 

   ```java
   @RequestMapping
   @RestController
   public class IndexController {
   
       @GetMapping("/hi")
       public String hi(HttpServletRequest request) {
           System.out.println(request);
           return "hi";
       }
   }
   ```

3. 做完以上步骤，项目中已经包含Spring Security了，会默认对系统进行保护。这时候启动项目会有类似如下的日志：

   ```te
   2024-01-25 22:33:10.014  INFO 22956 --- [ main] .s.s.UserDetailsServiceAutoConfiguration : 
   
   Using generated security password: 4d31c0fe-0f31-4677-84cb-ea0fc28a56d7
   ```

   显示user用户的密码。

4. 这时候使用浏览器访问 `http://127.0.0.1:8080/hi` 会弹出一个登录页面。在页面中输入user和上面显示的密码。登录成功之后就能返回访问接口的结果了。

### 配置内存用户名密码