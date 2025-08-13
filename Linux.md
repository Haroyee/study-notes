# Linux

## 一、常用命令

### 1、文件与目录操作

| 命令    | 作用          | 示例                                             |
| :------ | :------------ | :----------------------------------------------- |
| `ls`    | 列出目录内容  | `ls -l /home`（详细列表）                        |
| `cd`    | 切换目录      | `cd ~/Documents`（进入用户文档目录）             |
| `pwd`   | 显示当前路径  | `pwd`                                            |
| `mkdir` | 创建目录      | `mkdir -p project/{src,bin}`（递归创建嵌套目录） |
| `rm`    | 删除文件/目录 | `rm -r old_dir`（递归删除目录）⚠️                 |
| `cp`    | 复制文件      | `cp file.txt backup/ -v`（显示复制详情）         |
| `mv`    | 移动/重命名   | `mv file.txt new_name.txt`（重命名）             |
| `touch` | 创建空文件    | `touch log.txt`                                  |

### 2、文件查看与编辑

| 命令   | 作用         | 示例                                            |
| :----- | :----------- | :---------------------------------------------- |
| `cat`  | 显示文件内容 | `cat -n config.conf`（带行号显示）              |
| `less` | 分页查看文件 | `less system.log`（按`q`退出）                  |
| `head` | 查看文件头部 | `head -n 5 access.log`（前5行）                 |
| `tail` | 查看文件尾部 | `tail -f live.log`（实时追踪日志）              |
| `grep` | 文本搜索     | `grep -i "error" /var/log/syslog`（忽略大小写） |
| `nano` | 简单文本编辑 | `nano notes.md`                                 |
| `vim`  | 高级文本编辑 | `vim script.sh`                                 |

### 3、系统信息监控

| 命令    | 作用           | 示例                                |
| :------ | :------------- | :---------------------------------- |
| `top`   | 动态进程监控   | `top`（按`P`按CPU排序）             |
| `htop`  | 增强版进程监控 | `htop`（需安装）                    |
| `df`    | 磁盘空间检查   | `df -h`（人类可读格式）             |
| `free`  | 内存使用情况   | `free -m`（以MB显示）               |
| `uname` | 系统信息       | `uname -a`（显示内核版本等）        |
| `ps`    | 进程快照       | `ps auxgrep nginx`（查找nginx进程） |