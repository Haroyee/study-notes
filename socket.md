# SOCKET

计算机网络理论是网络编程的 “内功”，尤其是 TCP/IP 协议栈的核心概念，直接决定了你对网络编程中 “数据如何传输”“为什么需要特定 API” 的理解。下面从 “分层模型” 入手，聚焦与网络编程最相关的核心知识点，兼顾理论和实际编程场景的关联。

### **一、网络体系结构：为什么要分层？**

计算机网络是复杂系统，为了降低设计难度，采用 “分层思想”：每一层专注解决特定问题，通过 “接口” 向上层提供服务，向下层依赖服务。

- **OSI 七层模型**（理论模型）：物理层 → 数据链路层 → 网络层 → 传输层 → 会话层 → 表示层 → 应用层（实际中很少严格遵循）。

- **TCP/IP 四层模型**

  （实际应用模型）：更简洁，是网络编程的核心参考：

  ```plaintext
  应用层（HTTP/FTP/DNS等）  
  传输层（TCP/UDP）  
  网络层（IP/ICMP）  
  网络接口层（物理层+数据链路层，如以太网、Wi-Fi）  
  ```

  

**核心逻辑**：数据在发送方从 “应用层” 向下逐层 “封装”（加头部信息），在接收方从 “网络接口层” 向上逐层 “解封装”（拆头部），最终到达应用层。

### **二、从底层到高层：核心层与编程关联**

重点关注与网络编程直接相关的**网络层**和**传输层**，以及**应用层**的基础协议。

#### **1. 网络接口层（物理层 + 数据链路层）**

- 作用

  ：负责 “物理传输” 和 “相邻设备通信”。

  - 物理层：定义电压、接口等物理特性（如网线、无线信号），传输 “比特流”（0 和 1）。
  - 数据链路层：将比特流包装成 “帧”（Frame），解决 “同一局域网内设备识别”（通过 MAC 地址）和 “差错检测”（如 CRC 校验）。

- 与编程的关联

  ：几乎不直接操作（由操作系统和网卡驱动处理），但需知道：

  - 局域网内通信依赖 MAC 地址（如 ARP 协议用于 IP 地址转 MAC 地址）。
  - 帧有长度限制（如以太网 MTU 通常 1500 字节），过大的数据会被分片（影响传输效率）。

#### **2. 网络层：实现 “跨网络通信”**

核心协议：**IP 协议**（IPv4 为主，IPv6 逐步普及），解决 “不同网络间的数据路由” 问题。

- **核心概念**：
  - **IP 地址**：标识网络中一台主机的 “逻辑地址”（如 IPv4 的`192.168.1.1`，32 位；IPv6 的`2001:db8::1`，128 位）。
  - **子网与掩码**：通过子网掩码（如`255.255.255.0`）划分网络，区分 “本网段” 和 “跨网段”（跨网段需通过网关转发）。
  - **路由**：数据从源主机到目标主机的 “路径选择”（由路由器完成，基于路由表）。
  - **IP 分组**：数据被拆分为 “IP 分组”（Packet）传输，每个分组独立路由（可能走不同路径），接收方再重组。
- **辅助协议**：
  - **ICMP**：控制报文协议，用于诊断网络（如`ping`命令基于 ICMP 的 echo 请求 / 应答）。
  - **ARP**：地址解析协议，将 IP 地址转换为 MAC 地址（同一局域网内通信需要）。
- **与编程的关联**：
  - 网络编程中需指定目标 IP 地址（如`connect()`函数需要 IP）。
  - 需处理 IP 分组分片（但通常由操作系统内核处理，应用层无需关心）。

#### **3. 传输层：实现 “进程间通信”**

核心协议：**TCP**和**UDP**，是网络编程的 “核心战场”，负责 “端到端” 的数据传输（区分同一主机上的不同进程）。

##### **（1）端口（Port）：进程的 “门牌号”**

- 一台主机有 65536 个端口（0-65535），用于区分同一主机上的不同进程（如 HTTP 默认 80，SSH 默认 22）。
- 网络通信的 “四元组”：`(源IP, 源端口, 目标IP, 目标端口)`，唯一标识一个网络连接。

##### **（2）TCP：可靠的 “字节流” 传输**

TCP（Transmission Control Protocol，传输控制协议）是 “面向连接、可靠、有序” 的协议，适合对数据准确性要求高的场景（如文件传输、HTTP）。

- **核心特性**：

  - **面向连接**：通信前必须建立连接（三次握手），结束后释放连接（四次次挥手）。

  - 可靠传输

    ：

    - 确认机制：接收方收到数据后发送 ACK 确认，发送方未收到 ACK 则重传。
    - 序列号：保证数据有序序（接收方按序号重组）。
    - 超时重传：发送方等待超时后重传未确认数据。

  - **流量控制**：通过 “滑动窗口” 限制发送速率，避免接收方缓冲区溢出（接收方告知自己的窗口大小）。

  - **拥塞控制**：通过慢启动、拥塞避免等算法，避免网络因数据过多而拥塞。

- **三次握手（建立连接）**：

  1. 客户端 → 服务器：SYN（请求建立连接，带初始序列号 seq=x）。

  2. 服务器 → 客户端：SYN+ACK（同意连接，seq=y，确认 ACK=x+1）。

  3. 客户端 → 服务器：ACK（确认收到，ACK=y+1）。

     

     （为什么三次？确保双方 “发送” 和 “接收” 能力都正常）

- **四次挥手（释放连接）**：

  1. 客户端 → 服务器：FIN（请求关闭，seq=u）。

  2. 服务器 → 客户端：ACK（确认关闭，ACK=u+1）。

  3. 服务器 → 客户端：FIN（服务器也准备关闭，seq=v）。

  4. 客户端 → 服务器：ACK（确认，ACK=v+1）。

     

     （为什么四次？因为服务器可能还有数据要发送，不能立即发 FIN）

- **与编程的关联**：

  - TCP 编程需严格遵循 “连接→通信→关闭” 流程（`listen()`/`connect()`/`accept()`）。
  - 数据是 “流式” 的（无边界），会导致 “粘包” 问题（需应用层设计协议解决）。

##### **（3）UDP：高效的 “数据报” 传输**

UDP（User Datagram Protocol，用户数据报协议）是 “无连接、不可靠、无序” 的协议，适合对实时性要求高的场景（如视频通话、游戏、DNS）。

- **核心特性**：
  - **无连接**：通信前无需建立连接，直接发送数据（速度快）。
  - **不可靠**：不保证送达，无确认、无重传（可能丢包、乱序）。
  - **数据报**：每次发送是一个独立 “数据报”（有边界），大小受限（通常不超过 1500 字节，避免分片）。
- **与编程的关联**：
  - UDP 编程无需`listen()`/`accept()`，直接用`recvfrom()`/`sendto()`收发（需指定目标 IP 和端口）。
  - 应用层需自己处理丢包、重传（如实时游戏中可忽略旧数据，或加简单确认机制）。

#### **4. 应用层：定义 “数据格式和交互逻辑”**

应用层协议基于 TCP 或 UDP，定义 “数据如何解析” 和 “双方如何交互”（如 HTTP 的请求 / 响应格式）。

- **常见协议**：
  - **HTTP**：基于 TCP，用于网页传输（请求方法 GET/POST、状态码 200/404 等）。
  - **FTP**：基于 TCP，用于文件传输（控制连接 + 数据连接）。
  - **DNS**：基于 UDP，用于域名解析（将`www.baidu.com`转为 IP）。
  - **SMTP/POP3**：基于 TCP，用于邮件发送和接收。
- **与编程的关联**：
  - 网络编程常需实现应用层协议的客户端 / 服务器（如写一个简单 HTTP 服务器，解析 GET 请求并返回数据）。
  - 自定义协议时，需设计数据格式（如 “头部 + 数据”，头部包含长度、类型等信息）。

### **三、核心总结：与网络编程的关联**

| 层级   | 核心协议 / 概念      | 网络编程中需关注的点                                |
| ------ | -------------------- | --------------------------------------------------- |
| 网络层 | IP 地址、路由        | 处理 IP 地址转换（字符串→二进制）、绑定本地 IP      |
| 传输层 | TCP（连接 / 可靠）   | 掌握`listen()`/`connect()`/`accept()`流程，处理粘包 |
| 传输层 | UDP（无连接 / 高效） | 用`recvfrom()`/`sendto()`收发，处理数据报边界       |
| 应用层 | 协议格式             | 设计或解析应用层数据（如自定义协议、HTTP 解析）     |

理解这些理论后，再学 Socket 编程会更清晰：比如`socket()`函数的第一个参数`AF_INET`对应 IP 协议族，第二个参数`SOCK_STREAM`对应 TCP（字节流），`SOCK_DGRAM`对应 UDP（数据报）。后续的并发模型、IO 多路复用，本质也是为了更高效地处理 TCP/UDP 连接和数据传输。

如果有某部分想深入（比如 TCP 的滑动窗口、UDP 的广播），可以再具体展开～



Socket 编程是 Linux 网络通信的核心，其本质是通过操作系统提供的 **套接 et API** 实现进程间的网络通信。无论是 TCP（可靠连接）还是 UDP（无连接），都需要通过 Socket 接口完成数据传输。下面将分 **TCP 通讯** 和 **UDP 通讯** 两种场景，详细讲解从创建到关闭的完整步骤，并附代码示例。

### 一、Socket 基础：核心概念与数据结构

在开始步骤前，先明确两个核心点：

- **Socket 本质**：Linux 中 “一切皆文件”，Socket 是一个特殊的 “文件描述符”（整数），通过 `read()`/`write()` 等 IO 函数即可读写网络数据。
- **地址结构**：网络通信需要标识 “对方的位置”（IP + 端口），Linux 用 `struct sockaddr_in` 存储 IPv4 地址信息（需转换为通用的 `struct sockaddr` 传给系统调用）。

c



运行









```c
// IPv4 地址结构（重点）
struct sockaddr_in {
    sa_family_t     sin_family;   // 协议族（必须是 AF_INET）
    in_port_t       sin_port;     // 端口号（网络字节序）
    struct in_addr  sin_addr;     // IP 地址（网络字节序）
    unsigned char   sin_zero[8];  // 填充字段，需设为 0
};

struct in_addr {
    uint32_t s_addr;  // 32位 IPv4 地址（网络字节序）
};
```

- 字节序转换

  ：网络协议规定 “网络字节序为大端序”，而本地主机可能是小端序，需用以下函数转换：

  - `htons()`：本地端口 → 网络端口（host to network short）
  - `htonl()`：本地 IP → 网络 IP（host to network long）
  - `ntohs()`/`ntohl()`：网络字节序 → 本地字节序

### 二、TCP 通讯步骤（可靠连接）

TCP 是 “面向连接” 的协议，通讯前必须先建立连接（三次握手），流程分为 **服务器端** 和 **客户端** 两部分，步骤严格对称。

#### 步骤 1：服务器端初始化（绑定端口 + 监听连接）

服务器需要先 “绑定” 一个端口并 “监听”，等待客户端连接。

| 步骤              | 系统调用                                                     | 作用                                         |
| ----------------- | ------------------------------------------------------------ | -------------------------------------------- |
| 1. 创建 Socket    | `int socket(int domain, int type, int protocol);`            | 创建一个 Socket 文件描述符                   |
| 2. 绑定 IP 和端口 | `int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);` | 将 Socket 与本地 IP + 端口绑定               |
| 3. 监听连接       | `int listen(int sockfd, int backlog);`                       | 将 Socket 转为 “被动监听” 状态，等待连接     |
| 4. 接受客户端连接 | `int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);` | 阻塞等待客户端连接，返回新的 Socket 用于通信 |

#### 步骤 2：客户端初始化（发起连接）

客户端不需要绑定固定端口（系统会自动分配临时端口），直接向服务器发起连接。

| 步骤           | 系统调用                                                     | 作用                                         |
| -------------- | ------------------------------------------------------------ | -------------------------------------------- |
| 1. 创建 Socket | `socket()` （同服务器）                                      | 创建客户端 Socket                            |
| 2. 发起连接    | `int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);` | 向服务器的 IP + 端口发起连接（触发三次握手） |

#### 步骤 3：数据传输（双向通信）

连接建立后，服务器（通过 `accept()` 返回的新 Socket）和客户端（通过自己的 Socket）可双向读写数据。

| 操作   | 系统调用                                                     | 作用                                   |
| ------ | ------------------------------------------------------------ | -------------------------------------- |
| 读数据 | `ssize_t recv(int sockfd, void *buf, size_t len, int flags);` | 从 Socket 读取数据（`flags` 通常为 0） |
| 写数据 | `ssize_t send(int sockfd, const void *buf, size_t len, int flags);` | 向 Socket 写入数据（`flags` 通常为 0） |

#### 步骤 4：关闭连接（释放资源）

通信结束后，需关闭 Socket 释放文件描述符（触发四次挥手）。

| 步骤        | 系统调用             | 作用                        |
| ----------- | -------------------- | --------------------------- |
| 关闭 Socket | `int close(int fd);` | 关闭 Socket，释放连接和资源 |

#### TCP 代码示例：回声服务器与客户端

**服务器端（echo_server.c）**：接收客户端消息并原样返回

c



运行









```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUF_SIZE 1024

int main() {
    // 1. 创建 TCP Socket（IPv4，字节流，TCP协议）
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket failed");
        exit(1);
    }

    // 2. 绑定 IP 和端口（本地任意IP，端口 8888）
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;                 // IPv4
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 绑定所有本地IP
    serv_addr.sin_port = htons(PORT);               // 端口（网络字节序）

    if (bind(listen_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind failed");
        close(listen_fd);
        exit(1);
    }

    // 3. 监听连接（最大等待队列长度为 5）
    if (listen(listen_fd, 5) == -1) {
        perror("listen failed");
        close(listen_fd);
        exit(1);
    }
    printf("Server listening on port %d...\n", PORT);

    // 4. 接受客户端连接（阻塞等待）
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int conn_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_addr_len);
    if (conn_fd == -1) {
        perror("accept failed");
        close(listen_fd);
        exit(1);
    }
    printf("Client connected: %s:%d\n", 
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    // 5. 数据通信：接收并回声
    char buf[BUF_SIZE];
    while (1) {
        // 读客户端数据
        ssize_t n = recv(conn_fd, buf, BUF_SIZE-1, 0);
        if (n <= 0) {
            if (n == 0) printf("Client disconnected\n");
            else perror("recv failed");
            break;
        }
        buf[n] = '\0';
        printf("Received: %s\n", buf);

        // 回声回去
        send(conn_fd, buf, n, 0);
    }

    // 6. 关闭连接
    close(conn_fd);
    close(listen_fd);
    return 0;
}
```

**客户端（echo_client.c）**：向服务器发送消息并接收回声

c



运行









```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define SERVER_IP "127.0.0.1"  // 服务器IP（本地回环）
#define PORT 8888
#define BUF_SIZE 1024

int main() {
    // 1. 创建 TCP Socket
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (sock_fd == -1) {
        perror("socket failed");
        exit(1);
    }

    // 2. 连接服务器
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    // 将字符串IP转为网络字节序IP
    if (inet_pton(AF_INET, SERVER_IP, &serv_addr.sin_addr) <= 0) {
        perror("invalid address");
        close(sock_fd);
        exit(1);
    }

    if (connect(sock_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("connect failed");
        close(sock_fd);
        exit(1);
    }
    printf("Connected to server %s:%d\n", SERVER_IP, PORT);

    // 3. 发送并接收数据
    char buf[BUF_SIZE];
    while (1) {
        printf("Enter message (q to quit): ");
        fgets(buf, BUF_SIZE, stdin);
        // 移除换行符
        if (buf[strlen(buf)-1] == '\n') {
            buf[strlen(buf)-1] = '\0';
        }

        // 发送数据
        send(sock_fd, buf, strlen(buf), 0);

        // 退出条件
        if (strcmp(buf, "q") == 0) break;

        // 接收回声
        ssize_t n = recv(sock_fd, buf, BUF_SIZE-1, 0);
        if (n <= 0) {
            perror("recv failed");
            break;
        }
        buf[n] = '\0';
        printf("Echo: %s\n", buf);
    }

    // 4. 关闭连接
    close(sock_fd);
    return 0;
}
```

### 三、UDP 通讯步骤（无连接）

UDP 是 “无连接” 协议，无需建立连接，直接发送数据，流程更简单（无监听 / 接受步骤）。

#### 步骤 1：服务器端初始化（仅绑定端口）

UDP 服务器器只需绑定端口，无需监听，直接等待数据。

| 步骤              | 系统调用                          | 作用                                                   |
| ----------------- | --------------------------------- | ------------------------------------------------------ |
| 1. 创建 Socket    | `socket(AF_INET, SOCK_DGRAM, 0);` | 创建 UDP Socket（SOCK_DGRAM 表示数据报）               |
| 2. 绑定 IP 和端口 | `bind()` （同 TCP）               | 绑定本地 IP + 端口（必须绑定，否则客户端不知道发给谁） |

#### 步骤 2：客户端初始化（无需连接）

UDP 客户端无需连接，直接发送数据（需指定服务器的 IP + 端口）。

| 步骤           | 系统调用                          | 作用            |
| -------------- | --------------------------------- | --------------- |
| 1. 创建 Socket | `socket(AF_INET, SOCK_DGRAM, 0);` | 创建 UDP Socket |

#### 步骤 3：数据传输（收发需指定地址）

UDP 无连接，每次收发都需明确对方的 IP + 端口。

| 操作     | 系统调用                                                     | 作用                         |
| -------- | ------------------------------------------------------------ | ---------------------------- |
| 接收数据 | `ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);` | 接收数据，并获取发送方的地址 |
| 发送数据 | `ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);` | 向指定地址发送数据           |

#### 步骤 4：关闭 Socket

同 TCP，用 `close()` 关闭 Socket 即可。

#### UDP 代码示例：简单消息收发

**服务器端（udp_server.c）**：接收客户端消息并打印

c



运行









```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUF_SIZE 1024

int main() {
    // 1. 创建 UDP Socket
    int sock_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock_fd == -1) {
        perror("socket failed");
        exit(1);
    }

    // 2. 绑定端口
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(PORT);

    if (bind(sock_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind failed");
        close(sock_fd);
        exit(1);
    }
    printf("UDP server listening on port %d...\n", PORT);

    // 3. 接收数据
    char buf[BUF_SIZE];
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);

    while (1) {
        // 接收客户端消息，并获取客户端地址
        ssize_t n = recvfrom(sock_fd, buf, BUF_SIZE-1, 0, 
                           (struct sockaddr*)&client_addr, &client_addr_len);
        if (n == -1) {
            perror("recvfrom failed");
            continue;
        }
        buf[n] = '\0';
        printf("Received from %s:%d: %s\n",
               inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port), buf);

        // 可选：回复客户端
        const char* reply = "Message received";
        sendto(sock_fd, reply, strlen(reply), 0,
               (struct sockaddr*)&client_addr, client_addr_len);
    }

    // 4. 关闭 Socket（实际不会执行，需手动终止）
    close(sock_fd);
    return 0;
}
```

**客户端（udp_client.c）**：向服务器发送消息

c



运行









```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define SERVER_IP "127.0.0.1"
#define PORT 8888
#define BUF_SIZE 1024

int main() {
    // 1. 创建 UDP Socket
    int sock_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock_fd == -1) {
        perror("socket failed");
        exit(1);
    }

    // 服务器地址
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    inet_pton(AF_INET, SERVER_IP, &serv_addr.sin_addr);

    // 2. 发送数据
    char buf[BUF_SIZE];
    while (1) {
        printf("Enter message (q to quit): ");
        fgets(buf, BUF_SIZE, stdin);
        if (buf[strlen(buf)-1] == '\n') {
            buf[strlen(buf)-1] = '\0';
        }

        // 发送给服务器
        sendto(sock_fd, buf, strlen(buf), 0,
               (struct sockaddr*)&serv_addr, sizeof(serv_addr));

        if (strcmp(buf, "q") == 0) break;

        // 接收服务器回复
        struct sockaddr_in from_addr;
        socklen_t from_len = sizeof(from_addr);
        ssize_t n = recvfrom(sock_fd, buf, BUF_SIZE-1, 0,
                           (struct sockaddr*)&from_addr, &from_len);
        if (n == -1) {
            perror("recvfrom failed");
            continue;
        }
        buf[n] = '\0';
        printf("Server reply: %s\n", buf);
    }

    // 3. 关闭 Socket
    close(sock_fd);
    return 0;
}
```

### 四、编译与测试

1. **编译代码**：

   bash

   

   

   

   

   

   ```bash
   # TCP 服务器和客户端
   gcc echo_server.c -o echo_server
   gcc echo_client.c -o echo_client
   
   # UDP 服务器和客户端
   gcc udp_server.c -o udp_server
   gcc udp_client.c -o udp_client
   ```

   

2. **运行测试**：

   - 先启动服务器，再启动客户端，输入消息即可看到通讯效果。
   - 用 `netstat -tulnp` 查看端口监听状态，用 `tcpdump -i lo port 8888` 抓包分析数据传输。

### 五、关键注意事项

1. **错误处理**：所有系统调用（`socket()`/`bind()` 等）都可能失败，必须检查返回值并处理（用 `perror()` 打印错误原因）。
2. **字节序**：端口和 IP 必须用 `htons()`/`htonl()` 转换为网络字节序，否则在不同字节序的主机间会通讯失败。
3. **TCP 粘包**：TCP 是字节流，多次 `send()` 可能被合并，需在应用层设计协议（如固定长度头）解决。
4. **UDP 数据报大小**：单次发送的数据不能超过 MTU（通常 1500 字节），否则会被分片，影响效率。

通过以上步骤，你可以实现最基础的网络通讯。后续学习并发模型（多线程 /epoll）后，可扩展为能处理多客户端的服务器。



使用 C++ 进行 Linux 网络编程时，核心仍依赖 Socket API（与 C 语言相同，因为系统调用是 C 接口），但可以利用 C++ 的**面向对象特性**、**标准库（如`std::string`、`std::vector`）** 和**异常处理**，让代码更简洁、安全、易维护。

下面分别实现 TCP 和 UDP 的 C++ 版本网络通讯示例，重点展示 C++ 特性的应用（如类封装、字符串处理、异常处理等）。

### 一、TCP 通讯：C++ 封装实现（回声服务器与客户端）

#### 核心思路：

用`TcpServer`和`TcpClient`类封装 Socket 操作，通过**构造函数初始化 Socket**，**析构函数自动释放资源**（避免泄漏），用`std::string`处理数据（替代 C 的`char*`），用异常`std::exception`处理错误（替代`perror`+`exit`）。

##### 1. TCP 服务器（`TcpServer.cpp`）

cpp



运行









```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <stdexcept>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// TCP服务器类
class TcpServer {
private:
    int listen_fd;       // 监听Socket
    int conn_fd;         // 与客户端通信的Socket
    sockaddr_in serv_addr; // 服务器地址
    sockaddr_in client_addr; // 客户端地址
    socklen_t client_len;

public:
    // 构造函数：初始化Socket并绑定端口
    TcpServer(uint16_t port, const std::string& ip = "0.0.0.0") {
        // 1. 创建TCP Socket
        listen_fd = socket(AF_INET, SOCK_STREAM, 0);
        if (listen_fd == -1) {
            throw std::runtime_error("socket创建失败: " + std::string(strerror(errno)));
        }

        // 2. 初始化服务器地址
        std::memset(&serv_addr, 0, sizeof(serv_addr));
        serv_addr.sin_family = AF_INET;
        serv_addr.sin_port = htons(port); // 端口转换为网络字节序
        // IP地址转换（字符串→网络字节序）
        if (inet_pton(AF_INET, ip.c_str(), &serv_addr.sin_addr) <= 0) {
            close(listen_fd);
            throw std::runtime_error("IP地址无效: " + std::string(strerror(errno)));
        }

        // 3. 绑定地址
        if (bind(listen_fd, (sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
            close(listen_fd);
            throw std::runtime_error("bind失败: " + std::string(strerror(errno)));
        }

        // 4. 监听连接（最大等待队列5）
        if (listen(listen_fd, 5) == -1) {
            close(listen_fd);
            throw std::runtime_error("listen失败: " + std::string(strerror(errno)));
        }

        std::cout << "服务器启动成功，监听 " << ip << ":" << port << std::endl;
        client_len = sizeof(client_addr);
        conn_fd = -1; // 初始未连接
    }

    // 接受客户端连接（阻塞）
    void acceptClient() {
        conn_fd = accept(listen_fd, (sockaddr*)&client_addr, &client_len);
        if (conn_fd == -1) {
            throw std::runtime_error("accept失败: " + std::string(strerror(errno)));
        }
        // 打印客户端信息（网络字节序→本地字节序）
        char client_ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client_addr.sin_addr, client_ip, INET_ADDRSTRLEN);
        std::cout << "客户端连接: " << client_ip << ":" << ntohs(client_addr.sin_port) << std::endl;
    }

    // 接收数据（返回收到的字符串）
    std::string recvData(size_t buf_size = 1024) {
        if (conn_fd == -1) {
            throw std::runtime_error("未连接客户端，无法接收数据");
        }
        std::vector<char> buf(buf_size); // 用vector管理缓冲区（自动释放）
        ssize_t n = recv(conn_fd, buf.data(), buf_size - 1, 0);
        if (n <= 0) {
            if (n == 0) {
                throw std::runtime_error("客户端已断开连接");
            } else {
                throw std::runtime_error("recv失败: " + std::string(strerror(errno)));
            }
        }
        return std::string(buf.data(), n); // 转换为string返回
    }

    // 发送数据
    void sendData(const std::string& data) {
        if (conn_fd == -1) {
            throw std::runtime_error("未连接客户端，无法发送数据");
        }
        ssize_t n = send(conn_fd, data.c_str(), data.size(), 0);
        if (n == -1) {
            throw std::runtime_error("send失败: " + std::string(strerror(errno)));
        }
    }

    // 析构函数：关闭Socket（自动释放资源）
    ~TcpServer() {
        if (conn_fd != -1) close(conn_fd);
        if (listen_fd != -1) close(listen_fd);
        std::cout << "服务器资源已释放" << std::endl;
    }
};

// 服务器主函数
int main() {
    try {
        TcpServer server(8888); // 监听8888端口
        server.acceptClient();  // 等待客户端连接

        // 循环收发数据
        while (true) {
            std::string data = server.recvData();
            std::cout << "收到客户端: " << data << std::endl;

            if (data == "q") { // 退出条件
                std::cout << "客户端请求断开，服务器退出" << std::endl;
                break;
            }

            server.sendData("服务器收到: " + data); // 回声+前缀
        }
    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }
    return 0;
}
```

##### 2. TCP 客户端（`TcpClient.cpp`）

```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <stdexcept>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// TCP客户端类
class TcpClient {
private:
    int sock_fd;               // 客户端Socket
    sockaddr_in serv_addr;     // 服务器地址

public:
    // 构造函数：创建Socket并连接服务器
    TcpClient(const std::string& server_ip, uint16_t server_port) {
        // 1. 创建TCP Socket
        sock_fd = socket(AF_INET, SOCK_STREAM, 0);
        if (sock_fd == -1) {
            throw std::runtime_error("socket创建失败: " + std::string(strerror(errno)));
        }

        // 2. 初始化服务器地址
        std::memset(&serv_addr, 0, sizeof(serv_addr));
        serv_addr.sin_family = AF_INET;
        serv_addr.sin_port = htons(server_port);
        if (inet_pton(AF_INET, server_ip.c_str(), &serv_addr.sin_addr) <= 0) {
            close(sock_fd);
            throw std::runtime_error("服务器IP无效: " + std::string(strerror(errno)));
        }

        // 3. 连接服务器
        if (connect(sock_fd, (sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
            close(sock_fd);
            throw std::runtime_error("连接服务器失败: " + std::string(strerror(errno)));
        }

        std::cout << "成功连接服务器 " << server_ip << ":" << server_port << std::endl;
    }

    // 发送数据
    void sendData(const std::string& data) {
        ssize_t n = send(sock_fd, data.c_str(), data.size(), 0);
        if (n == -1) {
            throw std::runtime_error("send失败: " + std::string(strerror(errno)));
        }
    }

    // 接收数据
    std::string recvData(size_t buf_size = 1024) {
        std::vector<char> buf(buf_size);
        ssize_t n = recv(sock_fd, buf.data(), buf_size - 1, 0);
        if (n <= 0) {
            if (n == 0) {
                throw std::runtime_error("服务器已断开连接");
            } else {
                throw std::runtime_error("recv失败: " + std::string(strerror(errno)));
            }
        }
        return std::string(buf.data(), n);
    }

    // 析构函数：关闭Socket
    ~TcpClient() {
        if (sock_fd != -1) close(sock_fd);
        std::cout << "客户端资源已释放" << std::endl;
    }
};

// 客户端主函数
int main() {
    try {
        TcpClient client("127.0.0.1", 8888); // 连接本地8888端口

        // 循环发送数据
        std::string input;
        while (true) {
            std::cout << "请输入消息（输入q退出）: ";
            std::getline(std::cin, input); // 用getline读取整行（含空格）

            client.sendData(input);

            if (input == "q") {
                std::cout << "客户端退出" << std::endl;
                break;
            }

            std::string reply = client.recvData();
            std::cout << "服务器回复: " << reply << std::endl;
        }
    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }
    return 0;
}
```

### 二、UDP 通讯：C++ 封装实现（消息收发）

#### 核心思路：

UDP 无需连接，用`UdpServer`和`UdpClient`类封装，重点处理 “发送时指定目标地址” 和 “接收时获取源地址”，同样用 C++ 的字符串和异常处理。

##### 1. UDP 服务器（`UdpServer.cpp`）

cpp



运行









```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <stdexcept>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// UDP服务器类
class UdpServer {
private:
    int sock_fd;
    sockaddr_in serv_addr;
    sockaddr_in client_addr;
    socklen_t client_len;

public:
    // 构造函数：创建Socket并绑定端口
    UdpServer(uint16_t port, const std::string& ip = "0.0.0.0") {
        sock_fd = socket(AF_INET, SOCK_DGRAM, 0); // SOCK_DGRAM表示UDP
        if (sock_fd == -1) {
            throw std::runtime_error("socket创建失败: " + std::string(strerror(errno)));
        }

        std::memset(&serv_addr, 0, sizeof(serv_addr));
        serv_addr.sin_family = AF_INET;
        serv_addr.sin_port = htons(port);
        if (inet_pton(AF_INET, ip.c_str(), &serv_addr.sin_addr) <= 0) {
            close(sock_fd);
            throw std::runtime_error("IP无效: " + std::string(strerror(errno)));
        }

        if (bind(sock_fd, (sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
            close(sock_fd);
            throw std::runtime_error("bind失败: " + std::string(strerror(errno)));
        }

        std::cout << "UDP服务器启动，监听 " << ip << ":" << port << std::endl;
        client_len = sizeof(client_addr);
    }

    // 接收数据（同时获取客户端地址）
    std::string recvFrom(std::string& client_ip, uint16_t& client_port) {
        std::vector<char> buf(1024);
        ssize_t n = recvfrom(sock_fd, buf.data(), buf.size() - 1, 0,
                           (sockaddr*)&client_addr, &client_len);
        if (n == -1) {
            throw std::runtime_error("recvfrom失败: " + std::string(strerror(errno)));
        }

        // 解析客户端IP和端口（网络字节序→本地）
        char ip_str[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client_addr.sin_addr, ip_str, INET_ADDRSTRLEN);
        client_ip = ip_str;
        client_port = ntohs(client_addr.sin_port);

        return std::string(buf.data(), n);
    }

    // 发送数据到指定客户端
    void sendTo(const std::string& data, const std::string& client_ip, uint16_t client_port) {
        sockaddr_in dest_addr;
        std::memset(&dest_addr, 0, sizeof(dest_addr));
        dest_addr.sin_family = AF_INET;
        dest_addr.sin_port = htons(client_port);
        if (inet_pton(AF_INET, client_ip.c_str(), &dest_addr.sin_addr) <= 0) {
            throw std::runtime_error("客户端IP无效: " + std::string(strerror(errno)));
        }

        ssize_t n = sendto(sock_fd, data.c_str(), data.size(), 0,
                         (sockaddr*)&dest_addr, sizeof(dest_addr));
        if (n == -1) {
            throw std::runtime_error("sendto失败: " + std::string(strerror(errno)));
        }
    }

    ~UdpServer() {
        close(sock_fd);
        std::cout << "UDP服务器资源已释放" << std::endl;
    }
};

int main() {
    try {
        UdpServer server(8888);

        while (true) {
            std::string client_ip;
            uint16_t client_port;
            std::string data = server.recvFrom(client_ip, client_port);
            std::cout << "收到 " << client_ip << ":" << client_port << " 的消息: " << data << std::endl;

            if (data == "q") {
                server.sendTo("已收到退出请求", client_ip, client_port);
                break;
            }

            server.sendTo("服务器已收到: " + data, client_ip, client_port);
        }
    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }
    return 0;
}
```

##### 2. UDP 客户端（`UdpClient.cpp`）

cpp



运行









```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <stdexcept>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// UDP客户端类
class UdpClient {
private:
    int sock_fd;
    sockaddr_in serv_addr;
    socklen_t serv_len;

public:
    UdpClient(const std::string& server_ip, uint16_t server_port) {
        sock_fd = socket(AF_INET, SOCK_DGRAM, 0);
        if (sock_fd == -1) {
            throw std::runtime_error("socket创建失败: " + std::string(strerror(errno)));
        }

        std::memset(&serv_addr, 0, sizeof(serv_addr));
        serv_addr.sin_family = AF_INET;
        serv_addr.sin_port = htons(server_port);
        if (inet_pton(AF_INET, server_ip.c_str(), &serv_addr.sin_addr) <= 0) {
            close(sock_fd);
            throw std::runtime_error("服务器IP无效: " + std::string(strerror(errno)));
        }

        serv_len = sizeof(serv_addr);
        std::cout << "UDP客户端初始化完成，目标服务器 " << server_ip << ":" << server_port << std::endl;
    }

    // 发送数据到服务器
    void sendToServer(const std::string& data) {
        ssize_t n = sendto(sock_fd, data.c_str(), data.size(), 0,
                         (sockaddr*)&serv_addr, serv_len);
        if (n == -1) {
            throw std::runtime_error("sendto失败: " + std::string(strerror(errno)));
        }
    }

    // 从服务器接收数据
    std::string recvFromServer() {
        std::vector<char> buf(1024);
        ssize_t n = recvfrom(sock_fd, buf.data(), buf.size() - 1, 0,
                           nullptr, nullptr); // 忽略源地址（已知是服务器）
        if (n == -1) {
            throw std::runtime_error("recvfrom失败: " + std::string(strerror(errno)));
        }
        return std::string(buf.data(), n);
    }

    ~UdpClient() {
        close(sock_fd);
        std::cout << "UDP客户端资源已释放" << std::endl;
    }
};

int main() {
    try {
        UdpClient client("127.0.0.1", 8888);

        std::string input;
        while (true) {
            std::cout << "请输入消息（输入q退出）: ";
            std::getline(std::cin, input);

            client.sendToServer(input);

            if (input == "q") {
                std::cout << "客户端退出" << std::endl;
                break;
            }

            std::string reply = client.recvFromServer();
            std::cout << "服务器回复: " << reply << std::endl;
        }
    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }
    return 0;
}
```

### 三、编译与运行

1. **编译命令**（需用`g++`，支持 C++11 及以上）：

   bash

   

   

   

   

   

   ```bash
   # TCP
   g++ TcpServer.cpp -o tcp_server -std=c++11
   g++ TcpClient.cpp -o tcp_client -std=c++11
   
   # UDP
   g++ UdpServer.cpp -o udp_server -std=c++11
   g++ UdpClient.cpp -o udp_client -std=c++11
   ```

   

2. **运行步骤**：

   - 先启动服务器（`./tcp_server` 或 `./udp_server`），再启动客户端（`./tcp_client` 或 `./udp_client`）。
   - 在客户端输入消息，观察服务器和客户端的交互效果。

### 四、C++ 相比 C 的优势

1. **资源自动管理**：通过类的析构函数自动关闭 Socket，避免 C 中忘记`close()`导致的文件描述符泄漏。
2. **安全的字符串处理**：用`std::string`替代`char*`，避免缓冲区溢出（无需手动计算长度）。
3. **异常处理**：用`try-catch`统一处理错误，代码逻辑更清晰（C 中需重复检查返回值）。
4. **面向对象封装**：将 Socket 操作封装到类中，便于复用和扩展（如后续添加并发功能）。

通过以上示例，你可以快速掌握 C++ 网络编程的核心模式。后续学习并发（如多线程、`epoll`）时，可基于这些类进一步扩展（例如在`TcpServer`中添加线程池处理多客户

```c++
class Socket{
        private:
                int m_sockfd;
                struct sockaddr_in m_addr;
        public:
                Socket();

                Socket(int );

                ~Socket();

                bool bind(const string &,int);

                bool connect(const string &,int)

                bool listen(int);

                int accept();

                int send(const char *,int);

                int recv(char *,int);

                void close();



}
```



```
src/socket.cpp: In constructor ‘Socket::Socket()’:
src/socket.cpp:11:34: error: expected ‘;’ before ‘std’
   11 |  memset(&m_addr,0,sizeof(m_addr))
      |                                  ^
      |                                  ;
   12 |  std::cout<<"socker created!"<<std::endl;
      |  ~~~                              
src/socket.cpp: In constructor ‘Socket::Socket(int)’:
src/socket.cpp:18:34: error: expected ‘;’ before ‘}’ token
   18 |  memset(&m_addr,0,sizeof(m_addr))
      |                                  ^
      |                                  ;
   19 | }
      | ~                                 
src/socket.cpp: At global scope:
src/socket.cpp:25:1: error: ISO C++ forbids declaration of ‘bind’ with no type [-fpermissive]
   25 | Socket::bind(const std::string &ip,int port){
      | ^~~~~~
src/socket.cpp:25:1: error: no declaration matches ‘int Socket::bind(const string&, int)’
In file included from src/socket.cpp:1:
include/socket.h:21:8: note: candidate is: ‘bool Socket::bind(const string&, int)’
   21 |   bool bind(const std::string &,int);
      |        ^~~~
include/socket.h:10:7: note: ‘class Socket’ defined here
   10 | class Socket{
      |       ^~~~~~
src/socket.cpp:42:1: error: ISO C++ forbids declaration of ‘listen’ with no type [-fpermissive]
   42 | Socket::listen(int backlog){
      | ^~~~~~
src/socket.cpp:42:1: error: no declaration matches ‘int Socket::listen(int)’
In file included from src/socket.cpp:1:
include/socket.h:25:8: note: candidate is: ‘bool Socket::listen(int)’
   25 |   bool listen(int);
      |        ^~~~~~
include/socket.h:10:7: note: ‘class Socket’ defined here
   10 | class Socket{
      |       ^~~~~~
src/socket.cpp:53:1: error: ISO C++ forbids declaration of ‘accept’ with no type [-fpermissive]
   53 | Socket::accept(){
      | ^~~~~~
src/socket.cpp: In member function ‘int Socket::accept()’:
src/socket.cpp:62:37: error: invalid conversion from ‘in_addr_t’ {aka ‘unsigned int’} to ‘const void*’ [-fpermissive]
   62 |   inet_ntop(AF_INET,c_addr.sin_addr.s_addr,&ip);
      |                     ~~~~~~~~~~~~~~~~^~~~~~
      |                                     |
      |                                     in_addr_t {aka unsigned int}
src/socket.cpp:62:44: error: cannot convert ‘std::string*’ {aka ‘std::__cxx11::basic_string<char>*’} to ‘char*’
   62 |   inet_ntop(AF_INET,c_addr.sin_addr.s_addr,&ip);
      |                                            ^~~
      |                                            |
      |                                            std::string* {aka std::__cxx11::basic_string<char>*}
In file included from include/socket.h:6,
                 from src/socket.cpp:1:
/usr/include/arpa/inet.h:65:27: note:   initializing argument 3 of ‘const char* inet_ntop(int, const void*, char*, socklen_t)’
   65 |          char *__restrict __buf, socklen_t __len)
      |          ~~~~~~~~~~~~~~~~~^~~~~
src/socket.cpp: At global scope:
src/socket.cpp:68:1: error: ISO C++ forbids declaration of ‘connect’ with no type [-fpermissive]
   68 | Socket::connect(const std::string &ip,int port){
      | ^~~~~~
src/socket.cpp:68:1: error: no declaration matches ‘int Socket::connect(const string&, int)’
In file included from src/socket.cpp:1:
include/socket.h:23:8: note: candidate is: ‘bool Socket::connect(const string&, int)’
   23 |   bool connect(const std::string &,int);
      |        ^~~~~~~
include/socket.h:10:7: note: ‘class Socket’ defined here
   10 | class Socket{
      |       ^~~~~~
make: *** [Makefile:21: build/socket.o] Error 1
```

