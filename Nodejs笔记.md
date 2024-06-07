## 1.Node.js 简介

### 特点

1. **异步和事件驱动**：Node.js 使用非阻塞、事件驱动的 I/O 模型，使其轻量且高效。
2. **单线程**：Node.js 使用单线程处理多并发连接，通过事件循环实现高效的并发处理。
3. **跨平台**：支持 Windows、Linux 和 macOS。

### 适用场景

- RESTful API 服务

- 实时应用（聊天、游戏）

- 数据流应用（如音频或视频流）

- 命令行工具

  

## 2.使用步骤

### 1. 安装 Node.js

从 [Node.js 官方网站](https://nodejs.org/) 下载并安装 Node.js。安装包中还包含 npm（Node 包管理工具）。

### 2. 基本命令

- 查看 Node.js 版本：

  ```sh
  node -v

- 查看 npm 版本：

  ```sh
  npm -v
  ```

- 设置npm缓存和全局模块安装路径：在安装目录下创建node_cache和node_global目录，使用以下命令进行配置

  ```shell
  npm config set prefix "xxx/node_global"
  npm config set cache "xxx/node_cache"
  ```

  用户目录下：.npmrc文件记录了以上路径配置。.node_repl_history文件记录node命令交互历史。

- 设置淘宝镜像源

  ```bash
  # 全局设置源
  npm config set registry https://registry.npm.taobao.org/
  # 查看当前源，验证是否配置成功
  npm config get registry
  # 安装包时使用某个源
  npm i packageName --registry=地址
  ```

- 恢复为npn官方镜像源

  ```bash
  npm config set registry https://registry.npmjs.org/
  ```




## 3. 创建第一个 Node.js 应用

创建一个简单的 HTTP 服务器。

1. 创建一个新的文件夹，例如 `my-first-node-app`，并在其中创建一个名为 `app.js` 的文件。

2. 在 `app.js` 文件中编写以下代码：

   ```javascript
   const http = require('http');
   
   const hostname = '127.0.0.1';
   const port = 3000;
   
   const server = http.createServer((req, res) => {
     res.statusCode = 200;
     res.setHeader('Content-Type', 'text/plain');
     res.end('Hello, World!\n');
   });
   
   server.listen(port, hostname, () => {
     console.log(`Server running at http://${hostname}:${port}/`);
   });
   ```

3. 在命令行中运行你的应用：

   ```
   node app.js
   ```

4. 打开浏览器并访问 `http://127.0.0.1:3000`，你会看到 "Hello, World!"。

## 4. Node.js 模块

Node.js 提供了丰富的内置模块，你可以使用这些模块来完成各种任务。常见的模块包括：

- `http`: 用于创建 HTTP 服务器和客户端。
- `fs`: 文件系统模块，用于处理文件的读取和写入。
- `path`: 用于处理和转换文件路径。
- `events`: 事件模块，用于事件驱动编程。

## 5. 使用 npm 管理包

npm 是 Node.js 的包管理工具，允许你轻松安装、更新和卸载第三方包。

- 初始化一个新的 npm 项目：

  ```
  npm init -y
  ```

- 安装一个包，例如 Express（一个流行的 web 框架）：

  ```
  npm install express
  ```

- 使用已安装的包：

  ```javascript
  const express = require('express');
  const app = express();
  const port = 3000;
  
  app.get('/', (req, res) => {
    res.send('Hello, Express!');
  });
  
  app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}/`);
  });
  ```

## 6.使用NVM管理Nodejs

### NVM介绍

NVM（Node Version Manager）是一个用来管理多个 Node.js 版本的工具，它允许你在同一台计算机上安装和切换不同版本的 Node.js。NVM 有助于在不同的项目中使用不同的 Node.js 版本，尤其是当你需要测试和调试时。

使用nvm的好处：

1. **安装不同版本的 Node.js**：你可以轻松地安装任何版本的 Node.js。
2. **切换 Node.js 版本**：你可以快速切换到所需的 Node.js 版本。
3. **设置默认版本**：你可以设置一个默认的 Node.js 版本，这个版本会在新终端窗口中自动使用。
4. **全局模块隔离**：不同版本的 Node.js 有各自的全局模块(node_global和node_cache目录隔离)，避免了不同项目之间的模块冲突。

### 安装 NVM

** Windows**

1. 从 [nvm-windows 的 GitHub 页面](https://github.com/coreybutler/nvm-windows/releases) 下载最新的安装程序，最好使用`exe`安装包进行安装。

2. 运行安装程序并按照提示完成安装。

3. 安装完成后，打开命令提示符（cmd）或 PowerShell 并验证安装：

   ```sh
   nvm version
   ```

**macOS 和 Linux**

1. 打开终端并运行以下命令安装 NVM：

   ```sh
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
   ```

2. 安装完成后，重新加载 shell 配置文件：

   ```sh
   source ~/.zshrc   # 对于 Zsh
   ```

3. 验证安装：

   ```sh
   nvm --version
   ```

### 使用 NVM

要安装指定版本的 Node.js，例如 `14.17.0`：

```
nvm install 14.17.0
```

查看已安装的 Node.js 版本列表：

```
nvm list installed
```

查看所有可以安装的 Node.js 版本：

```
nvm list available
```

使用特定版本的 Node.js:

```
nvm use 14.17.0
```

设置默认版本，当你打开新的终端窗口时会使用该版本：

```
nvm alias default 14.17.0
```

卸载指定版本的 Node.js：

```
nvm uninstall 14.17.0
```