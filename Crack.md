### Typora激活（本地伪装）：

#### 1，设置激活状态为true

安装路径下找到 `resources\page-dist\static\js\LicenseIndex.xxx.chunk.js`
将 `e.hasActivated="true"==e.hasActivated` 替换为 `e.hasActivated="true"=="true"`

这样就已经后台激活完成，但是每次开软件还是会提醒激活。

#### 2，快速关闭已激活提示弹窗

在安装路径下找到 `resources\page-dist\license.html`
将 `</body></html>` 替换为 `</body><script>setTimeout(()=>{window.close();},50)</script></html>`

#### 3，去掉左下角"未激活"提示

在安装路径下找到 `resources\locales\zh-Hans.lproj\Panel.json` 
将 `"UNREGISTERED":"未激活"` 替换为 `"UNREGISTERED":" "`

重新Typora打开就可以了