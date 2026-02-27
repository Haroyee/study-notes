### 基于 C++11 标准的 Reactor 服务器框架

github 地址：https://github.com/Haroyee/C-Reactor

gitee 地址：https://gitee.com/poida/C-Reactor

平台工具：vim+vscode开发、makefile+g++编译、gdb调试

**项目描述:**

1. 基于 C++11 标准和 epoll 事件模型，采用主从Reactor多线程模型，支持高并发网络连接管理。
2. 采用模块化设计，核心组件包括 Reactor 事件调度器、AcceptorHandler 连接分配器、ConnectionHandler 连接处理器
2. epoll模型采用边缘触发模式
3. 支持两种客户端连接分配策略（ROUND_ROBIN、 LOAD_BASED）
5. 通过fcntl设置 socket 为非阻塞模式

**项目问题**

1. socket在bind阶段报错当前端口被占用。
2. 客户端连接服务端成功后，客户端发送给服务端无响应。

**分析定位问题**

1. 针对端口被占用问题，首先通过netstat命令查询占用端口的进程，kill对应进程，初步解决了问题，后续通过设置端口复用选项，运行端口复用。
2. 通过 gdb 调试发现事件循环阶段，从reactor查询连接处理器时条件不通过，从reactor处理器map为空。线程执行eventLoop时执行的是从reactor的拷贝，俩者的处理器map不同。用share_ptr管理从reactor数组，使线程执行的是从reactor自身的evenLoop而非拷贝的的evenLoop。

