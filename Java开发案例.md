## Java开发案例

### Java实现图片质量压缩

如果不想引入其他依赖，可以使用Java原生代码实现。支持压缩png、jpg、jpeg三种格式。但是因为png格式比较特殊，这里单独写一个方法实现。思路是先缩小尺寸，再恢复到原来的尺寸。

代码如下，拿走不谢：

```java
/**
 * 压缩jpg和jpeg图片
 * @param inputStream 图片输入流
 * @param outputStream 图片输入流
 * @param compressRatio 压缩率
 * @param extensionName 拓展名
 */
public static void compressJpgAngJpegImage(InputStream inputStream, OutputStream outputStream,
                                 float compressRatio, String extensionName) throws IOException {
    // 这一行通过输入流将图片读取为 BufferedImage 对象
    BufferedImage image = ImageIO.read(inputStream);

    // 获取特定格式的图像写入器（比如JPG JPEG），并获取默认的写入参数，用于设置图像的压缩方式和质量
    ImageWriter writer = ImageIO.getImageWritersByFormatName(extensionName).next();
    ImageWriteParam param = writer.getDefaultWriteParam();

    // 设置压缩模式为 MODE_EXPLICIT，表示要显式地设置压缩参数,通过 setCompressionQuality() 方法设置压缩质量
    param.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
    param.setCompressionQuality(compressRatio);

    // 输入图片并关闭
    writer.setOutput(ImageIO.createImageOutputStream(outputStream));
    writer.write(null, new IIOImage(image, null, null), param);
    writer.dispose();
}

/**
 * 压缩png图片
 * @param inputStream 图片输入流
 * @param outputStream 图片输入流
 * @param compressRatio 压缩率
 */
public static void compressPngImage(InputStream inputStream, OutputStream outputStream, float compressRatio) throws IOException {
    BufferedImage inputImage = ImageIO.read(inputStream);

    // 获取压缩因子
    float factor = 1 / compressRatio;

    int originWidth = inputImage.getWidth();
    int originHeight = inputImage.getHeight();

    // 压缩率计算压缩尺寸
    int tempWidth = (int)(originWidth / factor);
    int tempHeight = (int)(originHeight / factor);

    // 缩小尺寸
    BufferedImage outputImage = new BufferedImage(tempWidth, tempHeight, BufferedImage.TYPE_INT_ARGB);
    outputImage.getGraphics().drawImage(inputImage.getScaledInstance(tempWidth, tempHeight, BufferedImage.SCALE_SMOOTH), 0, 0, null);

    // 恢复原始尺寸
    BufferedImage compressedImage = new BufferedImage(originWidth, originHeight, BufferedImage.TYPE_INT_ARGB);
    compressedImage.getGraphics().drawImage(outputImage.getScaledInstance(originWidth, originHeight, BufferedImage.SCALE_SMOOTH), 0, 0, null);

    ImageIO.write(compressedImage, IMAGE_PNG, outputStream);
}
```

如果嫌以上代码太多了，可以使用`Thumbnail`库，地址是<https://github.com/coobird/thumbnailator>，可以实现图片的压缩、格式转换、加水印等简单功能。实现起来也很简单，具体代码参考官方文档。

不过可惜就是`Thumbnail`无法压缩png格式，并且可能越压缩图片文件越大。



### MultipartFile的转换

Spring 中的 MultipartFile 是用于处理上传文件的接口，通常用于处理 HTTP 请求中包含的文件上传。它是 Spring MVC 框架提供的一种文件上传解决方案，可以方便地处理客户端上传的文件数据。

MultipartFile 接口提供了一些常用的方法来操作上传的文件，包括获取文件名、获取文件类型、获取文件大小、获取文件输入流等。在 Spring MVC 的控制器中，可以通过方法参数直接接收 MultipartFile 对象，Spring 框架会自动将客户端上传的文件数据封装成 MultipartFile 对象，从而方便地进行文件上传的处理。

MultipartFile 接口源码如下：

```java
public interface MultipartFile extends InputStreamSource {
    // 获取文件参数的名称
    String getName();

    // 获取上传文件的原始文件名
    @Nullable
    String getOriginalFilename();

    // 获取上传文件的类型
    @Nullable
    String getContentType();
	// 判断文件是否为空
    boolean isEmpty();
	// 获取上传文件的大小，以字节为单位
    long getSize();
	// 将文件内容读取为字节数组获取上传文件的输入流，可以用于读取文件内容
    byte[] getBytes() throws IOException;
	// 获取上传文件的输入流，可以用于读取文件内容
    InputStream getInputStream() throws IOException;
}
```

当我们需要对 MultipartFile  文件做一些其他的转换操作，只需要获取到它的输入流就可以了。

当然，如果一个接口它需要的参数是 MultipartFile 类型，我们可以实现自己的 MultipartFile 类。然后只需要保存 content 和 filename 两个核心信息就可以了，其他的信息都可以通过这两个数据得到。

MultipartFile  有很多实现类，对于controller中的文件上传，实现类是 `org.springframework.web.multipart.support`包下面的内部类`StandardMultipartHttpServletRequest.StandardMultipartFile`。



### 判断IP地址是否在CIDR范围中

一般在CIDR用来配置IP白名单等比较方便，这时候就需要判断一个IP是否在CIDR的IP地址范围中。工具方法如下：

```java
 /**
 * 判断ip地址是否在在CIDR网段中
 * @param ip ip地址
 * @param cidr cidr网段
 * @return 判断结果
 */
public static boolean isIpInCidr(String ip, String cidr) throws Exception {
    // 注意做判空处理

    // 先将CIDR表示法分割成网络地址和前缀长度
    String[] parts = cidr.split("/");
    InetAddress ipAddress = InetAddress.getByName(ip);
    InetAddress networkAddress = InetAddress.getByName(parts[0]);
    int prefixLength = Integer.parseInt(parts[1]);

    // 将IP地址和网络地址转换为字节数组
    byte[] ipBytes = ipAddress.getAddress();
    byte[] networkBytes = networkAddress.getAddress();

    // 将这些字节数组转换为整数，计算掩码
    int ipInt = ByteBuffer.wrap(ipBytes).getInt();
    int networkInt = ByteBuffer.wrap(networkBytes).getInt();
    int mask = -1 << (32 - prefixLength);

    // 并使用按位与操作符（&）计算掩码，比较IP地址和网络地址的掩码结果
    return (ipInt & mask) == (networkInt & mask);
}
```

原理，要判断一个IP地址是否在一个CIDR网段内，我们可以按照以下步骤进行：

1. 将IP地址和CIDR中的网络地址转换为整数形式。这可以通过将每个字节的值乘以相应的权重并相加来实现。例如，对于IP地址 `192.168.1.1`，其整数形式为 `(192 * 256^3) + (168 * 256^2) + (1 * 256^1) + 1`。
2. 计算CIDR中的子网掩码。子网掩码是一个32位的二进制数，其中前缀长度指定的位数为1，其余位为0。例如，对于 `/24`，子网掩码为 `11111111.11111111.11111111.00000000`，即 `255.255.255.0`。将其转换为整数形式，得到 `(255 * 256^3) + (255 * 256^2) + (255 * 256^1) + 0`。对于代码中`-1 << (32 - prefixLength)`，通过-1直接左移若干位数，就能得到掩码的二进制了。
3. 对IP地址的整数形式应用子网掩码。这可以通过按位与操作符（&）实现。例如，如果IP地址的整数形式为 `ipInt`，子网掩码的整数形式为 `mask`，则结果为 `ipInt & mask`。
4. 比较结果。如果IP地址的整数形式与CIDR的网络地址的整数形式经过子网掩码处理后相等，那么该IP地址就位于CIDR网段内。



### 查找匹配正则表达式的内容

示例：查找字符串 `${name} is good, he is ${age} years old` 中的被`${}`包裹起来的变量的名。

```java
String expression = "${name} is good, he is ${age} years old";
// 正则表达式 \\$\\{：匹配 ${。([^}]+)：捕获 ${} 之间的内容，即变量名(使用一堆括号定义一个捕获组)。\\}：匹配 }。
// 捕获组是通过括号 () 将匹配到的部分内容保存起来，以便在后续操作中可以引用或处理这些内容。捕获组可以用来提取、引用、或操作字符串中的某些部分，非常有用。
Pattern pattern = Pattern.compile("\\$\\{([^}]+)\\}");
Matcher matcher = pattern.matcher(expression);
// 不断往后匹配
while (matcher.find()) {
    // 因为只有一个捕获组，所以只需要获取第一个捕获组(下标从1开始)的内容即可
    String g = matcher.group(1);
    System.out.println(g);
}
```



### Java 4种文件读取方式速度对比

通过字节流、缓冲字节流、随机文件访问、文件内存映射 计算 Git 安装包的 crc32校验和

```java
/**
 * 通过字节流、缓冲字节流、随机文件访问、文件内存映射 计算文件 crc32校验和
 * author: sunny
 * date: 2022/7/12 15:53
 */
public class CalReadFileTime {

    private static String filePath = "D:\\下载\\chrome\\Git-2.37.0-64-bit.exe";
    public static void main(String[] args) {
        inputStream();
        bufferedInputStream();
        randomAccessFile();
        fileMemoryMap();
    }

    /**
     * 普通字节流，一个字节一个字节从硬盘读取数据
     */
    private static void inputStream() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (InputStream inputStream = new FileInputStream(filePath)){
            int abyteValue;
            while ((abyteValue = inputStream.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("inputStream:" + crc32.getValue() + " duration:" + duration.toMillis());

    }

    /**
     * 带缓冲的字节流，没有数据时会从硬盘读取一块数据到缓冲区中
     */
    private static void bufferedInputStream() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream(filePath))){
            int abyteValue;
            while ((abyteValue = bufferedInputStream.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("bufferedInputStream:" + crc32.getValue() + "duration:" + duration.toMillis());

    }

    /**
     * 随机文件访问，也是一个字节一个字节读。因为提供了访问指针，所以理论上随度比 inputStream 慢
     */
    private static void randomAccessFile() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (RandomAccessFile randomAccessFile = new RandomAccessFile(filePath, "r")){
            int abyteValue;
            while ((abyteValue = randomAccessFile.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("randomAccessFile:" + crc32.getValue() + "duration:" + duration.toMillis());
    }

    /**
     * 文件内存映射，通过虚拟内存将整个或者部分文件内容加载到内存中
     */
    private static void fileMemoryMap() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (FileChannel fileChannel = FileChannel.open(Paths.get(filePath))){
            ByteBuffer byteBuffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
            while (byteBuffer.hasRemaining()) {
                // byteBuffer.get(byte[]) 方法可以获取到所有的字节数据
                crc32.update(byteBuffer.get());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("fileMemoryMap:" + crc32.getValue() + "duration:" + duration.toMillis());
    }
}
```

**结果对比**

```verilog
inputStream:3199121536 duration:43084
bufferedInputStream:3199121536 duration:148
randomAccessFile:3199121536 duration:38037
fileMemoryMap:3199121536 duration:147
```

发现带缓存的输入流和文件内存映射这两种方式是最快的。



### 解决跨域问题的两种办法

在前后端分离的项目中，跨域问题的主要解决办法有两种。

1. 使用代理，如Vue项目开发过程中前端通过nodejs实现代理，或者在生产环境中，使用nginx做为代理。这样前端地址和访问接口地址一样，就不存在跨域问题了。

2. SpringBoot实现CORS（跨域资源共享）

   只要后端响应头 "Access-Control-Allow-Origin" 设置为 "*"，浏览器将不会有跨域警告。原理代码如下所示：

   ```java
   @GetMapping("/getName")
   public String getName(HttpServletResponse response, String name) throws Exception{
       // 设置响应头
       response.setHeader("Access-Control-Allow-Origin", "*");
       return name + "哈哈哈";
   }
   ```

   临时设置单个接口或者单个Controller中的接口，可以使用`@CrossOrigin`注解实现：

   ```java
   @CrossOrigin(origins = "*", maxAge = 3600)
   @RestController
   @RequestMapping("/api/user")
   public class UserController {
       @GetMapping("/info")
       public String getUserInfo() {
           return "hello";
       }
   }
   ```

   全局设置可以使用如下`WebMvcConfigurer`配置类进行配置：

   ```java
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.servlet.config.annotation.CorsRegistry;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
   
   @Configuration
   public class GlobalCorsConfig {
       @Bean
       public WebMvcConfigurer corsConfigurer() {
           return new WebMvcConfigurer() {
               @Override
               public void addCorsMappings(CorsRegistry registry) {
                   registry.addMapping("/**") // 匹配所有接口
                           .allowedOriginPatterns("*") // 允许所有源（你也可以指定前端地址）
                           .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                           .allowedHeaders("*")
                           .allowCredentials(true) // 允许携带cookie
                           .maxAge(3600); // OPTIONS 预检请求缓存1小时
               }
           };
       }
   }
   
   ```

   

