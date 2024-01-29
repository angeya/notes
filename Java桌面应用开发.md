## 常见场景

### 模拟键盘输入



### 获取和设置剪切板



### 监听全局按键

要监听全局快捷键（即无论焦点在哪个窗口都能触发的快捷键），需要使用`GlobalShortcut`库或者`JNativeHook`库来实现。这两个库都可以用于在Java应用程序中监听全局快捷键。

JNativeHook 为 Java 程序提供全局的键盘和鼠标事件侦听功能。你可以来处理程序外的键盘输入和鼠标动作。当然 JNativeHook 使用了 JNI 技术调用了系统的方法来实现该功能。

以下是使用`JNativeHook`库来监听全局快捷键的基本步骤：

1. 引入maven依赖

   ```java
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

   