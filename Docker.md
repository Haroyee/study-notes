# Docker

## 一、常用命令

### 1、拉取镜像

```bash
docker pull docker.io/author/mirror:version
##docker.io官方仓库
##author命名空间/作者
##mirror镜像名
##version镜像版本
```

### 2、查看已下载镜像

```bash
docker images
```

### 3、删除镜像

```bash
docker rmi name/id
```

### 4、查看运行中的容器

```
docker ps
```

### 5、启动容器

- -d :表示分离运行
- --name :为容器命名
- -p :端口映射
- -v ：绑定挂载
- -e ：自定义环境变量

```bash
docker run -d \
--name gsuidcore \
-e TZ=Asia/Shanghai \
-e GSCORE_HOST=0.0.0.0 \
-p 18765:8765 \
-v /opt/gscore_data:/gsuid_core/data \
-v /opt/gscore_plugins:/gsuid_core/gsuid_core/plugins \
lilixxs666/gsuid-core:dev

docker create ... #使用镜像创建容器，用法与docker run相同，不过运行需要使用docker start 容器id
```

**关于挂载**

通过挂载可以时容器与宿主机共享一个文件目录，对这个目录操作会在宿主机和容器中发生相同的变化。

而容器删除后，这个目录的数据仍留在宿主机中，因此挂载主要有以下功能：

1. **数据不丢失**：容器删除 / 重启后，关键数据（如 MySQL 数据文件）仍保留；
2. **数据共享**：多个容器可访问同一组数据（如 Nginx 和 PHP-FPM 共享静态文件）；
3. **方便管理**：宿主机直接修改配置 / 数据，无需进入容器；

**如何使用挂载**

```bash
-v 宿主机目录:容器内目录
## or
-v 卷的名字:容器内目录
```

**其他参数：**

- --restart ：容器重启策略

  - `docker run -d --restart always nginx`（总是重启）

    `docker run -d --restart on-failure:3 nginx`（失败重启最多 3 次）

    `docker run -d --restart unless-stopped nginx`（除非手动停止，否则重启）

- --rm ：容器停止后**自动删除**，适合临时测试（避免残留无用容器）

- -it ：交互模式运行（`-i` 保持标准输入打开，`-t` 分配伪终端），常用于调试

  - `docker run -it ubuntu /bin/bash`（以/bin/bash终端进入 Ubuntu 容器终端）

### 6、卷操作

```bash
docker volume create mysql-data-new #创建命名卷（推荐）
docker volume ls #查看所有数据卷
docker volume inspect mysql-data #查看卷详情（含宿主机实际路径）


#删除卷（需先停止/删除使用该卷的容器）
docker rm -f mysql8  # 停止并删除容器
docker volume rm mysql-data  # 删除卷

# 清理无主卷（匿名卷/未使用的命名卷）
docker volume prune
```

### 7、停止/启动/重启容器

```bash
docker stop/start/restart 容器ID # 停止/启动/重启容器
```

### 8、删除容器

```bash
docker rm 容器ID               # 删除容器
```

### 9、查看容器日志

```bash
docker logs 容器ID             # 查看容器日志
docker logs --follow/-f 容器ID # 持续追踪
```

### 10、进入容器内进行调试

```bash
docker exec -it 容器ID /bin/bash

cat /ect/os-release #查看容器的操作系统环境版本
```

## 二、构建镜像

### 1、编写Dockerfile

```dockerfile
# FROM后接镜像-基础镜像：Python 轻量版
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 复制依赖文件到，工作目录/app（利用缓存，代码变动不重装依赖）
COPY requirements.txt .

# 运行安装依赖（--no-cache-dir 避免缓存，减小镜像体积）
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码到工作目录
COPY . .

# 暴露端口
EXPOSE 5000

# 启动命令（用 gunicorn 替代内置服务器）
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

2、通过dockerfile构建镜像

```bash
docker build -t docker_name .
# . 表示用当前目录的dockerfile构建名为docker_name的镜像
```

3、推送自己构建的镜像到自己的仓库

```bash
#1.登陆
docker login 
#2.构建镜像，镜像名字前需要带上作者名
docker build -t author/docker_name.
#3.推送镜像
docker push author/docker_name
```

## 三、多容器编排技术

1、dockercompose

编写 `docker-compose.yml` 文件

例如有以下容器：

```bash
#gscore
docker run -d \
--name gsuidcore \
-e TZ=Asia/Shanghai \
-e GSCORE_HOST=0.0.0.0 \
-p 18765:8765 \
-v /opt/gscore_data:/gsuid_core/data \
-v /opt/gscore_plugins:/gsuid_core/gsuid_core/plugins \
lilixxs666/gsuid-core:dev

#astrbot
sudo docker run -itd \
-p 6180-6200:6180-6200 \
-p 11451:11451 \
-v $PWD/data:/AstrBot/data \
-v /etc/localtime:/etc/localtime:ro \
-v /etc/timezone:/etc/timezone:ro \
--name astrbot \
m.daocloud.io/docker.io/soulter/astrbot:latest

#napcat
docker run -d \
-e NAPCAT_GID=$(id -g) \
-e NAPCAT_UID=$(id -u) \
-p 3000:3000 \
-p 3001:3001 \
-p 6099:6099 \
--name napcat \
--restart=always \
mlikiowa/napcat-docker:latest
```

docker-compose.yml文件为：

```yml
version: '3.8'  # Compose 文件版本，推荐使用3.8（兼容Docker 19.03+）

services:
  # 第一个服务：gsuidcore（对应gscore）
  gsuidcore:
    container_name: gsuidcore  # 容器名称，与原命令一致
    image: lilixxs666/gsuid-core:dev  # 镜像地址
    restart: unless-stopped  # 重启策略（推荐，除非手动停止否则重启）
    environment:  # 环境变量
      - TZ=Asia/Shanghai
      - GSCORE_HOST=0.0.0.0
    ports:  # 端口映射
      - "8765:8765"
    volumes:  # 数据卷挂载
      - ./gscore_data:/gsuid_core/data  # 宿主机目录:容器目录
      - ./gscore_plugins:/gsuid_core/gsuid_core/plugins
    networks:
      - app-network  # 加入自定义网络（方便容器间通信）

  # 第二个服务：astrbot
  astrbot:
    container_name: astrbot  # 容器名称，与原命令一致
    image: m.daocloud.io/docker.io/soulter/astrbot:latest  # 镜像地址
    restart: unless-stopped  # 补充重启策略（原命令未指定，建议添加）
    ports:  # 端口映射（多端口范围）
      - "6180-6200:6180-6200"
      - "11451:11451"
    volumes:  # 数据卷挂载（ro 表示只读）
      - ./data:/AstrBot/data  # 相对路径（当前目录data），原命令$PWD/data等价于./data
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    networks:
      - app-network  # 加入自定义网络

  # 第三个服务：napcat
  napcat:
    container_name: napcat  # 容器名称，与原命令一致
    image: mlikiowa/napcat-docker:latest  # 镜像地址
    restart: always  # 重启策略（与原命令--restart=always一致）
    environment:  # 环境变量（获取当前用户UID/GID，Linux下自动替换）
      - NAPCAT_UID=${NAPCAT_UID}  # 自动读取.env中的值
      - NAPCAT_GID=${NAPCAT_GID}
    ports:  # 端口映射
      - "3000:3000"
      - "3001:3001"
      - "6099:6099"
    networks:
      - app-network  # 加入自定义网络

# 自定义网络（可选，方便容器间通信，如astrbot调用napcat/gsuidcore）
networks:
  app-network:
    driver: bridge  # 默认桥接网络，与Docker默认网络模式一致
```

同时编写start.sh

```bash
#!/bin/bash
set -e  # 出错时退出

# 1. 自动获取当前用户的UID/GID
NAPCAT_UID=$(id -u)
NAPCAT_GID=$(id -g)

# 2. 生成.env文件（覆盖原有内容）
echo "NAPCAT_UID=${NAPCAT_UID}" > .env
echo "NAPCAT_GID=${NAPCAT_GID}" >> .env

# 3. 启动Compose
docker-compose up -d

# 4. 验证变量是否生效
echo "✅ 启动完成"

```



docker-compose**常用命令**

```bash
# 后台启动所有服务（-d 后台运行）
docker-compose up -d
# 查看服务状态
docker-compose ps
# 查看日志（如查看astrbot日志）
docker-compose logs -f astrbot
# 停止所有服务
docker-compose stop
# 停止并删除容器（保留数据卷）
docker-compose down
# 停止并删除容器+数据卷（谨慎使用）
docker-compose down -v
```

