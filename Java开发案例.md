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



