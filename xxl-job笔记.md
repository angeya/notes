### GLUE模式 (Java)

#### 简介

原理：每个 “GLUE模式(Java)” 任务的代码，实际上是“一个继承自“IJobHandler”的实现类的类代码”，“执行器”接收到“调度中心”的调度请求时，会通过Groovy类加载器加载此代码，实例化成Java对象，同时注入此代码中声明的Spring服务（请确保Glue代码中的服务和类引用在“执行器”项目中存在），然后调用该对象的execute方法，执行任务逻辑。

#### 使用

1. 新增一条GLUE模式任务

   在`xxl-job-admin`控制面板中新增一条任务，**运行模式** 选择 `GLUE(java)`,`JobHandler`不需要填写。具体的处理逻辑在`GLUE IDE`中编写。

2. 配置代码

   保存好任务记录之后。在记录的右侧操作按钮中选择`GLUE IDE`，这时候页面会打开一个代码编辑页面，进行处理器代码的编写。这里建议代码在IDEA中写好再复制进来，因为这个`GLUE IDE`没有语法检查，很容易出错。

   ```java
   package com.xxl.job.service.handler;
   
   import com.xxl.job.core.log.XxlJobLogger;
   import com.xxl.job.core.biz.model.ReturnT;
   import com.xxl.job.core.handler.IJobHandler;
   import javax.annotation.Resource;
   import lombok.extern.slf4j.Slf4j;
   import com.angeya.xxl.service.IPfSlowSqlLogService;
   
   public class DemoGlueJobHandler extends IJobHandler {
   
       // 这里注意要import Resource和对应Bean的类
       @Resource
       private IPfSlowSqlLogService slowSqlLogService;  
     
       @Override
       public ReturnT<String> execute(String param) throws Exception {
           XxlJobLogger.log("XXL-JOB, Hello World.");
           System.out.println("666");
           XxlJobLogger.log("slow sql log list: {}", this.slowSqlLogService.getList());
           return ReturnT.SUCCESS;
       }
   }
   ```

   代码编写完成之后，点击保存。

3. 执行

   在操作按钮中点击执行一次或者启动进行执行调试。

#### 日志输出

日志输出请使用xxl自带的`XxlJobLogger.log();`方法，使用和 `Slf4j` 一样，这个类是内置的，不需要手动import。

需要注意的是，如果通过`XxlJobLogger.log();`输出日志，日志不会显示在运行的进程的控制台里。而是会输出到日志文件中，日志文件的默认目录格式为：`/data/applogs/xxl-job/jobhandler/格式化日期/数据库调度日志记录的ID.log`

使用`System.out.println();`方法输出的方式会显示在运行的进程的控制台里，但是不会打印到日志文件中。

### GLUE脚本模式

- shell脚本：任务运行模式选择为 "GLUE模式(Shell)"时支持 "Shell" 脚本任务；

- python脚本：任务运行模式选择为 "GLUE模式(Python)"时支持 "Python" 脚本任务；

- php脚本：任务运行模式选择为 "GLUE模式(PHP)"时支持 "PHP" 脚本任务；

- nodejs脚本：任务运行模式选择为 "GLUE模式(NodeJS)"时支持 "NodeJS" 脚本任务；

- powershell：任务运行模式选择为 "GLUE模式(PowerShell)"时支持 "PowerShell" 脚本任务；`

脚本任务通过 Exit Code 判断任务执行结果