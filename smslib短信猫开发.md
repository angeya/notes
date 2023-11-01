# smslib 短信猫开发

## 连接短信猫

1. 首先安装短信猫的驱动程序：BroadMobi_Driver_Installer.exe（Linux版本的暂无）
2. 插入短信猫之后，在计算机管理->设备管理->端口中找到对应的端口，右键属性可以查看波特率
3. 使用Secure CRT或者MobaXterm等客户端，新建串口连接，填入对应的端口和波特率
4. 在终端界面，输入AT并回车，如果能够返回OK则说明短信猫连接正常

## 准备串口链接库

1. 准备串口链接库文件

   64位系统：根据系统下载mfz-rxtx-2.2-20081207-win-x64.zip或者mfz-rxtx-2.2-20081207-linux-x86_64.zip，地址：http://fizzed.com/oss/rxtx-for-java

   32位系统：下载rxtx-2.1-7-bins-r2.zip（内含各个系统的文件），地址：http://rxtx.qbang.org/pub/rxtx/rxtx-2.1-7-bins-r2.zip

2. 将文件放入正确位置

   将 rxtxSerial.dll 和 rxtxParallel.dll 文件放入 JAVA_HOME/jre/bin/ 目录中

   将 RXTXcomm.jar 放入  JAVA_HOME/jre/lib/ext 目录中，也可以直接在项目中引入（即加入到项目的classpath中）

## 应用开发

1. 项目依赖

   ```xml
   <dependency>
       <groupId>org.smslib</groupId>
       <artifactId>smslib</artifactId>
       <version>3.5.5</version>
   </dependency>
   
   <dependency>
       <groupId>org.jsmpp</groupId>
       <artifactId>jsmpp</artifactId>
       <version>2.3.0</version>
   </dependency>
   
   <dependency>
       <groupId>commons-net</groupId>
       <artifactId>commons-net</artifactId>
       <version>3.0.1</version>
   </dependency>
   
   <dependency>
       <groupId>org.apache.logging.log4j</groupId>
       <artifactId>log4j-1.2-api</artifactId>
       <version>2.1</version>
   </dependency>
   
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-api</artifactId>
       <version>1.6.3</version>
   </dependency>
   
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-log4j12</artifactId>
       <version>1.6.4</version>
   </dependency>
   ```

2. 日志配置，需要使用xml方式，简单模板如下

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Configuration status="WARN">
       <Appenders>
           <Console name="Console" target="SYSTEM_OUT">
               <PatternLayout pattern="%d{YYYY-MM-dd HH:mm:ss} [%t] %-5p %c{1}:%L - %msg%n" />
           </Console>
   
           <RollingFile name="RollingFile" filename="log/test.log"
                        filepattern="${logPath}/%d{YYYYMMddHHmmss}-fargo.log">
               <PatternLayout pattern="%d{YYYY-MM-dd HH:mm:ss} [%t] %-5p %c{1}:%L - %msg%n" />
               <Policies>
                   <SizeBasedTriggeringPolicy size="10 MB" />
               </Policies>
               <DefaultRolloverStrategy max="20" />
           </RollingFile>
   
       </Appenders>
       <Loggers>
           <Root level="info">
               <AppenderRef ref="Console" />
               <AppenderRef ref="RollingFile" />
           </Root>
       </Loggers>
   </Configuration>
   ```

3. 收发短信（源码包 smslib-v3-master.zip 中有文档和示例）

   ```java
   package com.sinovatio.smscat;
   import java.util.ArrayList;
   import java.util.List;
   import java.util.Properties;
   
   import org.smslib.InboundMessage;
   import org.smslib.Message.MessageEncodings;
   import org.smslib.OutboundMessage;
   import org.smslib.Service;
   import org.smslib.modem.SerialModemGateway;
   
   public class SmsService {
       /**
        * 私有静态实例
        */
       private static SmsService instance = null;
       /**
        * 是否开启服务
        */
       private boolean isStartService = false;
       /**
        * 私有构造方法
        */
       private SmsService() {
       }
       /**
        * 获取实例（单例模式）
        * @return
        */
       public static SmsService getInstance() {
           if (instance == null) {
               instance = new SmsService();
           }
           return instance;
       }
       /**
        * 开启短信服务
        */
       public void startService() {
           System.out.println("开始初始化SMS服务！");
           // 初始化网关，参数信息依次为：COMID,COM号,比特率,制造商,Modem模式(端口号，波特率写对就可以)
           // 可以设置为从配置文件中读取，也可以优化为自动检测端口和波特率
           SerialModemGateway gateway = new SerialModemGateway("modem.com10", "COM10", 9600, "wavecom", "");
           gateway.setInbound(true);
           gateway.setOutbound(true);
           gateway.setSimPin("0000"); // 一般为0000或者1234
           OutboundNotification outboundNotification = new OutboundNotification();
           InboundNotification inboundNotification = new InboundNotification();
           Service service = Service.getInstance();
           if (service == null) {
               System.out.println("初始化SMS服务失败！");
               return;
           }
           service.setOutboundMessageNotification(outboundNotification);
           service.setInboundMessageNotification(inboundNotification);
           try {
               service.addGateway(gateway);
               // 开启服务
               service.startService();
               System.out.println("初始化SMS服务成功！");
               isStartService = true;
           } catch (Exception e) {
               System.out.println("开启SMS服务异常:" + e.getMessage());
           }
       }
   
       /**
        * 停止SMS服务
        */
       public void stopService() {
           try {
               Service.getInstance().stopService();
           } catch (Exception e) {
               System.out.println("关闭SMS服务异常:" + e.getMessage());
           }
           isStartService = false;
       }
   
       /**
        * 发送短信
        * @param toNumber 手机号码
        * @param message  短信内容
        */
       public void sendMessage(String toNumber, String message) {
           if (!isStartService) {
               System.out.println("尚未开启SMS服务！");
               return;
           }
           // 封装信息
           OutboundMessage msg = new OutboundMessage(toNumber, message);
           msg.setEncoding(MessageEncodings.ENCUCS2);
           try {
               // 发送信息
               Service.getInstance().sendMessage(msg);
           } catch (Exception e) {
               System.out.println("SMS服务发送信息发生异常:" + e.getMessage());
               isStartService = false;
           }
       }
   
       /**
        * 读取新信息
        * 这里应该是异步的，而且增加一个回调
        */
       public void readNewMessage(){
           while(!Thread.interrupted()) {
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               List<InboundMessage> msgList = this.readMessage(InboundMessage.MessageClasses.UNREAD);
               if (msgList.size() > 0) {
                   System.out.println("接收到新消息");
                   this.printMsgs(msgList);
               }
           }
       }
   
       /**
        * 读取已读的信息
        */
       public void readOldMessage(){
           List<InboundMessage> msgList = this.readMessage(InboundMessage.MessageClasses.READ);
       }
   
       /**
        * 读取所有信息
        */
       public void readAllMessage(){
           List<InboundMessage> msgList = this.readMessage(InboundMessage.MessageClasses.ALL);
       }
   
       /**
        * 读信息
        * @param msgType 读取信息类型
        * @return 信息列表
        */
       private List<InboundMessage> readMessage(InboundMessage.MessageClasses msgType) {
           if (!isStartService) {
               System.out.println("尚未开启SMS服务！");
               return new ArrayList<>();
           }
           // 封装信息
           List<InboundMessage> msgList = new ArrayList<>();
           try {
               Service.getInstance().readMessages(msgList, msgType);
           } catch (Exception e) {
               System.out.println("SMS服务读取信息发生异常:" + e.getMessage());
               isStartService = false;
           }
           return msgList;
       }
   
   
   
       /**
        * 打印信息
        * @param msgList 信息雷暴
        */
       private void printMsgs(List<InboundMessage> msgList){
           int num = 1;
           for (InboundMessage msg : msgList)
           {
               System.out.println("第" + num + "条；发件人："+msg.getOriginator() + ";内容：" + msg.getText() + "\n");
               num++;
           }
       }
   }
   
   ```

   客户端：调用服务

   ```java
   public class Client {
       public static void main(String[] args) throws Exception{
           SmsService smsService = SmsService.getInstance();
           smsService.startService();
   //        smsService.sendMessage("10086", "测试 Test4！");
           //读消息需要异步和增加一个回调
           smsService.readNewMessage();
           smsService.readOldMessage();
           smsService.readAllMessage();
           // 停止服务
   //        smsService.stopService();
       }
   }
   ```