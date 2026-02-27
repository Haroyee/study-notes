# C++ 异步编程详解

C++ 提供了强大的异步编程支持，主要通过 `<future>`, `<thread>`, `<async>` 等头文件实现。异步编程允许你执行并发操作而不需要手动管理线程，提高了代码的可读性和可维护性。

## 异步编程的核心组件

### 1. std::async - 异步执行函数

`std::async` 是启动异步操作的最简单方式，它返回一个 `std::future` 对象，用于获取异步操作的结果。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute(int x, int y) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟耗时操作
    return x + y;
}

int main() {
    // 启动异步任务
    std::future<int> result = std::async(std::launch::async, compute, 10, 20);
    
    // 在主线程中做其他工作
    std::cout << "Main thread working: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 获取异步操作的结果（如果尚未完成，会阻塞等待）
    int value = result.get();
    std::cout << "Result: " << value << std::endl;
    
    return 0;
}
```

### 2. std::future 和 std::promise - 异步结果传递

`std::promise` 和 `std::future` 配对使用，可以在线程之间传递结果。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void compute(std::promise<int>&& prom, int x, int y) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    prom.set_value(x + y); // 设置结果
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    // 启动线程并传递promise
    std::thread t(compute, std::move(prom), 10, 20);
    
    // 获取结果（会阻塞直到结果可用）
    int value = fut.get();
    std::cout << "Result: " << value << std::endl;
    
    t.join();
    return 0;
}
```

### 3. std::packaged_task - 包装可调用对象

`std::packaged_task` 可以将任何可调用对象包装成一个异步任务。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute(int x, int y) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return x + y;
}

int main() {
    // 创建packaged_task
    std::packaged_task<int(int, int)> task(compute);
    std::future<int> fut = task.get_future();
    
    // 在单独线程中执行任务
    std::thread t(std::move(task), 10, 20);
    
    // 获取结果
    int value = fut.get();
    std::cout << "Result: " << value << std::endl;
    
    t.join();
    return 0;
}
```

## 异步编程的高级用法

### 1. 处理异常

异步操作中的异常可以通过 `std::future` 传递到调用线程。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <stdexcept>

int compute(int x, int y) {
    if (y == 0) {
        throw std::runtime_error("Division by zero!");
    }
    return x / y;
}

int main() {
    try {
        std::future<int> result = std::async(std::launch::async, compute, 10, 0);
        int value = result.get(); // 这里会抛出异常
        std::cout << "Result: " << value << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    
    return 0;
}
```

### 2. 等待多个异步操作

使用 `std::future` 的 `wait_for` 和 `wait_until` 方法可以等待异步操作完成。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>
#include <vector>

int compute(int id, int seconds) {
    std::cout << "Task " << id << " started in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(seconds));
    std::cout << "Task " << id << " completed" << std::endl;
    return id * 10;
}

int main() {
    std::vector<std::future<int>> futures;
    
    // 启动多个异步任务
    for (int i = 0; i < 5; ++i) {
        futures.push_back(std::async(std::launch::async, compute, i, i + 1));
    }
    
    // 等待所有任务完成
    for (auto& fut : futures) {
        // 等待最多10秒
        auto status = fut.wait_for(std::chrono::seconds(10));
        if (status == std::future_status::ready) {
            std::cout << "Result: " << fut.get() << std::endl;
        } else {
            std::cout << "Task timed out" << std::endl;
        }
    }
    
    return 0;
}
```

### 3. 使用 std::shared_future

`std::shared_future` 允许多个线程等待同一个异步结果。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>
#include <vector>

void compute(std::promise<int>&& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    prom.set_value(42);
}

void wait_for_result(std::shared_future<int> fut, int id) {
    std::cout << "Thread " << id << " waiting for result..." << std::endl;
    int value = fut.get();
    std::cout << "Thread " << id << " got result: " << value << std::endl;
}

int main() {
    std::promise<int> prom;
    std::shared_future<int> fut = prom.get_future().share();
    
    // 启动计算线程
    std::thread compute_thread(compute, std::move(prom));
    
    // 启动多个等待线程
    std::vector<std::thread> wait_threads;
    for (int i = 0; i < 3; ++i) {
        wait_threads.emplace_back(wait_for_result, fut, i);
    }
    
    compute_thread.join();
    for (auto& t : wait_threads) {
        t.join();
    }
    
    return 0;
}
```

## 异步编程模式

### 1. 异步管道模式

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>

template<typename T>
class AsyncQueue {
private:
    std::queue<T> queue;
    std::mutex mtx;
    std::condition_variable cv;
    bool done = false;
    
public:
    void push(T value) {
        std::lock_guard<std::mutex> lock(mtx);
        queue.push(std::move(value));
        cv.notify_one();
    }
    
    bool pop(T& value) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !queue.empty() || done; });
        
        if (queue.empty() && done) {
            return false;
        }
        
        value = std::move(queue.front());
        queue.pop();
        return true;
    }
    
    void set_done() {
        std::lock_guard<std::mutex> lock(mtx);
        done = true;
        cv.notify_all();
    }
};

void producer(AsyncQueue<int>& queue, int count) {
    for (int i = 0; i < count; ++i) {
        queue.push(i);
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    queue.set_done();
}

void consumer(AsyncQueue<int>& queue, int id) {
    int value;
    while (queue.pop(value)) {
        std::cout << "Consumer " << id << " got: " << value << std::endl;
    }
    std::cout << "Consumer " << id << " done" << std::endl;
}

int main() {
    AsyncQueue<int> queue;
    
    // 启动生产者和消费者
    std::thread prod(producer, std::ref(queue), 10);
    std::thread cons1(consumer, std::ref(queue), 1);
    std::thread cons2(consumer, std::ref(queue), 2);
    
    prod.join();
    cons1.join();
    cons2.join();
    
    return 0;
}
```

### 2. 异步任务链

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <functional>

template<typename T>
std::future<T> then(std::future<T>&& input, std::function<T(T)> func) {
    return std::async(std::launch::async, [input = std::move(input), func]() mutable {
        T value = input.get();
        return func(value);
    });
}

int main() {
    // 创建初始任务
    auto task1 = std::async(std::launch::async, []() {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        return 10;
    });
    
    // 创建任务链
    auto task2 = then(std::move(task1), [](int x) {
        std::cout << "Stage 2 processing: " << x << std::endl;
        return x * 2;
    });
    
    auto task3 = then(std::move(task2), [](int x) {
        std::cout << "Stage 3 processing: " << x << std::endl;
        return x + 5;
    });
    
    // 获取最终结果
    int result = task3.get();
    std::cout << "Final result: " << result << std::endl;
    
    return 0;
}
```

## 性能考虑和最佳实践

### 1. 选择合适的启动策略

`std::async` 支持两种启动策略：
- `std::launch::async`：在新线程中异步执行
- `std::launch::deferred`：延迟执行，直到调用 `get()` 或 `wait()`

```cpp
// 明确指定启动策略
auto result1 = std::async(std::launch::async, compute, 10, 20); // 异步执行
auto result2 = std::async(std::launch::deferred, compute, 10, 20); // 延迟执行

// 默认策略（可能是async或deferred）
auto result3 = std::async(compute, 10, 20);
```

### 2. 避免过多的线程创建

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>
#include <chrono>

// 使用线程池而不是为每个任务创建新线程
class ThreadPool {
private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop = false;
    
public:
    ThreadPool(size_t threads) {
        for (size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this]() {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(mtx);
                        cv.wait(lock, [this]() { return stop || !tasks.empty(); });
                        
                        if (stop && tasks.empty()) {
                            return;
                        }
                        
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    task();
                }
            });
        }
    }
    
    template<typename F, typename... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<decltype(f(args...))> {
        using return_type = decltype(f(args...));
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> res = task->get_future();
        {
            std::lock_guard<std::mutex> lock(mtx);
            if (stop) {
                throw std::runtime_error("enqueue on stopped ThreadPool");
            }
            tasks.emplace([task]() { (*task)(); });
        }
        cv.notify_one();
        return res;
    }
    
    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cv.notify_all();
        for (std::thread& worker : workers) {
            worker.join();
        }
    }
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;
    
    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i]() {
                std::cout << "Task " << i << " in thread " << std::this_thread::get_id() << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                return i * i;
            })
        );
    }
    
    for (auto& result : results) {
        std::cout << "Result: " << result.get() << std::endl;
    }
    
    return 0;
}
```

## C++20 中的新特性

C++20 引入了 `std::jthread`（可连接线程）和 `std::stop_token`，提供了更好的线程管理。

```cpp
#include <iostream>
#include <thread>
#include <stop_token>
#include <chrono>

void worker(std::stop_token stoken) {
    while (!stoken.stop_requested()) {
        std::cout << "Working..." << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Worker stopped" << std::endl;
}

int main() {
    std::jthread jt(worker);
    
    // 主线程做其他工作
    std::this_thread::sleep_for(std::chrono::seconds(3));
    
    // 请求停止工作线程
    jt.request_stop();
    
    // jthread 自动在析构时join
    return 0;
}
```

## 总结

C++ 异步编程提供了强大的工具来编写并发应用程序：

1. **核心组件**：`std::async`, `std::future`, `std::promise`, `std::packaged_task`
2. **高级模式**：异步管道、任务链、线程池
3. **最佳实践**：选择合适的启动策略、避免过多线程创建、正确处理异常
4. **C++20 增强**：`std::jthread` 和 `std::stop_token` 提供更好的线程管理

异步编程可以提高应用程序的响应性和性能，但也需要谨慎处理同步和资源管理问题。正确使用这些工具可以编写出高效、可维护的并发代码。



# `std::async` 详解

`std::async` 是 C++11 引入的一个高级异步编程工具，它提供了一种简单的方式来启动异步任务并获取结果。`std::async` 封装了线程创建和管理细节，使得异步编程更加直观和容易。

## 基本语法

```cpp
#include <future>
#include <iostream>

template< class Function, class... Args >
std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
async( Function&& f, Args&&... args );

template< class Function, class... Args >
std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
async( std::launch policy, Function&& f, Args&&... args );
```

## 启动策略

`std::async` 接受一个可选的启动策略参数，用于指定如何执行异步任务：

1. **`std::launch::async`** - 在新线程中异步执行函数
2. **`std::launch::deferred`** - 延迟执行，直到调用 `get()` 或 `wait()` 时在当前线程执行
3. **`std::launch::async | std::launch::deferred`** - 默认策略，由实现决定

## 基本用法示例

### 1. 简单的异步计算

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute(int a, int b) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟耗时操作
    return a + b;
}

int main() {
    // 启动异步任务
    std::future<int> result = std::async(std::launch::async, compute, 10, 20);
    
    // 在主线程中做其他工作
    std::cout << "Main thread working: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 获取异步操作的结果（如果尚未完成，会阻塞等待）
    int value = result.get();
    std::cout << "Result: " << value << std::endl;
    
    return 0;
}
```

### 2. 使用 lambda 表达式

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int main() {
    // 使用 lambda 表达式作为异步任务
    std::future<int> result = std::async(std::launch::async, []() {
        std::cout << "Lambda executing in thread: " << std::this_thread::get_id() << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(2));
        return 42;
    });
    
    // 做其他工作
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // 获取结果
    std::cout << "Result: " << result.get() << std::endl;
    
    return 0;
}
```

## 启动策略详解

### 1. `std::launch::async` - 异步执行

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void task() {
    std::cout << "Task running in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Task completed" << std::endl;
}

int main() {
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // 明确要求异步执行
    auto future = std::async(std::launch::async, task);
    
    // 等待任务完成
    future.wait();
    std::cout << "Back to main thread" << std::endl;
    
    return 0;
}
```

### 2. `std::launch::deferred` - 延迟执行

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void task() {
    std::cout << "Task running in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Task completed" << std::endl;
}

int main() {
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // 延迟执行
    auto future = std::async(std::launch::deferred, task);
    
    std::cout << "Task not started yet" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 调用 get() 或 wait() 时在当前线程执行
    std::cout << "About to start task..." << std::endl;
    future.get(); // 在当前线程执行任务
    
    std::cout << "Back to main thread" << std::endl;
    
    return 0;
}
```

### 3. 默认策略

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void task() {
    std::cout << "Task running in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Task completed" << std::endl;
}

int main() {
    std::cout << "Main thread: " << std::this_thread::get_id() << std::endl;
    
    // 使用默认策略
    auto future = std::async(task);
    
    // 具体行为由实现决定
    future.wait();
    std::cout << "Back to main thread" << std::endl;
    
    return 0;
}
```

## 异常处理

`std::async` 可以捕获并传递异步任务中抛出的异常：

```cpp
#include <iostream>
#include <future>
#include <stdexcept>
#include <thread>

int compute(int a, int b) {
    if (b == 0) {
        throw std::runtime_error("Division by zero!");
    }
    return a / b;
}

int main() {
    try {
        // 启动可能抛出异常的异步任务
        std::future<int> result = std::async(std::launch::async, compute, 10, 0);
        
        // 获取结果（会重新抛出异常）
        int value = result.get();
        std::cout << "Result: " << value << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "Exception caught: " << e.what() << std::endl;
    }
    
    return 0;
}
```

## 高级用法

### 1. 等待多个异步任务

```cpp
#include <iostream>
#include <future>
#include <vector>
#include <thread>
#include <chrono>

int task(int id, int duration) {
    std::cout << "Task " << id << " started in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(duration));
    std::cout << "Task " << id << " completed" << std::endl;
    return id * 10;
}

int main() {
    std::vector<std::future<int>> futures;
    
    // 启动多个异步任务
    for (int i = 0; i < 5; ++i) {
        futures.push_back(std::async(std::launch::async, task, i, i + 1));
    }
    
    // 等待所有任务完成并收集结果
    std::vector<int> results;
    for (auto& fut : futures) {
        results.push_back(fut.get());
    }
    
    // 输出结果
    std::cout << "All tasks completed. Results: ";
    for (int res : results) {
        std::cout << res << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

### 2. 超时处理

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int long_running_task() {
    std::cout << "Long running task started" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(5)); // 模拟长时间运行的任务
    std::cout << "Long running task completed" << std::endl;
    return 42;
}

int main() {
    // 启动长时间运行的任务
    std::future<int> result = std::async(std::launch::async, long_running_task);
    
    // 等待一段时间，如果超时则放弃
    auto status = result.wait_for(std::chrono::seconds(3));
    
    if (status == std::future_status::ready) {
        std::cout << "Task completed: " << result.get() << std::endl;
    } else if (status == std::future_status::timeout) {
        std::cout << "Task timed out" << std::endl;
        // 注意：任务仍在后台运行！
    } else {
        std::cout << "Task status: deferred" << std::endl;
    }
    
    return 0;
}
```

### 3. 使用 `std::shared_future` 共享结果

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>

int compute() {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return 42;
}

void use_result(std::shared_future<int> fut, int id) {
    std::cout << "Thread " << id << " waiting for result..." << std::endl;
    int value = fut.get();
    std::cout << "Thread " << id << " got result: " << value << std::endl;
}

int main() {
    // 启动计算任务
    std::future<int> result = std::async(std::launch::async, compute);
    
    // 转换为 shared_future 以允许多个线程访问结果
    std::shared_future<int> shared_result = result.share();
    
    // 创建多个线程等待结果
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i) {
        threads.emplace_back(use_result, shared_result, i);
    }
    
    // 等待所有线程完成
    for (auto& t : threads) {
        t.join();
    }
    
    return 0;
}
```

## 性能考虑和最佳实践

### 1. 避免过多的线程创建

```cpp
#include <iostream>
#include <future>
#include <vector>
#include <chrono>

// 对于大量小任务，考虑使用线程池而不是为每个任务创建新线程
void process_item(int item) {
    // 处理项目（模拟工作）
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
}

int main() {
    const int num_items = 1000;
    std::vector<std::future<void>> futures;
    
    // 注意：为每个项目创建一个异步任务可能不是最佳选择
    for (int i = 0; i < num_items; ++i) {
        futures.push_back(std::async(std::launch::async, process_item, i));
    }
    
    // 等待所有任务完成
    for (auto& fut : futures) {
        fut.wait();
    }
    
    std::cout << "All items processed" << std::endl;
    
    return 0;
}
```

### 2. 明确指定启动策略

```cpp
#include <iostream>
#include <future>

int compute() {
    return 42;
}

int main() {
    // 明确指定启动策略，避免意外行为
    auto future1 = std::async(std::launch::async, compute);    // 明确异步执行
    auto future2 = std::async(std::launch::deferred, compute); // 明确延迟执行
    
    // 默认策略（由实现决定）
    auto future3 = std::async(compute);
    
    std::cout << "Result 1: " << future1.get() << std::endl;
    std::cout << "Result 2: " << future2.get() << std::endl;
    std::cout << "Result 3: " << future3.get() << std::endl;
    
    return 0;
}
```

### 3. 资源管理和异常安全

```cpp
#include <iostream>
#include <future>
#include <memory>

class Resource {
public:
    Resource() { std::cout << "Resource acquired" << std::endl; }
    ~Resource() { std::cout << "Resource released" << std::endl; }
    void use() { std::cout << "Using resource" << std::endl; }
};

void task(std::shared_ptr<Resource> resource) {
    resource->use();
    // 模拟工作
    throw std::runtime_error("Task failed");
}

int main() {
    try {
        // 使用智能指针确保资源正确释放
        auto resource = std::make_shared<Resource>();
        auto future = std::async(std::launch::async, task, resource);
        
        future.get(); // 会抛出异常
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    // Resource 会被正确释放，即使任务抛出异常
    
    return 0;
}
```

## 与其它异步工具的比较

### `std::async` vs `std::thread`

| 特性     | `std::async`               | `std::thread`              |
| -------- | -------------------------- | -------------------------- |
| 易用性   | 高（自动管理线程）         | 低（需要手动管理）         |
| 返回值   | 支持（通过 `std::future`） | 不支持（需要通过其他机制） |
| 异常处理 | 自动传递异常               | 需要手动处理               |
| 资源管理 | 自动                       | 手动                       |
| 灵活性   | 较低                       | 较高                       |

### `std::async` vs `std::packaged_task`

| 特性       | `std::async` | `std::packaged_task` |
| ---------- | ------------ | -------------------- |
| 线程管理   | 自动         | 手动                 |
| 使用简便性 | 高           | 中                   |
| 灵活性     | 较低         | 较高                 |
| 适用场景   | 简单异步任务 | 需要更多控制的场景   |

## 总结

`std::async` 是 C++11 提供的一个强大而简单的异步编程工具：

1. **简单易用**：封装了线程创建和管理的复杂性
2. **灵活性**：支持不同的启动策略（异步、延迟）
3. **异常安全**：自动捕获和传递异常
4. **结果获取**：通过 `std::future` 方便地获取异步结果
5. **资源管理**：自动管理线程生命周期

使用 `std::async` 时应注意：
- 明确指定启动策略以避免意外行为
- 对于大量小任务，考虑使用线程池而不是为每个任务创建新线程
- 使用智能指针等 RAII 技术确保资源正确释放
- 注意异常处理，使用 `try-catch` 块捕获异步任务中的异常

`std::async` 是大多数异步编程场景的优秀选择，特别是在需要简单性和安全性的情况下。对于需要更精细控制的场景，可以考虑使用 `std::thread` 和 `std::packaged_task` 等低级工具。



# `std::future` 与 `std::promise` 详解

`std::future` 和 `std::promise` 是 C++11 引入的并发编程工具，它们提供了一种机制来在线程之间传递值和异常。这两个类通常一起使用，形成一个生产者-消费者模式，其中 `std::promise` 是生产者，`std::future` 是消费者。

## `std::future` 详解

`std::future` 是一个模板类，表示一个未来可能得到的值。它提供了一种访问异步操作结果的机制。

### 基本用法

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute() {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return 42;
}

int main() {
    // 使用 std::async 创建 future
    std::future<int> result = std::async(std::launch::async, compute);
    
    // 在主线程中做其他工作
    std::cout << "Main thread working: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 获取结果（如果尚未完成，会阻塞等待）
    int value = result.get();
    std::cout << "Result: " << value << std::endl;
    
    return 0;
}
```

### `std::future` 的主要方法

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int main() {
    std::future<int> result = std::async(std::launch::async, []() {
        std::this_thread::sleep_for(std::chrono::seconds(2));
        return 100;
    });
    
    // 检查 future 是否有效
    if (result.valid()) {
        std::cout << "Future is valid" << std::endl;
    }
    
    // 等待结果可用（阻塞）
    result.wait();
    std::cout << "Result is ready" << std::endl;
    
    // 等待一段时间，返回状态
    auto status = result.wait_for(std::chrono::seconds(1));
    if (status == std::future_status::ready) {
        std::cout << "Result is ready" << std::endl;
    } else if (status == std::future_status::timeout) {
        std::cout << "Timeout occurred" << std::endl;
    } else {
        std::cout << "Deferred" << std::endl;
    }
    
    // 获取结果（只能调用一次）
    int value = result.get();
    std::cout << "Value: " << value << std::endl;
    
    // 再次调用 get() 会导致异常
    try {
        result.get();
    } catch (const std::future_error& e) {
        std::cout << "Future error: " << e.what() << std::endl;
    }
    
    return 0;
}
```

## `std::promise` 详解

`std::promise` 是一个模板类，用于存储一个值或异常，该值或异常可以在以后通过与之关联的 `std::future` 对象获取。

### 基本用法

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void compute(std::promise<int>&& prom) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    prom.set_value(42); // 设置值
    std::cout << "Value set" << std::endl;
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    // 启动线程并传递 promise
    std::thread t(compute, std::move(prom));
    
    // 获取结果（会阻塞直到值被设置）
    int value = fut.get();
    std::cout << "Result: " << value << std::endl;
    
    t.join();
    return 0;
}
```

### `std::promise` 的主要方法

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <stdexcept>

void worker(std::promise<int>&& prom) {
    try {
        // 模拟一些工作
        std::this_thread::sleep_for(std::chrono::seconds(1));
        
        // 设置值
        prom.set_value(42);
        
        // 注意：设置值后不能再修改 promise
    } catch (...) {
        // 捕获任何异常并通过 promise 传递
        prom.set_exception(std::current_exception());
    }
}

void worker_with_exception(std::promise<int>&& prom) {
    try {
        throw std::runtime_error("Something went wrong!");
    } catch (...) {
        prom.set_exception(std::current_exception());
    }
}

int main() {
    // 正常情况
    std::promise<int> prom1;
    std::future<int> fut1 = prom1.get_future();
    std::thread t1(worker, std::move(prom1));
    
    try {
        int value = fut1.get();
        std::cout << "Value: " << value << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception: " << e.what() << std::endl;
    }
    t1.join();
    
    // 异常情况
    std::promise<int> prom2;
    std::future<int> fut2 = prom2.get_future();
    std::thread t2(worker_with_exception, std::move(prom2));
    
    try {
        int value = fut2.get();
        std::cout << "Value: " << value << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception: " << e.what() << std::endl;
    }
    t2.join();
    
    return 0;
}
```

## `std::future` 和 `std::promise` 的高级用法

### 1. 使用 `std::shared_future`

`std::shared_future` 允许多个线程等待同一个异步结果。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>

void compute(std::promise<int>&& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    prom.set_value(42);
}

void wait_for_result(std::shared_future<int> fut, int id) {
    std::cout << "Thread " << id << " waiting for result..." << std::endl;
    int value = fut.get();
    std::cout << "Thread " << id << " got result: " << value << std::endl;
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    // 转换为 shared_future
    std::shared_future<int> shared_fut = fut.share();
    
    // 启动计算线程
    std::thread compute_thread(compute, std::move(prom));
    
    // 启动多个等待线程
    std::vector<std::thread> wait_threads;
    for (int i = 0; i < 3; ++i) {
        wait_threads.emplace_back(wait_for_result, shared_fut, i);
    }
    
    compute_thread.join();
    for (auto& t : wait_threads) {
        t.join();
    }
    
    return 0;
}
```

### 2. 超时处理

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void compute(std::promise<int>&& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(5)); // 长时间运行
    prom.set_value(42);
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    std::thread t(compute, std::move(prom));
    
    // 等待一段时间，如果超时则放弃
    auto status = fut.wait_for(std::chrono::seconds(3));
    
    if (status == std::future_status::ready) {
        std::cout << "Result: " << fut.get() << std::endl;
    } else if (status == std::future_status::timeout) {
        std::cout << "Timeout occurred" << std::endl;
        // 注意：计算线程仍在运行！
    }
    
    t.join(); // 仍然需要等待线程结束
    return 0;
}
```

### 3. 使用 `std::packaged_task`

`std::packaged_task` 是 `std::promise` 和 `std::function` 的结合体，用于包装任何可调用对象。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute(int x, int y) {
    std::cout << "Computing in thread: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return x + y;
}

int main() {
    // 创建 packaged_task
    std::packaged_task<int(int, int)> task(compute);
    std::future<int> fut = task.get_future();
    
    // 在单独线程中执行任务
    std::thread t(std::move(task), 10, 20);
    
    // 获取结果
    int value = fut.get();
    std::cout << "Result: " << value << std::endl;
    
    t.join();
    return 0;
}
```

### 4. 复杂数据类型传递

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>
#include <string>

struct Result {
    int value;
    std::string message;
    std::vector<int> data;
};

void compute(std::promise<Result>&& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    Result result;
    result.value = 42;
    result.message = "Computation successful";
    result.data = {1, 2, 3, 4, 5};
    
    prom.set_value(std::move(result)); // 使用移动语义提高效率
}

int main() {
    std::promise<Result> prom;
    std::future<Result> fut = prom.get_future();
    
    std::thread t(compute, std::move(prom));
    
    Result result = fut.get();
    std::cout << "Value: " << result.value << std::endl;
    std::cout << "Message: " << result.message << std::endl;
    std::cout << "Data: ";
    for (int num : result.data) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    t.join();
    return 0;
}
```

## 异常处理最佳实践

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <stdexcept>

void compute(std::promise<int>&& prom) {
    try {
        // 模拟可能抛出异常的操作
        if (rand() % 2 == 0) {
            throw std::runtime_error("Random error occurred");
        }
        
        // 正常计算
        std::this_thread::sleep_for(std::chrono::seconds(1));
        prom.set_value(42);
    } catch (...) {
        // 捕获所有异常并通过 promise 传递
        try {
            prom.set_exception(std::current_exception());
        } catch (...) {
            // set_exception 也可能抛出异常（如果 promise 已设置值）
            // 这种情况下，我们无法做太多
        }
    }
}

int main() {
    for (int i = 0; i < 5; ++i) {
        std::promise<int> prom;
        std::future<int> fut = prom.get_future();
        
        std::thread t(compute, std::move(prom));
        
        try {
            int value = fut.get();
            std::cout << "Result: " << value << std::endl;
        } catch (const std::exception& e) {
            std::cout << "Exception: " << e.what() << std::endl;
        }
        
        t.join();
    }
    
    return 0;
}
```

## 性能考虑和最佳实践

### 1. 避免不必要的拷贝

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <vector>

void process_large_data(std::promise<std::vector<int>>&& prom) {
    std::vector<int> large_data(1000000, 42); // 创建大量数据
    
    // 使用移动语义避免拷贝
    prom.set_value(std::move(large_data));
}

int main() {
    std::promise<std::vector<int>> prom;
    std::future<std::vector<int>> fut = prom.get_future();
    
    std::thread t(process_large_data, std::move(prom));
    
    // 获取数据（使用移动语义）
    std::vector<int> result = fut.get();
    std::cout << "Data size: " << result.size() << std::endl;
    
    t.join();
    return 0;
}
```

### 2. 使用 `std::ref` 传递引用

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <functional>

void modify_data(std::promise<void>&& prom, std::vector<int>& data) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 修改数据
    for (int& num : data) {
        num *= 2;
    }
    
    prom.set_value();
}

int main() {
    std::vector<int> data = {1, 2, 3, 4, 5};
    
    std::promise<void> prom;
    std::future<void> fut = prom.get_future();
    
    // 使用 std::ref 传递引用
    std::thread t(modify_data, std::move(prom), std::ref(data));
    
    fut.wait(); // 等待修改完成
    
    std::cout << "Modified data: ";
    for (int num : data) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    t.join();
    return 0;
}
```

## 总结

`std::future` 和 `std::promise` 是 C++ 并发编程中强大的工具，它们提供了一种机制来在线程之间安全地传递值和异常。

### 关键点：

1. **`std::future`**:
   - 表示一个未来可能得到的值
   - 提供 `get()`, `wait()`, `wait_for()`, `wait_until()` 等方法
   - 只能获取一次结果

2. **`std::promise`**:
   - 用于存储一个值或异常
   - 提供 `set_value()`, `set_exception()` 等方法
   - 与 `std::future` 关联，通过 `get_future()` 方法

3. **`std::shared_future`**:
   - 允许多个线程等待同一个异步结果
   - 可以通过 `std::future::share()` 创建

4. **`std::packaged_task`**:
   - 包装任何可调用对象，自动管理 `std::promise`

5. **最佳实践**:
   - 使用移动语义避免不必要的拷贝
   - 正确处理异常
   - 使用 `std::ref` 传递引用
   - 考虑使用超时处理避免无限等待

`std::future` 和 `std::promise` 提供了一种强大而灵活的方式来实现线程间的通信和同步，是现代 C++ 并发编程的重要组成部分。