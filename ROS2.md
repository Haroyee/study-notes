# ROS2

# 一、安装

1. 启用 Universe 仓库

ROS 2 依赖的部分包来自 Ubuntu 的 Universe 仓库，先确保开启：

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

2. **设置 locale（避免字符编码问题）**

```bash
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### 步骤 1：添加 ROS 2 软件源

1. **添加 GPG 密钥（验证包完整性）**

```bash
sudo apt update && sudo apt install -y curl gnupg2 lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

1. **添加 ROS 2 源到系统源列表**

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### 步骤 2：安装 ROS 2 Foxy

1. **更新软件源索引**

```bash
sudo apt update
```

若出现 “密钥过期 / 无法验证” 报错，重新执行步骤 1 的 GPG 密钥命令即可。

1. **选择安装版本（二选一）**

- **完整版（推荐，含所有工具、示例、文档）**：

```bash
sudo apt install -y ros-foxy-desktop
```

- **基础版（仅核心库，无图形化工具）**：

```bash
sudo apt install -y ros-foxy-ros-base
```

1. **安装开发工具（可选，用于编写 / 编译 ROS 2 代码）**

```bash
sudo apt install -y python3-colcon-common-extensions python3-rosdep python3-vcstool
```

### 步骤 3：初始化 rosdep（关键，解决依赖）

rosdep 用于自动安装 ROS 包的依赖，必须初始化：

```bash
sudo rosdep init
rosdep update
```

若出现 “sudo: rosdep：找不到命令”，先安装 rosdep：

```bash
sudo apt install -y python3-rosdep
```

若`rosdep update`超时 / 失败（网络问题），可换国内源（替换 rosdistro 文件）：

```bash
sudo sed -i 's/raw.githubusercontent.com/ghproxy.com\/https:\/\/raw.githubusercontent.com/g' /usr/lib/python3/dist-packages/rosdep2/sources_list.py
rosdep update
```

### 步骤 4：配置 ROS 2 环境（每次启动终端自动加载）

1. **临时配置（仅当前终端生效）**

```bash
source /opt/ros/foxy/setup.bash
```

1. **永久配置（推荐，所有终端生效）**

```bash
echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```