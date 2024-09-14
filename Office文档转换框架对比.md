## 框架对比

### Ice-blue Spire (8.3.6)

收费。有内存问题。

| 文件名                     | 文件大小 | 页数 | 耗时ms | 还原度 | 其他                                            |
| -------------------------- | -------- | ---- | ------ | ------ | ----------------------------------------------- |
| 简单demo.docx              | 26k      | 4    | 8.4s   | 99%    |                                                 |
| 电冶表格格式乱.doc         | 93k      | 2    | 0.7s   | 99%    | 本地近乎完美。linux下格式会乱，个别字几乎被遮挡 |
| 习酒业务说明.docx          | 19M      | 107  | 144.7s | 95%    | 现场会oom，本地没有oom，但是出现idea黑屏        |
| Syncplant工程实施说明.docx | 22M      | 17   | 45.8s  | 95%    |                                                 |
| 平台用户手册.docx          | 82M      | 265  | 无穷大 | 0      | 图片多，处理两三分钟后，oom                     |



### Asponse

收费。收费软件中算最好了。

价格 （永久）：
**开发人员：**1679 美元：1 个开发人员和 1 个部署位置。5037 美元：1 个开发人员和无限的部署位置。33580 美元：1 个开发人员和 50 个商业部署

**网站：**8395 美元：最多 10 个开发人员和 10 个部署位置。23506 美元：最多 10 名开发人员和无限的部署位置。83950 美元：最多 10 名开发人员和 250 个商业部署

**计量：**1999 美元/月

| 文件名                     | 文件大小 | 页数 | 耗时ms | 还原度 | 其他               |
| -------------------------- | -------- | ---- | ------ | ------ | ------------------ |
| 简单demo.docx              | 26k      | 4    | 2.1s   | 99%    |                    |
| 电冶表格格式乱.doc         | 93k      | 2    | 8.9s   | 95%    | 出现不该换行的地方 |
| 习酒现场文件.docx          | 19M      | 107  | 21.1s  | 95%    |                    |
| Syncplant工程实施说明.docx | 22M      | 17   | 5.4s   | 90%    | 页面会变成22页     |
| 平台用户手册.docx          | 82M      | 265  | 43.4s  | 90%    | 页眉位置向左偏移   |



### Doc4j

开源。免费开箱即用的框架中算是很出色了。

| 文件名                     | 文件大小 | 页数 | 耗时ms | 还原度 | 其他                               |
| -------------------------- | -------- | ---- | ------ | ------ | ---------------------------------- |
| 简单demo.docx              | 26k      | 4    | 13.9s  | 85%    | 部分字体缺失                       |
| 电冶表格格式乱.doc         | 93k      | 2    | 无穷大 | 0      | 被认为是Excel表格，报错了          |
| 习酒现场文件.docx          | 19M      | 107  | 48.3s  | 70%    | 部分字体缺失(应该不难解决)         |
| Syncplant工程实施说明.docx | 22M      | 17   | 25.2s  | 80%    | 间隔变小 14页                      |
| 平台用户手册.docx          | 82M      | 265  | 148.5s | 70%    | 字体缺失，标题有重叠。页眉位置不对 |



### OpenOffice / LibreOffice +jodconverter

Apache旗下开源组件。(不直接支持docx，pptx等格式，但是换了后缀名不影响转换，而且还原度都不错)

| 文件名                     | 文件大小 | 页数 | 耗时ms | 还原度 | 其他                           |
| -------------------------- | -------- | ---- | ------ | ------ | ------------------------------ |
| 简单demo.docx              | 26k      | 4    | 1.8s   | 90%    | 个别字重叠                     |
| 电冶表格格式乱.doc         | 93k      | 2    | 0.7s   | 100%   | 近乎完美                       |
| 习酒现场文件.docx          | 19M      | 107  | 19.2s  | 95%    |                                |
| Syncplant工程实施说明.docx | 22M      | 17   | 3.2s   | 95%    |                                |
| 平台用户手册.docx          | 82M      | 265  | 39.2s  | 90%    | 表格中个别字间距变大，偶现换行 |



### KKFileView

安装包很大，1-2G。安装相对麻烦，在本地运行上传文件会被加密，转换乱码了。底层使用OpenOffice/LibreOffice。



### Poi

Apache旗下开源框架。

poi虽然很强大，但用起来是最痛苦的，用的组件多，各种版本冲突，各种缺少组件。而且转换效果不好。



## 对比总结

从还原度来看，Spire、Aspose、OpenOffice都是可以使用的。

**Spire(8.3.6)：**整体还原度不错，但是时间和内存开销很大。

**Aspose：**没什么问题，但是售价较高。

**OpenOffice：**没有什么明显问题，而且免费。但是不能开箱即用，需要安装OpenOffice或者LibreOffice。

综上，如果不付费的话，可以使用OpenOffice，或者和Spire一起使用。付费可以使用Aspose，Spire新版性能和还原度还未来得及评测。



## 安装与使用

### Spire

依赖

```xml
<dependency>
    <groupId>e-iceblue</groupId>
    <artifactId>spire.doc</artifactId>
    <version>10.11.6</version>
    <scope>system</scope>
    <systemPath>D:/Code/test/Spire.Doc.jar</systemPath>
</dependency>
```

示例代码

```java
import com.spire.doc.*;

public class WordToPdfSpire {
    public static void wordToPdf(String docFile,String pdfFile){
        com.spire.license.LicenseProvider.setLicenseFile("D:\\test\\license.elic.xml");
        //实例化Document类的对象
        Document doc = new Document();

        //加载Word
        doc.loadFromFile(docFile);

        //保存为PDF格式
        doc.saveToFile(pdfFile,FileFormat.PDF);
    }
}
```



### Aspose.words

依赖

```xml
<dependency>
    <groupId>com.aspose</groupId>
    <artifactId>aspose-words</artifactId>
    <version>22.11</version>
    <classifier>jdk17</classifier>
</dependency>
```

示例代码

```java
package org.example;

import com.aspose.words.Document;
import com.aspose.words.DocumentBuilder;
import com.aspose.words.ImportFormatMode;
import com.aspose.words.License;

import java.io.File;
import java.nio.file.Files;

public class WordToPdfUtil {
    private static boolean license = false;

    public static void main(String[] args) throws Exception{
        long start = System.currentTimeMillis();
        // For complete examples and data files, please go to https://github.com/aspose-words/Aspose.Words-for-Java.git.
        License license = new License();

        try {
            license.setLicense(Files.newInputStream(new File("D:\\Desktop\\开发中间文件\\预览框架\\aspose\\Aspose.TotalProductFamily.lic").toPath()));

            System.out.println("License set successfully.");
        }
        catch (Exception e) {
            // We do not ship any license with this example,
            // visit the Aspose site to obtain either a temporary or permanent license.
            System.out.println("\nThere was an error setting the license: " + e.getMessage());
        }

        Document docA = new Document();
        DocumentBuilder builder = new DocumentBuilder(docA);

        // Insert text to the document start.
        builder.moveToDocumentStart();
        builder.write("First Hello World paragraph");

        Document docB = new Document("D:\\Desktop\\temp\\文件预览\\testFiles\\dy.doc");
        // Add document B to the and of document A, preserving document B formatting.
        docA.appendDocument(docB, ImportFormatMode.KEEP_SOURCE_FORMATTING);

        docA.save("D:\\Desktop\\temp\\文件预览\\testFiles\\dy1.pdf");

        System.out.println("耗时: " + (System.currentTimeMillis() - start));
    }
}
```



### Doc4j

依赖

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.21</version>
</dependency>
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j-JAXB-Internal</artifactId>
    <version>8.2.4</version>
</dependency>
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j-export-fo</artifactId>
    <version>8.2.4</version>
</dependency>
```

示例代码

```java
import org.docx4j.Docx4J;
import org.docx4j.fonts.IdentityPlusMapper;
import org.docx4j.fonts.Mapper;
import org.docx4j.fonts.PhysicalFonts;
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.nio.file.Files;
import java.nio.file.Paths;

public class Doc4jTest {
    public static void main(String[] args) throws Exception{
        long start = System.currentTimeMillis();
        String in = "D:\\Desktop\\temp\\文件预览\\testFiles\\dy.doc";
        String out = "D:\\Desktop\\temp\\文件预览\\testFiles\\dy-3.pdf";
        wordToPdf(in, out);
        System.out.println("耗时: " + (System.currentTimeMillis() - start));
    }

    public static void wordToPdf(String docFile,String pdfFile) throws Exception {
        final Logger logger = LoggerFactory.getLogger(Docx4J.class);

        WordprocessingMLPackage pkg = Docx4J.load(new File(docFile));
        Mapper fontMapper = new IdentityPlusMapper();
        fontMapper.put("隶书", PhysicalFonts.get("LiSu"));
        fontMapper.put("宋体", PhysicalFonts.get("SimSun"));
        fontMapper.put("微软雅黑", PhysicalFonts.get("Microsoft Yahei"));
        fontMapper.put("黑体", PhysicalFonts.get("SimHei"));
        fontMapper.put("楷体", PhysicalFonts.get("KaiTi"));
        fontMapper.put("新宋体", PhysicalFonts.get("NSimSun"));
        fontMapper.put("华文行楷", PhysicalFonts.get("STXingkai"));
        fontMapper.put("华文仿宋", PhysicalFonts.get("STFangsong"));
        fontMapper.put("仿宋", PhysicalFonts.get("FangSong"));
        fontMapper.put("幼圆", PhysicalFonts.get("YouYuan"));
        fontMapper.put("华文宋体", PhysicalFonts.get("STSong"));
        fontMapper.put("华文中宋", PhysicalFonts.get("STZhongsong"));
        fontMapper.put("等线", PhysicalFonts.get("SimSun"));
        fontMapper.put("等线 Light", PhysicalFonts.get("SimSun"));
        fontMapper.put("华文琥珀", PhysicalFonts.get("STHupo"));
        fontMapper.put("华文隶书", PhysicalFonts.get("STLiti"));
        fontMapper.put("华文新魏", PhysicalFonts.get("STXinwei"));
        fontMapper.put("华文彩云", PhysicalFonts.get("STCaiyun"));
        fontMapper.put("方正姚体", PhysicalFonts.get("FZYaoti"));
        fontMapper.put("方正舒体", PhysicalFonts.get("FZShuTi"));
        fontMapper.put("华文细黑", PhysicalFonts.get("STXihei"));
        fontMapper.put("宋体扩展", PhysicalFonts.get("simsun-extB"));
        fontMapper.put("仿宋_GB2312", PhysicalFonts.get("FangSong_GB2312"));

        pkg.setFontMapper(fontMapper);
        Docx4J.toPDF(pkg, Files.newOutputStream(Paths.get(pdfFile)));
    }
}

```



### OpenOffice/LibreOffice

这里以LibreOffice为例，需要先安装LibreOffice。

**Linux(CentOS)上安装LibreOffice**

需要上传tar包并解压，进入RPMS目录，然后执行`yum install *.rpm`安装。安装完成后，执行命令获取版本`libreofficex.x --version`，其中x.x是LibreOffice的两位版本号。如果能列出版本号则说明成功。如果提示其他信息，可以排查是不是gcc版本相关的报错，适当升级gcc版本或者降低LibreOffice版本。

**LibreOffice本地命令转换文件**

```bash
# 格式：libreoffice7.6 --headless --convert-to <目标格式> <文件路径> --outdir <输出目录>
libreoffice7.6 --headless --convert-to pdf document.docx --outdir ./ # docx -> pdf
libreoffice7.6 --headless --convert-to pdf presentation.pptx --outdir ./ # pptx -> pdf
```



**启动LibreOffice服务模式**

```bash
# Linux
nohup libreoffice7.6 --headless --accept="socket,host=0.0.0.0,port=8100;urp;" --nologo --nofirststartwizard &
# Windows，找到安装目录下的Program目录，打开命令行
soffice.exe -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard
```

`--headless`: 以无界面模式启动 LibreOffice。

`--accept="socket,host=0.0.0.0,port=8100;urp;"`: 让 LibreOffice 监听 0.0.0.0 主机上的 `8100` 端口，允许远程连接。

`--nologo`: 禁止显示 LibreOffice 的启动 logo。

`--nofirststartwizard`: 跳过首次启动向导。

依赖

```xml
<dependency>
    <groupId>org.apache.directory.studio</groupId>
    <artifactId>org.apache.commons.io</artifactId>
    <version>2.4</version>
</dependency>
<dependency>
    <groupId>com.artofsolving</groupId>
    <artifactId>jodconverter</artifactId>
    <version>2.2.1</version>
</dependency>
<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>juh</artifactId>
    <version>3.1.0</version>
</dependency>

<dependency>
    <groupId>org.openoffice</groupId>
    <artifactId>unoil</artifactId>
    <version>3.0.0</version>
</dependency>
```

示例代码

```java
import com.artofsolving.jodconverter.DocumentConverter;
import com.artofsolving.jodconverter.openoffice.connection.OpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.connection.SocketOpenOfficeConnection;
import com.artofsolving.jodconverter.openoffice.converter.OpenOfficeDocumentConverter;

import java.io.File;
import java.net.ConnectException;

public class WordToPdfTestLibreOffice {
    
    public static void main(String[] args) throws Exception{
        long start = System.currentTimeMillis();
        String in = "D:\\Desktop\\temp\\文件预览\\testFiles\\test.doc";
        String out = "D:\\Desktop\\temp\\文件预览\\testFiles\\test-5.pdf";
        wordToPdf(in, out);
        System.out.println("耗时: " + (System.currentTimeMillis() - start));
    }
    
    public static void wordToPdf(String docFile,String pdfFile) throws ConnectException {
        // 源文件目录
        File inputFile = new File(docFile);
        // 输出文件目录
        File outputFile = new File(pdfFile);
        if (!outputFile.getParentFile().exists()) {
            outputFile.getParentFile().exists();
        }
        // 连接openoffice服务
        OpenOfficeConnection connection = new SocketOpenOfficeConnection(
                "127.0.0.1", 8100);
        connection.connect();
        // 转换word到pdf
        DocumentConverter converter = new OpenOfficeDocumentConverter(
                connection);
        converter.convert(inputFile, outputFile);
        // 关闭连接
        connection.disconnect();
    }
}
```



