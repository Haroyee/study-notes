# Shell编程

# 一、 核心概念：什么是 Shell 和 Shell 脚本？

在开始之前，我们先理解两个基本概念：

1. **Shell (壳)**：
   - 它是一个命令行解释器，充当用户和操作系统内核之间的 “桥梁”。
   - 当你在终端（Terminal）里输入 `ls`、`cd` 等命令时，就是由 Shell 来解析并执行的。
   - 常见的 Shell 有 `sh` (Bourne Shell), `bash` (Bourne Again Shell), `zsh`, `fish` 等。**我们主要学习 `bash`，因为它是最常用、功能最强大的标准 Shell。**
2. **Shell 脚本 (Shell Script)**：
   - 简单来说，就是一个包含一系列 Shell 命令的文本文件。
   - 它的作用是**自动化**一系列命令的执行，避免你每次都手动输入。
   - 它可以实现非常复杂的任务，从简单的文件备份到复杂的系统管理。



# 二、shell基础

## 1、创建并执行一个shell脚本

### （1）**创建shell脚本有多种方式**:

```bash
#vim 创建，不存在时会新建文件，存在则打开文件
vim new_script.sh

#touch 创建
touch new_script.sh
```

### （2）**编写内容：**

- **添加 Shebang**：在脚本的**第一行**，必须写上 `#!/bin/xxx`。这告诉系统要用 什么环境 来执行这个脚本。

  ```sh
  #!/bin/bash     # Bash (最常用)
  #!/bin/sh      # POSIX Shell
  #!/bin/zsh     # Z Shell
  #!/bin/ksh     # Korn Shell
  ```

- 编写命令

  ```sh
  #!/bin/bash     # Bash (最常用)
  echo "hello world!"
  ```

### （3）执行shell脚本

- **修改shell脚本的操作权限（重要）：**

  ```bash
  #利用chmod命令修改文件权限，755代表当前用户可读写执行，同组用户和其他用户只可读和执行
  chmod 755 new_script.sh
  ```

- **执行shell脚本：**

  ```bash
  #直接执行
  ./new_script.sh
  #使用bash执行
  bash new_script.sh
  #当前环境执行
  source new_script.sh
  . new_script.sh
  ```

- **输出：**

  ```bash
  $. new_script.sh
  hello world!
  ```

## 2、shell变量

