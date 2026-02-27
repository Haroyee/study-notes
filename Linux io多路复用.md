在 Linux 网络编程中，select、poll 和 epoll 是三种经典的 **IO 多路复用模型**，核心作用是通过一个进程 / 线程同时管理多个文件描述符（如套接字），仅在有 IO 事件（可读 / 可写）时才处理，避免阻塞在单个 IO 操作上。三者的设计思路相似，但性能和适用场景差异显著。

### 一、select 模型

select 是最早的 IO 多路复用机制，跨平台（支持 Linux、Windows 等），但存在明显的性能限制。

#### 核心原理

1. **事件集合**：通过三个文件描述符集合（readfds、writefds、exceptfds）分别监控 “可读”“可写”“异常” 事件。

1. **阻塞等待**：调用 select() 后，内核阻塞等待，直到有事件就绪或超时，返回就绪的文件描述符数量。

1. **遍历检查**：返回后需遍历所有文件描述符，通过 FD_ISSET() 检查哪些 fd 就绪。

#### 关键限制

- **文件描述符上限**：受 FD_SETSIZE 限制（默认 1024），无法直接支持高并发。

- **效率低**：每次调用需将整个 fd 集合从用户态复制到内核态；返回后需遍历所有 fd 检查就绪状态，耗时随 fd 数量增加而增长。

#### 核心函数

| 函数原型                                                     | 作用                     | 关键参数                                                     |
| ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------------ |
| int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout) | 阻塞等待事件就绪         | - nfds：需监控的最大 fd + 1（必须设置，否则无效）- readfds/writefds/exceptfds：关注的事件集合（输入输出参数，需每次重置）<br>- timeout：超时时间（NULL` 表示永久阻塞）返回值：就绪事件数（0 表示超时，-1 表示错误） |
| FD_ZERO(fd_set *set)                                         | 清空 fd 集合             | -                                                            |
| FD_SET(int fd, fd_set *set)                                  | 将 fd 添加到集合         | -                                                            |
| FD_CLR(int fd, fd_set *set)                                  | 从集合中移除 fd          | -                                                            |
| FD_ISSET(int fd, fd_set *set)                                | 检查 fd 是否在就绪集合中 | 返回非 0 表示就绪                                            |

#### 示例代码（select 处理多客户端）

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/select.h>

#define MAX_FD 1024 // 受 FD_SETSIZE 限制
#define PORT 8083

int main() {
    // 1. 创建监听套接字
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) { perror("socket"); exit(1); }

    // 2. 绑定地址
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    if (bind(listen_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind"); exit(1);
    }

    // 3. 监听连接
    if (listen(listen_fd, 10) == -1) { perror("listen"); exit(1); }

    fd_set read_fds;    // 关注的可读事件集合
    fd_set all_fds;     // 保存所有需监控的 fd（避免每次重新添加）
    int max_fd = listen_fd; // 记录最大 fd（供 select 的 nfds 参数）

    FD_ZERO(&all_fds);
    FD_SET(listen_fd, &all_fds); // 先添加监听 fd

    printf("select server listening on port %d...\n", PORT);

    while (1) {
        // 每次调用 select 前需重置 read_fds（因为 select 会修改集合）
        read_fds = all_fds;

        // 阻塞等待可读事件（无超时）
        int nfds = select(max_fd + 1, &read_fds, NULL, NULL, NULL);
        if (nfds == -1) { perror("select"); exit(1); }

        // 遍历所有 fd，检查哪些就绪
        for (int fd = 0; fd <= max_fd; fd++) {
            if (!FD_ISSET(fd, &read_fds)) { continue; }

            if (fd == listen_fd) { // 监听 fd 就绪：新连接
                struct sockaddr_in client_addr;
                socklen_t client_len = sizeof(client_addr);
                int client_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_len);
                if (client_fd == -1) { perror("accept"); continue; }

                // 将新客户端 fd 添加到监控集合
                FD_SET(client_fd, &all_fds);
                if (client_fd > max_fd) { max_fd = client_fd; } // 更新最大 fd
                printf("new client: %d\n", client_fd);

            } else { // 客户端 fd 就绪：有数据可读
                char buf[1024];
                ssize_t n = read(fd, buf, sizeof(buf)-1);
                if (n <= 0) { // 客户端断开或错误
                    printf("client %d disconnected\n", fd);
                    FD_CLR(fd, &all_fds); // 从集合中移除
                    close(fd);
                } else { // 处理数据
                    buf[n] = '\0';
                    printf("client %d: %s", fd, buf);
                    // 回复客户端
                    const char* resp = "received: ";
                    write(fd, resp, strlen(resp));
                    write(fd, buf, n);
                }
            }
        }
    }

    close(listen_fd);
    return 0;
}
```

### 二、poll 模型

poll 是对 select 的改进，解决了 select 的文件描述符上限问题，但核心效率问题（用户态与内核态复制、遍历检查）仍存在。

#### 核心原理

1. **事件数组**：通过 struct pollfd 结构体数组管理事件，每个元素包含一个 fd 和关注的事件（无需区分读 / 写 / 异常集合）。

1. **阻塞等待**：调用 poll() 后，内核阻塞等待事件就绪，返回就绪的 fd 数量。

1. **遍历检查**：返回后遍历数组，通过 revents 字段检查哪些 fd 就绪（revents 是内核填充的实际发生的事件）。

#### 改进与局限

- **无 fd 上限**：仅受系统资源限制（理论上支持更多连接）。

- **无需重置集合**：pollfd 数组可重复使用，无需像 select 那样每次清空重置。

- **仍有开销**：每次调用需复制整个数组到内核态；返回后仍需遍历所有元素检查就绪状态。

#### 核心函数与结构体

| 结构体 / 函数                                          | 说明                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| struct pollfd { int fd; short events; short revents; } | - fd：要监控的文件描述符- events：关注的事件（如 POLLIN 可读、POLLOUT 可写）- revents：内核返回的实际发生的事件（输出参数） |
| int poll(struct pollfd *fds, nfds_t nfds, int timeout) | 阻塞等待事件就绪- fds：pollfd 数组- nfds：数组长度- timeout：超时时间（毫秒，-1 表示永久阻塞）返回值：就绪事件数（0 表示超时，-1 表示错误） |

#### 示例代码（poll 处理多客户端）

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <poll.h>

#define MAX_CONN 1024 // 可自定义（不受固定上限限制）
#define PORT 8084

int main() {
    // 1. 创建监听套接字
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) { perror("socket"); exit(1); }

    // 2. 绑定地址
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    if (bind(listen_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind"); exit(1);
    }

    // 3. 监听连接
    if (listen(listen_fd, 10) == -1) { perror("listen"); exit(1); }

    // 初始化 pollfd 数组（第一个元素为监听 fd）
    struct pollfd fds[MAX_CONN];
    memset(fds, 0, sizeof(fds));
    fds[0].fd = listen_fd;
    fds[0].events = POLLIN; // 关注可读事件
    int nfds = 1; // 当前数组中有效元素数量

    printf("poll server listening on port %d...\n", PORT);

    while (1) {
        // 阻塞等待事件（无超时）
        int ready = poll(fds, nfds, -1);
        if (ready == -1) { perror("poll"); exit(1); }

        // 遍历 pollfd 数组检查就绪事件
        for (int i = 0; i < nfds; i++) {
            if (!(fds[i].revents & POLLIN)) { continue; } // 非可读事件跳过

            if (fds[i].fd == listen_fd) { // 监听 fd 就绪：新连接
                struct sockaddr_in client_addr;
                socklen_t client_len = sizeof(client_addr);
                int client_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_len);
                if (client_fd == -1) { perror("accept"); continue; }

                // 检查连接数是否超过上限
                if (nfds >= MAX_CONN) {
                    printf("too many connections, close %d\n", client_fd);
                    close(client_fd);
                    continue;
                }

                // 添加新客户端 fd 到 pollfd 数组
                fds[nfds].fd = client_fd;
                fds[nfds].events = POLLIN; // 关注可读事件
                nfds++;
                printf("new client: %d (total: %d)\n", client_fd, nfds-1);

            } else { // 客户端 fd 就绪：有数据可读
                int client_fd = fds[i].fd;
                char buf[1024];
                ssize_t n = read(client_fd, buf, sizeof(buf)-1);
                if (n <= 0) { // 客户端断开或错误
                    printf("client %d disconnected\n", client_fd);
                    // 移除该 fd（用最后一个元素覆盖，减少数组移动）
                    fds[i] = fds[nfds - 1];
                    nfds--;
                    i--; // 回退索引，避免跳过元素
                    close(client_fd);
                } else { // 处理数据
                    buf[n] = '\0';
                    printf("client %d: %s", client_fd, buf);
                    // 回复客户端
                    const char* resp = "received: ";
                    write(client_fd, resp, strlen(resp));
                    write(client_fd, buf, n);
                }
            }
        }
    }

    close(listen_fd);
    return 0;
}
```

### 三、epoll 模型（Linux 特有）

epoll 是 Linux 为高并发场景设计的 IO 多路复用机制，性能远超 select 和 poll，是解决 “C10K 问题” 的核心技术。

#### 核心原理

1. **事件表**：内核维护一个 “事件表”（通过 epoll_create() 创建），用于存储用户关注的 fd 和事件。

1. **事件注册**：通过 epoll_ctl() 向事件表添加 / 修改 / 删除 fd 及关注的事件（一次注册，多次使用）。

1. **就绪列表**：内核主动将就绪的事件放入 “就绪列表”，epoll_wait() 直接返回就绪列表中的事件，无需遍历所有 fd。

1. **高效机制**：通过内存映射（mmap）避免用户态与内核态的数据复制，仅处理就绪事件，效率极高。

#### 关键模式

- **水平触发（LT，默认）**：只要 fd 缓冲区有未处理数据，epoll_wait() 就持续通知（适合新手，编程简单）。

- **边缘触发（ET）**：仅在 fd 状态变化时通知一次（如从不可读到可读），需一次性处理完所有数据（配合非阻塞 IO，效率更高，编程复杂）。

#### 核心函数

| 函数原型                                                     | 作用                      | 关键参数                                                     |
| ------------------------------------------------------------ | ------------------------- | ------------------------------------------------------------ |
| int epoll_create(int size)                                   | 创建 epoll 实例（事件表） | size：早期需 >0（现在忽略），返回 epoll 描述符（epfd）       |
| int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event) | 管理事件表                | - op：EPOLL_CTL_ADD（添加）、EPOLL_CTL_MOD（修改）、EPOLL_CTL_DEL（删除）- event：struct epoll_event（events 为关注的事件，data.fd 为关联的 fd） |
| int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout) | 等待就绪事件              | - events：输出就绪事件的数组- maxevents：数组最大长度- timeout：超时时间（毫秒，-1 表示永久阻塞）返回值：就绪事件数（0 表示超时，-1 表示错误） |
| struct epoll_event { uint32_t events; epoll_data_t data; }   | 事件结构体                | - events：关注的事件（如 EPOLLIN 可读、EPOLLOUT 可写）- data：关联的数据（通常用 data.fd 存储 fd） |

#### 示例代码（epoll LT 模式）

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/epoll.h>

#define MAX_EVENTS 1024 // 每次最多处理的就绪事件数
#define PORT 8085

int main() {
    // 1. 创建监听套接字
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) { perror("socket"); exit(1); }

    // 2. 绑定地址
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    if (bind(listen_fd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
        perror("bind"); exit(1);
    }

    // 3. 监听连接
    if (listen(listen_fd, 10) == -1) { perror("listen"); exit(1); }

    // 4. 创建 epoll 实例
    int epfd = epoll_create(1); // 参数忽略
    if (epfd == -1) { perror("epoll_create"); exit(1); }

    // 5. 向 epoll 注册监听 fd（关注可读事件，LT 模式默认）
    struct epoll_event ev;
    ev.events = EPOLLIN; // 水平触发：有数据就通知
    ev.data.fd = listen_fd;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, listen_fd, &ev) == -1) {
        perror("epoll_ctl add listen_fd"); exit(1);
    }

    struct epoll_event events[MAX_EVENTS]; // 存放就绪事件
    printf("epoll (LT) server listening on port %d...\n", PORT);

    while (1) {
        // 阻塞等待就绪事件（最多返回 MAX_EVENTS 个）
        int nfds = epoll_wait(epfd, events, MAX_EVENTS, -1);
        if (nfds == -1) { perror("epoll_wait"); exit(1); }

        // 仅遍历就绪的事件（无需检查所有 fd）
        for (int i = 0; i < nfds; i++) {
            int fd = events[i].data.fd;

            if (fd == listen_fd) { // 监听 fd 就绪：新连接
                struct sockaddr_in client_addr;
                socklen_t client_len = sizeof(client_addr);
                int client_fd = accept(listen_fd, (struct sockaddr*)&client_addr, &client_len);
                if (client_fd == -1) { perror("accept"); continue; }

                // 向 epoll 注册客户端 fd（关注可读事件）
                ev.events = EPOLLIN;
                ev.data.fd = client_fd;
                if (epoll_ctl(epfd, EPOLL_CTL_ADD, client_fd, &ev) == -1) {
                    perror("epoll_ctl add client_fd");
                    close(client_fd);
                }
                printf("new client: %d\n", client_fd);

            } else { // 客户端 fd 就绪：有数据可读
                char buf[1024];
                ssize_t n = read(fd, buf, sizeof(buf)-1);
                if (n <= 0) { // 客户端断开或错误
                    printf("client %d disconnected\n", fd);
                    epoll_ctl(epfd, EPOLL_CTL_DEL, fd, NULL); // 从 epoll 移除
                    close(fd);
                } else { // 处理数据（LT 模式可分多次读）
                    buf[n] = '\0';
                    printf("client %d: %s", fd, buf);
                    // 回复客户端
                    const char* resp = "received: ";
                    write(fd, resp, strlen(resp));
                    write(fd, buf, n);
                }
            }
        }
    }

    close(listen_fd);
    close(epfd);
    return 0;
}
```

### 三者对比与适用场景

| 特性     | select                  | poll                   | epoll（Linux）       |
| -------- | ----------------------- | ---------------------- | -------------------- |
| fd 上限  | 固定（FD_SETSIZE=1024） | 无（受系统资源限制）   | 无（受系统资源限制） |
| 事件传递 | 需复制整个集合到内核态  | 需复制整个数组到内核态 | 内存映射（无复制）   |
| 就绪检查 | 遍历所有 fd             | 遍历所有 pollfd        | 直接返回就绪列表     |
| 效率     | 低（O (n)）             | 中（O (n)）            | 高（O (1)）          |
| 跨平台   | 支持（Linux/Windows）   | 支持（Linux/UNIX）     | 仅 Linux             |
| 适用场景 | 低并发、跨平台          | 中低并发、无 fd 上限   | 高并发（C10K+）      |

总结：高并发场景优先使用 epoll；跨平台或低并发场景可考虑 select 或 poll；epoll 的 LT 模式编程简单，ET 模式需配合非阻塞 IO 但效率更高。