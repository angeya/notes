# SpringBoot笔记

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

   



