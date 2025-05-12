## 常见场景

### 模拟键盘输入



### 获取和设置剪切板



### 监听全局按键

要监听全局快捷键（即无论焦点在哪个窗口都能触发的快捷键），需要使用`GlobalShortcut`库或者`JNativeHook`库来实现。这两个库都可以用于在Java应用程序中监听全局快捷键。

JNativeHook 为 Java 程序提供全局的键盘和鼠标事件侦听功能。你可以来处理程序外的键盘输入和鼠标动作。当然 JNativeHook 使用了 JNI 技术调用了系统的方法来实现该功能。

以下是使用`JNativeHook`库来监听全局快捷键的基本步骤：

1. 引入maven依赖

   ```xml
   <dependency>
       <groupId>com.1stleg</groupId>
       <artifactId>jnativehook</artifactId>
       <version>2.1.0</version>
   </dependency>
   ```

2. 应用

   ```java
   import org.jnativehook.GlobalScreen;
   import org.jnativehook.keyboard.NativeKeyEvent;
   import org.jnativehook.keyboard.NativeKeyListener;
   
   import java.util.logging.Level;
   import java.util.logging.Logger;
   
   public class ShortcutDemo  implements NativeKeyListener {
       
       // 监听按键压下
       @Override
       public void nativeKeyPressed(NativeKeyEvent e) {
           System.out.println("Key Pressed: " + NativeKeyEvent.getKeyText(e.getKeyCode()));
       }
   
       // 监听按键释放
       @Override
       public void nativeKeyReleased(NativeKeyEvent e) {
           System.out.println("Key Released: " + NativeKeyEvent.getKeyText(e.getKeyCode()));
       }
       // 监听键盘输入
       @Override
       public void nativeKeyTyped(NativeKeyEvent e) {
           System.out.println("Key Typed: " + NativeKeyEvent.getKeyText(e.getKeyCode()));
       }
   
       public static void main(String[] args) {
           try {
               GlobalScreen.registerNativeHook();
           } catch (Exception e) {
               System.err.println("Failed to register native hook");
               System.err.println(e.getMessage());
               System.exit(1);
           }
   
           // 禁用框架默认的jdk日志，否则会一直在刷
           Logger logger = Logger.getLogger(GlobalScreen.class.getPackage().getName());
           logger.setLevel(Level.OFF);
   
           ShortcutDemo demo = new ShortcutDemo();
           GlobalScreen.addNativeKeyListener(demo);
       }
   }
   ```




JavaFx开发与打包运行

这里以java21 + maven为例，开发一个Hello world应用。

1. maven依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>top.angeya</groupId>
           <artifactId>guis</artifactId>
           <version>1.0-SNAPSHOT</version>
       </parent>
   
       <artifactId>demo</artifactId>
   
       <properties>
           <maven.compiler.source>21</maven.compiler.source>
           <maven.compiler.target>21</maven.compiler.target>
           <javafx.version>21</javafx.version>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <!--不一定需求fxml，看项目需求-->
           <dependency>
               <groupId>org.openjfx</groupId>
               <artifactId>javafx-fxml</artifactId>
               <version>21</version>
           </dependency>
           <!--控件必备，依赖javafx-base-->
           <dependency>
               <groupId>org.openjfx</groupId>
               <artifactId>javafx-controls</artifactId>
               <version>21</version>
           </dependency>
           <!--其他依赖-->
       </dependencies>
   
       <build>
           <plugins>
               <!--Maven 编译器插件，设置java版本 -->
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-compiler-plugin</artifactId>
                   <version>3.13.0</version>
                   <configuration>
                       <source>21</source>
                       <target>21</target>
                   </configuration>
               </plugin>
   
               <!--如果没有其他maven依赖可以使用这个，不是fat包-->
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-jar-plugin</artifactId>
                   <version>3.3.0</version>
                   <configuration>
                       <archive>
                           <manifest>
                               <!-- 指定主类，替换为你的主类完整路径 -->
                               <mainClass>top.angeya.license.LicenseApp</mainClass>
                           </manifest>
                       </archive>
                   </configuration>
               </plugin>
   
               <!-- Maven Shade 插件以创建可执行的 fat JAR,这里是用来打包其他依赖的，javafx运行时组件需要再运行时指定 -->
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-shade-plugin</artifactId>
                   <version>3.5.0</version>
                   <executions>
                       <execution>
                           <phase>package</phase>
                           <goals>
                               <goal>shade</goal>
                           </goals>
                           <configuration>
                               <transformers>
                                   <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                       <mainClass>top.angeya.demo.DemoApp</mainClass>
                                   </transformer>
                               </transformers>
                           </configuration>
                       </execution>
                   </executions>
               </plugin>
           </plugins>
       </build>
   
   </project>
   ```

   

2. module-info.java

   ```java
   module top.angeya.demo {
       requires javafx.controls;
       exports top.angeya.demo to javafx.graphics;
   }
   ```

   

3. Hello World程序

   ```java
   package top.angeya.license;
   
   import com.alibaba.fastjson.JSONObject;
   import javafx.application.Application;
   import javafx.scene.Scene;
   import javafx.scene.control.Button;
   import javafx.scene.control.TextArea;
   import javafx.scene.input.MouseEvent;
   import javafx.scene.layout.Priority;
   import javafx.scene.layout.VBox;
   import javafx.stage.Stage;
   import org.apache.commons.lang3.StringUtils;
   import org.apache.shiro.codec.Hex;
   import org.apache.shiro.crypto.AesCipherService;
   
   import java.nio.charset.StandardCharsets;
   
   
   public class LicenseApp extends Application {
   
       @Override
       public void start(Stage stage) {
           Button button = new Button("Hello World");
           VBox vBox = new VBox(button);
           Scene scene = new Scene(vBox, 800, 600);
           stage.setScene(scene);
   
           stage.setTitle("Demo");
           stage.show();
       }
   
       public static void main(String[] args) {
           LicenseApp.launch(args);
       }
   }
   
   ```

   

4. 打包运行

   使用maven进行打包

   ```cmd
   mvn clean package
   ```

   下载 javafx 运行时依赖 [传送门](https://gluonhq.com/products/javafx/
)

   运行

   ```shell
   java --module-path D:\DevEnv\javafx-sdk-21.0.7\lib --add-modules javafx.controls -jar license-1.0-SNAPSHOT.jar
   # 运行后关闭命令行
   start javaw --module-path D:\DevEnv\javafx-sdk-21.0.7\lib --add-modules javafx.controls -jar license-1.0-SNAPSHOT.jar
   exit
   ```

   
