# Git

## 一、如何上传自己的项目到Gitee

### 1、在项目目录右键点击Git Bash Here，执行以下代码

```powershell
git init
```

### 2、配置用户名与邮箱

用户名：

```bash
git config --global user.name 'your name' 
```

邮箱：

```bash
git config --global user.email 'your email'
```

### 3、连接项目路径

origin：自定义存储库的名称

```bash
git remote set-url origin 项目路径
```

```bash
git remote add origin 项目路径
```

### 4、按顺序执行以下命令

拉取远程存储库的内容

```bash
git pull origin master
```

将所有文件的改动添加到暂存区

```bash
git add .
```

将暂存区的内容提交到版本库，并附上说明

```bash
git commit -m '你的项目说明'
```

```bash
git push origin master
```

### 5、输入验证信息

你的用户名以及令牌

## 二、常用命令

### 1、初始化仓库

在当前目录初始化一个新的 Git 仓库

```bash
git init
```

### 2、克隆已有仓库

从远程服务器克隆一个已存在的仓库。

```bash
git clone <url>
```

### 3、查看仓库状态

```bash
git statue
```

### 4、将文件存储到暂存区

```bash
git add 文件名
git add .    #存储所有文件
```

### 5、删除文件

```bash
git rm 文件  #删除暂存区与工作区文件
git rm --cache 文件  #删除暂存区文件
#可使用git commit 更新删除文件
```

### 6、查看日志

```bash
git log --online #提交日志
git reflog --online #操作日志（提交与回退日志）
```

### 7、回退

```bash
#版本id可由git log 获取
git reset --soft  版本id #只回退在线仓库，暂存区与工作区不变
git reset --hard  版本id #回退三个仓库的版本 
git reset --mixed 版本id #只保留工作区
```

### 8、分支

```bash
git brach 分支名       #新建分支
git checkout/switch 分支名    #切换分支
git merge 分支名       #将分支名的分支合并到目前分支，合并若冲突可以通过git statue和git diff定位冲突内容
git rebase 分支名      #将分支名的分支合并到目前分支，通常不在主分支执行该命令
git brach -d(D) 分支名 #d删除分支(已合并)，D强制删除分支(未合并)
```

### 9、拉取合并

```bash
git fetch origin master #仅从远程仓库下载更新，但不合并
git pull origin master #从远程仓库拉取更新并合并到本地（=git fetch + git merge）。
```



三、vscode中使用git

1、初始化仓库

![image-20250907164321635](C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20250907164321635.png)

2、添加远程仓库

![image-20250907164542441](C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20250907164542441.png)

输入仓库路径

![image-20250907164703230](C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20250907164703230.png)

输入仓库名称

![image-20250907164754381](C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20250907164754381.png)

## 三、代理

常用代理

```shell
https://gh-proxy.com/
```



## 四、使用时遇到的问题及处理方式

