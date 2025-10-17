# C++ 信号处理详解

信号处理是 C++ 中与操作系统交互的重要机制，它允许程序响应外部事件（如用户中断、程序错误等）。C++ 继承了 C 语言的信号处理机制，并提供了 `<csignal>` 头文件来支持信号处理功能。

## 一、信号处理的基本概念

### 1. 什么是信号？

信号是操作系统或其它进程发送给程序的中断通知，用于通知程序发生了某种事件。常见的信号包括：

- **SIGINT**：中断信号（通常由 Ctrl+C 产生）
- **SIGSEGV**：段错误（无效内存访问）
- **SIGFPE**：算术运算错误（如除以零）
- **SIGTERM**：终止信号
- **SIGABRT**：异常终止信号

### 2. 信号处理的方式

程序可以以三种方式响应信号：
1. 忽略信号
2. 执行默认操作
3. 捕获信号并执行自定义处理函数

## 二、基本的信号处理函数

### 1. `signal()` 函数

`signal()` 函数用于注册信号处理程序：

```cpp
#include <iostream>
#include <csignal>
#include <cstdlib>

// 信号处理函数
void signalHandler(int signalNum) {
    std::cout << "中断信号 (" << signalNum << ") 收到.\n";
    
    // 清理并退出
    exit(signalNum);
}

int main() {
    // 注册信号 SIGINT 和信号处理函数
    signal(SIGINT, signalHandler);
    
    std::cout << "按下 Ctrl+C 来中断程序...\n";
    
    // 无限循环
    while(1) {
        // 等待信号
    }
    
    return 0;
}
```

### 2. `raise()` 函数

`raise()` 函数用于向程序自身发送信号：

```cpp
#include <iostream>
#include <csignal>

void signalHandler(int signalNum) {
    std::cout << "信号 (" << signalNum << ") 收到.\n";
}

int main() {
    // 注册信号处理函数
    signal(SIGABRT, signalHandler);
    
    std::cout << "准备发送中断信号...\n";
    
    // 向自身发送 SIGABRT 信号
    raise(SIGABRT);
    
    std::cout << "程序继续执行...\n";
    
    return 0;
}
```

## 三、高级信号处理技巧

### 1. 处理多个信号

```cpp
#include <iostream>
#include <csignal>
#include <cstdlib>
#include <unistd.h> // 用于 sleep()

// 信号处理函数
void signalHandler(int signalNum) {
    switch(signalNum) {
        case SIGINT:
            std::cout << "收到中断信号 (SIGINT)\n";
            break;
        case SIGTERM:
            std::cout << "收到终止信号 (SIGTERM)\n";
            break;
        case SIGHUP:
            std::cout << "收到挂起信号 (SIGHUP)\n";
            break;
        default:
            std::cout << "收到未知信号: " << signalNum << "\n";
    }
    
    // 如果是中断信号，退出程序
    if (signalNum == SIGINT) {
        std::cout << "正在退出程序...\n";
        exit(0);
    }
}

int main() {
    // 注册多个信号处理函数
    signal(SIGINT, signalHandler);
    signal(SIGTERM, signalHandler);
    signal(SIGHUP, signalHandler);
    
    std::cout << "程序已启动，PID: " << getpid() << "\n";
    std::cout << "发送信号: kill -SIGTERM " << getpid() << " 或按 Ctrl+C\n";
    
    // 保持程序运行
    while(true) {
        sleep(1);
    }
    
    return 0;
}
```

### 2. 恢复默认信号处理

```cpp
#include <iostream>
#include <csignal>

void signalHandler(int signalNum) {
    std::cout << "信号 (" << signalNum << ") 收到.\n";
    
    // 恢复默认信号处理
    signal(SIGINT, SIG_DFL);
    std::cout << "已恢复默认信号处理，再次按下 Ctrl+C 将终止程序.\n";
}

int main() {
    // 注册自定义信号处理函数
    signal(SIGINT, signalHandler);
    
    std::cout << "按下 Ctrl+C 第一次会显示消息，第二次会终止程序...\n";
    
    // 无限循环
    while(1) {}
    
    return 0;
}
```

### 3. 忽略信号

```cpp
#include <iostream>
#include <csignal>

int main() {
    // 忽略 SIGINT 信号
    signal(SIGINT, SIG_IGN);
    
    std::cout << "已忽略 SIGINT 信号，按下 Ctrl+C 将无效...\n";
    
    // 无限循环
    while(1) {}
    
    return 0;
}
```

## 四、信号处理的最佳实践

### 1. 信号安全的编程

信号处理函数应该尽可能简单，避免使用非异步信号安全的函数：

```cpp
#include <iostream>
#include <csignal>
#include <cstdlib>
#include <unistd.h>
#include <atomic>

// 使用原子标志进行信号安全的通信
std::atomic<bool> signalReceived(false);

void signalHandler(int signalNum) {
    // 只设置标志，不做复杂操作
    signalReceived = true;
}

int main() {
    signal(SIGINT, signalHandler);
    
    std::cout << "按下 Ctrl+C 来测试信号安全处理...\n";
    
    while(!signalReceived) {
        // 正常程序逻辑
        std::cout << "程序运行中...\n";
        sleep(1);
    }
    
    std::cout << "收到信号，正在清理资源...\n";
    // 在主循环中处理清理工作
    
    return 0;
}
```

### 2. 使用 `sigaction()` 替代 `signal()`

`sigaction()` 提供了更强大和可移植的信号处理方式：

```cpp
#include <iostream>
#include <csignal>
#include <cstring> // 用于 strsignal()

void signalHandler(int signalNum, siginfo_t* info, void* context) {
    std::cout << "收到信号: " << strsignal(signalNum) << " (" << signalNum << ")\n";
    std::cout << "发送者PID: " << info->si_pid << "\n";
    
    if (signalNum == SIGINT) {
        std::cout << "正在优雅地退出...\n";
        exit(0);
    }
}

int main() {
    struct sigaction sa;
    
    // 设置信号处理函数
    sa.sa_sigaction = signalHandler;
    sa.sa_flags = SA_SIGINFO; // 使用扩展信号处理
    
    // 初始化信号集
    sigemptyset(&sa.sa_mask);
    
    // 注册信号处理
    if (sigaction(SIGINT, &sa, NULL) == -1) {
        std::cerr << "无法注册信号处理程序\n";
        return 1;
    }
    
    std::cout << "按下 Ctrl+C 测试 sigaction()...\n";
    
    while(1) {
        pause(); // 等待信号
    }
    
    return 0;
}
```

### 3. 阻塞和解除阻塞信号

```cpp
#include <iostream>
#include <csignal>
#include <unistd.h>

void signalHandler(int signalNum) {
    std::cout << "收到信号: " << signalNum << "\n";
}

int main() {
    sigset_t blockSet, oldSet;
    
    // 初始化信号集
    sigemptyset(&blockSet);
    sigaddset(&blockSet, SIGINT);
    
    // 阻塞 SIGINT 信号
    if (sigprocmask(SIG_BLOCK, &blockSet, &oldSet) == -1) {
        std::cerr << "无法阻塞信号\n";
        return 1;
    }
    
    std::cout << "SIGINT 信号已被阻塞，持续 5 秒...\n";
    std::cout << "在此期间按下 Ctrl+C 将不会生效\n";
    
    // 注册信号处理函数
    signal(SIGINT, signalHandler);
    
    // 等待 5 秒
    sleep(5);
    
    // 解除阻塞 SIGINT 信号
    std::cout << "解除阻塞 SIGINT 信号...\n";
    if (sigprocmask(SIG_SETMASK, &oldSet, NULL) == -1) {
        std::cerr << "无法解除阻塞信号\n";
        return 1;
    }
    
    std::cout << "现在可以按下 Ctrl+C 来测试信号处理...\n";
    
    // 等待信号
    while(1) {
        pause();
    }
    
    return 0;
}
```

## 五、实际应用场景

### 1. 优雅的服务器关闭

```cpp
#include <iostream>
#include <csignal>
#include <atomic>
#include <unistd.h>

std::atomic<bool> running(true);

void shutdownHandler(int signalNum) {
    std::cout << "收到关闭信号，正在优雅地停止服务器...\n";
    running = false;
}

int main() {
    // 设置信号处理
    signal(SIGINT, shutdownHandler);
    signal(SIGTERM, shutdownHandler);
    
    std::cout << "服务器已启动，PID: " << getpid() << "\n";
    std::cout << "发送 SIGTERM 或按下 Ctrl+C 来停止服务器\n";
    
    // 主服务器循环
    while(running) {
        std::cout << "处理请求中...\n";
        sleep(1);
    }
    
    std::cout << "清理资源...\n";
    std::cout << "服务器已停止\n";
    
    return 0;
}
```

### 2. 处理段错误和异常

```cpp
#include <iostream>
#include <csignal>
#include <cstdlib>
#include <execinfo.h> // 用于 backtrace

void segfaultHandler(int signalNum) {
    const int MAX_STACK_SIZE = 100;
    void* stackFrames[MAX_STACK_SIZE];
    
    // 获取调用栈
    int frameCount = backtrace(stackFrames, MAX_STACK_SIZE);
    char** frameStrings = backtrace_symbols(stackFrames, frameCount);
    
    std::cerr << "错误: 收到信号 " << signalNum << "\n";
    std::cerr << "调用栈:\n";
    
    for (int i = 0; i < frameCount; ++i) {
        std::cerr << frameStrings[i] << "\n";
    }
    
    free(frameStrings);
    exit(signalNum);
}

int main() {
    // 设置段错误处理
    signal(SIGSEGV, segfaultHandler);
    
    std::cout << "演示段错误处理...\n";
    
    // 故意制造段错误
    int* ptr = nullptr;
    *ptr = 42; // 这会导致段错误
    
    return 0;
}
```

### 3. 超时处理

```cpp
#include <iostream>
#include <csignal>
#include <unistd.h>

void timeoutHandler(int signalNum) {
    std::cout << "操作超时!\n";
    exit(1);
}

int main() {
    // 设置超时处理
    signal(SIGALRM, timeoutHandler);
    
    std::cout << "设置 3 秒超时...\n";
    alarm(3); // 3 秒后发送 SIGALRM 信号
    
    std::cout << "请输入一些文本 (您有 3 秒时间): ";
    
    std::string input;
    std::cin >> input;
    
    // 取消超时
    alarm(0);
    
    std::cout << "您输入了: " << input << "\n";
    std::cout << "操作完成!\n";
    
    return 0;
}
```

## 六、注意事项和限制

### 1. 信号处理函数的限制

信号处理函数中只能使用异步信号安全的函数。以下是不安全函数的示例：

```cpp
#include <iostream>
#include <csignal>
#include <vector>

// 不安全的信号处理函数示例
void unsafeHandler(int signalNum) {
    // 以下都是不安全的：
    std::cout << "收到信号\n"; // 不安全的I/O操作
    std::vector<int> vec;       // 动态内存分配
    static int count = 0;       // 非原子静态变量访问
    count++;
}

// 安全的信号处理函数示例
volatile sig_atomic_t signalReceived = 0;

void safeHandler(int signalNum) {
    // 只设置一个标志，这是安全的
    signalReceived = 1;
}

int main() {
    signal(SIGINT, safeHandler);
    
    while(!signalReceived) {
        // 主循环
    }
    
    std::cout << "收到信号，正在处理...\n"; // 在主循环中处理复杂操作
    
    return 0;
}
```

### 2. 可重入性问题

信号处理函数必须是可重入的，避免使用全局数据或静态数据：

```cpp
#include <iostream>
#include <csignal>
#include <unistd.h>

// 有问题的实现：使用全局变量
int globalCounter = 0;

void problemHandler(int signalNum) {
    globalCounter++; // 不安全：可能被主程序或其它信号处理函数同时访问
}

// 更好的实现：使用原子类型或sig_atomic_t
volatile sig_atomic_t safeCounter = 0;

void betterHandler(int signalNum) {
    safeCounter++; // 安全：sig_atomic_t保证原子访问
}

int main() {
    signal(SIGINT, betterHandler);
    
    std::cout << "按下多次 Ctrl+C 来测试...\n";
    
    while(safeCounter < 5) {
        // 等待多个信号
    }
    
    std::cout << "收到 " << safeCounter << " 次中断信号\n";
    
    return 0;
}
```

## 七、C++11 及以后的信号处理

C++11 引入了 `<csignal>` 的标准化版本，并提供了更好的类型安全：

```cpp
#include <iostream>
#include <csignal>
#include <atomic>

// 使用C++11的原子类型
std::atomic<bool> stopRequested{false};

// 信号处理函数
extern "C" void signalHandler(int signalNum) {
    // 在信号处理函数中只做最小的工作
    stopRequested = true;
}

int main() {
    // 使用C++11的signal函数（与C兼容但类型更安全）
    if (std::signal(SIGINT, signalHandler) == SIG_ERR) {
        std::cerr << "无法注册信号处理程序\n";
        return 1;
    }
    
    std::cout << "按下 Ctrl+C 来停止程序...\n";
    
    while(!stopRequested) {
        // 主程序逻辑
        std::cout << "工作...\n";
        sleep(1);
    }
    
    std::cout << "正在清理资源...\n";
    std::cout << "程序已停止\n";
    
    return 0;
}
```

## 总结

C++ 信号处理是与操作系统交互的重要机制，它允许程序响应外部事件。关键点包括：

1. **基本函数**：使用 `signal()` 注册处理函数，`raise()` 发送信号
2. **最佳实践**：信号处理函数应尽可能简单，避免使用非异步信号安全函数
3. **高级技术**：使用 `sigaction()` 替代 `signal()` 以获得更多功能
4. **实际应用**：用于优雅关闭、错误处理和超时控制等场景
5. **注意事项**：注意可重入性和线程安全问题

虽然 C++ 标准库提供了基本的信号处理支持，但对于复杂的应用程序，可能需要考虑使用更高级的异步事件处理机制，如多线程、事件循环或平台特定的 API。