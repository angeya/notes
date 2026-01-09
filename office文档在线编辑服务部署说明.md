## 文档在线编辑服务部署说明

### 简要说明

本说明书上出现的所有绝对地址仅供参考，需要根据实际安装路径做出调整。

平台使用的在线编辑服务是 ice-blue Spire.Online Editor （收费的，需要根据IP地址和MAC地址获取授权证书。）

> 参考文档：
>
> https://cloud.e-iceblue.cn/index.php/tutorials/cloud/privatization/Spire.Online Editor-privatization-deployment-windows
>
> https://cloud.e-iceblue.cn/index.php/tutorials/cloud/privatization/spire-cloud-quick-guide

### 准备工作

把安装需要的资源包放入到服务器并解压。资源应该包含：chinesefonts、安装包、测试文件、证书。

分别是中文字体，软件安装包（nodejs、mysql、erlang环境、Rabbit、Spire.Online Editor），用来验证结果的文档，正版授权证书。

### 软件安装

#### nodejs

安装包 node-v12.22.12-x64.msi ，直接一步步next安装即可。



#### erlang环境

安装包 otp_win64_23.3.4.20.exe ，直接一步步next安装即可。



#### RabbitMQ

安装包 rabbitmq-server-3.7.7.exe，直接一步步next安装即可。

安装完成后需要通过插件开启控制台功能，并且创建用户、密码和角色。

通过 http://127.0.0.1:15672 登录验证安装结果。



#### MySql

安装包 mysql-5.7.40-winx64.zip ，需要先解压，最后复制到合适的软件安装目录（有空格也没有关系啦），然后进行初始化。

安装完成之后需要创建用户，设置密码，开启远程登录权限，并设置开机自启。



#### Spire.Online Editor

安装包 Spire.Cloud.Window.CnPriApis_10.7.10/OnlineEditorSetup.exe，也是一步步next安装即可。

等需要填入RabbitMQ和MySql信息的时候，根据上面的安装信息填入即可，单机版不需要Redis。



### 初始化数据库

使用命令行初始化Spire.Online Editor的数据库

```cmd
mysql -h 127.0.0.1 -u root -p < "C:\Program Files (x86)\Spire Online Editor\packges\mysql57\createdb.sql"
```



### 绑定/更换证书

把证书 license.elic.xml 放入如下目录即可`C:\Program Files (x86)\Spire Online Editor\spire.cloud\service\ConverterService\bin\license`

绑定证书后需要重启或者开启相关服务，名称类似iceblue_xxx，大概有5个。



### 验证在线编辑服务

浏览器打开 http://127.0.0.1:3000，正常能够显示示例页面，从测试文件夹中上传docx或者xlsx等文件，打开可以正常编辑和保存。

集成的话，使用8000端口。



### 安装字体（可选）

这里一般安装中文字体就够了。

使用 windonws 系统进行私有化部署时，Spire.Online Editor 提供了 SpireEditorFontInstaller.exe 程序用于扫描字体。运行该程序使用本地字体。

- 如果字体文件拷贝在 spire.cloud/font 目录下，只需要执行 Install 安装按钮即可拷贝并扫描字体。
- 再次打开文档时，**Ctrl+F5** 强制刷新页面，如果仍不生效，则需要清理浏览器缓存后再打开文档。

可参考：https://cloud.e-iceblue.cn/index.php/tutorials/cloud/privatization/font-issues

### 替换文件（可选）

原始的界面会包含ice-blue的logo，以及如新建文件等不符合平台需求的元素，应当去掉。html和css文件已经修改好了，直接使用 **替换文件** 中的文件根据所在目录从Spire.Online Editor安装目录中替换掉即可。



### 在平台中集成使用

#### nacos配置

`sciyon.filesetting.docEditServiceUrl` 配置onlineEditor的服务器地址，如`http://10.10.24.100:8000`

`oss.file-server` 配置oss的地址，如`http://10.44.2.116:9000`



#### 开启编辑服务

系统开关：系统设置 -> 文件服务设置 -> 是否启用在线文档编辑。把开关打开即可。

组件支持：在平台的附件上传组件`sciyon-uploader`中，配置属性`:fileEdit="true"`即可，当遇到可以编辑的文件组件会显示编辑按钮。目前支持word文档和excel表格。



#### 编辑文件

在文件上传测试功能中，上传一个`docx`文件，进行编辑！



#### 基本工作原理

1. 平台提供基础的html页面，内含基本的编辑器配置，并引用`http://onlineEditorService:8000/web/editors/spireapi/SpireCloudEditor.js`提供强大的编辑能力。
2. 文档只要浏览器能访问到，编辑器中就可以。如果需要带token，在`fileAttrs.fileInfo.name`中设置，如：`http://xxx/demo.pdf?token=123`。（传token的这一点是平台已经稳定运行后，冰蓝的人员才告知的，因此目前的是不太鉴权的文件url，直接访问oss的地址）
3. 平台组件打开编辑器的时候，会把编辑器的html页面放在弹窗的iframe里面，并且把文件的id和链接传过来。
4. 保存的时候通过文件id、编辑服务中文件的url传给后台，后台替换掉之前的文件即可。



### 注意点

冰蓝的官方文档写的非常的糟糕！比本文档还要糟糕。