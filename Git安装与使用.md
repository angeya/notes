# Git安装与使用

## 1. 安装Git

- 进入官网 https://git-scm.com/ 下载并安装git（全部next即可）
- Git可以使用Git GUI和Git Bash，可以结合一起使用提高效率

## 2. 基础配置（git bash）

- 通过以下命令配置Git账户的用户名和邮箱，提交时别人会看到。

  ```shell
  git config --global user.name 'yourName'
  git config --global user.email 'yourEmail'
  ```

- 查看已有配置

  ```shell
  git config --list
  ```

第一次提交的时候，需要输入仓库的用户名密码，以后不用输入了。也可以把本机公钥加入到远程仓库中。

## 3. Git常用命令

- 常用命令

  ```shell
  git init # 项目初始化 
  git clone # 远程克隆项目到本地 
  git add file # 添加到暂存区 
  git commit -m "" # 添加commit信息 
  git push origin # 将本地分支推送到服务器上去 
  git pull origin # 拉取远程仓库代码与本地合并（相当于git fetch + git merge）
  git fetch # 拉取远程代码，但是不合并
  git merge # 合并远程代码和本地代码
  git pull origin master # 
  
  # 版本恢复
  git reset --hard commitID # 将暂存区设置为某个版本。既可以实现版本回退，也可以做实现重做（tortoise需要再show log中操作）
  git revert commitID # 撤销提交（创建一个新的提交，反转之前提交的操作）（tortoise需要再show log中操作，并需要手动提交）
  
  # 分支
  git branch # 查看所有分支和当前使用的分支
  git branch branchName # 创建一个新的分支
  git checkout branchName # 使用指定的分支，加上参数 -b 如果没有分支会创建再切换
  git push --set-upstream origin branchName # 提交到远程仓库指定分支，如果远程没有分支，则创建
  git merge srcBranch # 分支合并，指定支合并到当前分支
  git branch -d branchName # 删除指定分支（会做一些检查，比如是否已经合并），参数 -D 强制删除
  
  git log # 查看日志 
  git status # 查看当前状态，可以查看和远程仓库是不是一样的，不一样则有提交没有push
  git tag # 查看版本号 
  git diff # 查看尚未提交的更新
  
  git remote -v # 查看远程仓库信息
  git remote set-url origin url # 修改远程仓库地址
  ```

- fetch与pull

  pull是拉取当前服务器上最新的内容，并合并到当前本地仓库。

  fetch是拉取最新的内容，但是并不会自动合并到本地仓库，fetch到的信息保存在FETCH_FIRST文件中

  可以通过git log -p master..origin/master比较本地仓库与远程仓库的区别，也可以通过

  git merger origin/master命令合并fetch到的当前的本地仓库。
  
- 冲突解决

  当提交之后，上传到远程仓库发生冲突时，可以使用 git pull 命令拉取并合并远程仓库最新代码
  
  git pull命令会尝试自动合并代码，解决冲突，如果解决了只需要备注合并信息，然后再 push 即可
  
  如果 git 无法自动解决冲突，这时就需要手动解决（有冲突的两个版本的代码被>>>和===分隔开），解决好后再 add commit push 即可

## 4. Git工作示意图

![image-20220323202621424](images/git-construction.png)

## 5. 解决 Failed to connect to github.com port 443:connection timed out超时
```shell
# 设置代理
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy http://127.0.0.1:1080
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 6. Java项目 .gitignore 文件样例

```
#设置忽略文件
node_modules/
dist/
package-lock.json

#MAVEN打包文件忽略
target/

#文件格式忽略
*.log
*.class
*.war
*.iml
*.jar
*.nar
*.ear
*.zip
*.tar.gz
*.rar

#文件忽略
.idea/
.idea
.vscode
.project
.classpath
.settings

```

### 7.一些使用案例

### 7.1 代码已经被修改甚至被commit，这时候来了一个更紧急的任务

如果代码被commit，需要取消commit，然后stash changes。

这时代码被还原成了没有被修改的状态，即可进行更紧急任务的开发，开发完成push之后，stash pop可以弹出之前暂存的修改，继续操作。

### 7.2 把本地项目推到 github，但是github没有对应的仓库

1. 创建 Github 仓库： 在 Github 上创建一个新的仓库，用于存储本地项目的代码。你可以选择公开或私有，初始化 README 文件等选项，一般默认在main分支。

2. 在本地项目中运行 Git 初始化： 进入本地项目所在的目录，在终端（或命令行）中运行 `git init` 命令，初始化 Git 仓库。

3. 将本地代码提交到 Git 仓库： 使用 `git add .` 命令将所有变更添加到暂存区，使用 `git commit -m "Initial commit"` 命令将变更提交到本地 Git 仓库。

4. 关联本地 Git 仓库和 Github 仓库： 通过运行 `git remote add origin https://github.com/your-username/your-repository.git` 命令，将本地 Git 仓库与 Github 仓库关联起来。

5. 推送本地代码到 Github 仓库： 最后，运行 `git push -u origin main/master` 命令（一般main或者master分支），将本地代码推送到 Github 仓库中。

   也可以先执行 `git pull origin master --allow-unrelated-histories` 命令，将远程 Github 仓库中的文件同步到本地仓库中，然后再执行 `git push -u origin master` 命令。



