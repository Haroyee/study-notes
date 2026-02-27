# C++ 多线程编程详解

C++11 引入了标准的多线程支持，使得编写跨平台的多线程程序变得更加简单和可移植。多线程允许程序同时执行多个任务，从而提高性能和响应能力。

## 一、多线程基础

### 1. 创建线程

使用 `std::thread` 类创建线程：

```cpp
#include <iostream>
#include <thread>
#include <chrono>

// 简单的线程函数
void threadFunction() {
    std::cout << "Hello from thread! Thread ID: " << std::this_thread::get_id() << std::endl;
}

// 带参数的线程函数
void printMessage(const std::string& message, int count) {
    for (int i = 0; i < count; ++i) {
        std::cout << message << " (" << i + 1 << "/" << count << ")" << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

int main() {
    std::cout << "Main thread ID: " << std::this_thread::get_id() << std::endl;
    
    // 创建线程
    std::thread t1(threadFunction);
    std::thread t2(printMessage, "Hello from parameterized thread", 5);
    
    // 等待线程完成
    t1.join();
    t2.join();
    
    std::cout << "Both threads have completed execution." << std::endl;
    
    return 0;
}
```

### 2. 使用 Lambda 表达式创建线程

```cpp
#include <iostream>
#include <thread>
#include <vector>

int main() {
    std::vector<std::thread> threads;
    
    // 使用 Lambda 表达式创建多个线程
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back([i]() {
            std::cout << "Thread " << i << " is running. ID: " 
                      << std::this_thread::get_id() << std::endl;
        });
    }
    
    // 等待所有线程完成
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "All threads have completed." << std::endl;
    
    return 0;
}
```

## 二、线程同步

### 1. 互斥锁 (Mutex)

使用 `std::mutex` 保护共享资源：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex coutMutex;
int sharedCounter = 0;
std::mutex counterMutex;

void safePrint(const std::string& message) {
    std::lock_guard<std::mutex> lock(coutMutex);
    std::cout << message << std::endl;
}

void incrementCounter(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        // 使用 lock_guard 自动管理锁
        std::lock_guard<std::mutex> lock(counterMutex);
        ++sharedCounter;
        
        safePrint("Counter incremented to: " + std::to_string(sharedCounter) + 
                 " by thread: " + std::to_string(std::this_thread::get_id()));
        
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

int main() {
    std::thread t1(incrementCounter, 10);
    std::thread t2(incrementCounter, 10);
    
    t1.join();
    t2.join();
    
    std::cout << "Final counter value: " << sharedCounter << std::endl;
    
    return 0;
}
```

### 2. 使用 `std::lock_guard` 和 `std::unique_lock`

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex resourceMutex;
int sharedResource = 0;

void complexOperation() {
    // 使用 unique_lock 可以更灵活地控制锁
    std::unique_lock<std::mutex> lock(resourceMutex);
    
    // 操作共享资源
    int localCopy = sharedResource;
    
    // 可以临时释放锁，执行一些不涉及共享资源的操作
    lock.unlock();
    
    // 模拟一些不涉及共享资源的工作
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    
    // 重新获取锁
    lock.lock();
    sharedResource = localCopy + 1;
    
    std::cout << "Resource updated to: " << sharedResource 
              << " by thread: " << std::this_thread::get_id() << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back(complexOperation);
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "Final resource value: " << sharedResource << std::endl;
    
    return 0;
}
```

### 3. 条件变量 (Condition Variable)

使用 `std::condition_variable` 进行线程间通信：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> dataQueue;
bool finished = false;

void producer(int count) {
    for (int i = 0; i < count; ++i) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            dataQueue.push(i);
            std::cout << "Produced: " << i << std::endl;
        }
        cv.notify_one(); // 通知一个等待的消费者
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    
    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true;
    }
    cv.notify_all(); // 通知所有消费者
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // 等待条件满足：队列不为空或生产已完成
        cv.wait(lock, []{ return !dataQueue.empty() || finished; });
        
        // 如果队列为空且生产已完成，退出循环
        if (dataQueue.empty() && finished) {
            break;
        }
        
        // 处理数据
        if (!dataQueue.empty()) {
            int data = dataQueue.front();
            dataQueue.pop();
            lock.unlock(); // 尽早释放锁
            
            std::cout << "Consumer " << id << " processed: " << data << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(50));
        }
    }
    
    std::cout << "Consumer " << id << " finished." << std::endl;
}

int main() {
    std::thread prod(producer, 10);
    std::thread cons1(consumer, 1);
    std::thread cons2(consumer, 2);
    
    prod.join();
    cons1.join();
    cons2.join();
    
    std::cout << "All threads completed." << std::endl;
    
    return 0;
}
```

## 三、原子操作

使用 `std::atomic` 进行无锁编程：

```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

std::atomic<int> atomicCounter(0);
int nonAtomicCounter = 0;

void incrementAtomic(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        ++atomicCounter;
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    }
}

void incrementNonAtomic(int iterations) {
    for (int i = 0; i < iterations; ++i) {
        ++nonAtomicCounter; // 这可能导致数据竞争
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    }
}

int main() {
    const int iterations = 100;
    
    // 使用原子计数器
    std::thread t1(incrementAtomic, iterations);
    std::thread t2(incrementAtomic, iterations);
    
    t1.join();
    t2.join();
    
    std::cout << "Atomic counter result: " << atomicCounter << std::endl;
    
    // 使用非原子计数器（可能产生不正确的结果）
    std::thread t3(incrementNonAtomic, iterations);
    std::thread t4(incrementNonAtomic, iterations);
    
    t3.join();
    t4.join();
    
    std::cout << "Non-atomic counter result: " << nonAtomicCounter << std::endl;
    
    return 0;
}
```

## 四、异步编程

### 1. 使用 `std::async` 和 `std::future`

```cpp
#include <iostream>
#include <future>
#include <chrono>
#include <vector>

// 一个耗时的计算函数
long long calculateSum(int start, int end) {
    long long sum = 0;
    for (int i = start; i <= end; ++i) {
        sum += i;
        std::this_thread::sleep_for(std::chrono::microseconds(1));
    }
    return sum;
}

int main() {
    // 使用 async 异步执行任务
    std::future<long long> future1 = std::async(std::launch::async, calculateSum, 1, 1000);
    std::future<long long> future2 = std::async(std::launch::async, calculateSum, 1001, 2000);
    
    // 在主线程中做其他工作
    std::cout << "Main thread is doing other work..." << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    
    // 获取异步任务的结果（如果尚未完成，会阻塞等待）
    long long result1 = future1.get();
    long long result2 = future2.get();
    
    std::cout << "Result 1: " << result1 << std::endl;
    std::cout << "Result 2: " << result2 << std::endl;
    std::cout << "Total: " << result1 + result2 << std::endl;
    
    return 0;
}
```

### 2. 使用 `std::promise` 和 `std::future`

```cpp
#include <iostream>
#include <future>
#include <chrono>
#include <thread>

void performCalculation(std::promise<int> resultPromise, int input) {
    // 模拟耗时计算
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // 设置结果
    resultPromise.set_value(input * 2);
}

int main() {
    std::promise<int> resultPromise;
    std::future<int> resultFuture = resultPromise.get_future();
    
    // 启动计算线程
    std::thread calculationThread(performCalculation, std::move(resultPromise), 21);
    
    // 等待结果
    std::cout << "Waiting for result..." << std::endl;
    int result = resultFuture.get();
    
    std::cout << "The result is: " << result << std::endl;
    
    calculationThread.join();
    
    return 0;
}
```

## 五、线程池实现

实现一个简单的线程池：

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
public:
    ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    
                    {
                        std::unique_lock<std::mutex> lock(this->queueMutex);
                        this->condition.wait(lock, [this] {
                            return this->stop || !this->tasks.empty();
                        });
                        
                        if (this->stop && this->tasks.empty()) {
                            return;
                        }
                        
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    
                    task();
                }
            });
        }
    }
    
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<decltype(f(args...))> {
        using return_type = decltype(f(args...));
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> res = task->get_future();
        
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            
            if (stop) {
                throw std::runtime_error("enqueue on stopped ThreadPool");
            }
            
            tasks.emplace([task](){ (*task)(); });
        }
        
        condition.notify_one();
        return res;
    }
    
    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            stop = true;
        }
        
        condition.notify_all();
        
        for (std::thread &worker : workers) {
            worker.join();
        }
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    
    std::mutex queueMutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;
    
    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "Task " << i << " started by thread " 
                          << std::this_thread::get_id() << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "Task " << i << " finished" << std::endl;
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

## 六、最佳实践和注意事项

### 1. 避免死锁

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mutex1, mutex2;

void process1() {
    // 总是按相同的顺序获取锁
    std::lock_guard<std::mutex> lock1(mutex1);
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    std::lock_guard<std::mutex> lock2(mutex2);
    
    std::cout << "Process 1 completed" << std::endl;
}

void process2() {
    // 按相同的顺序获取锁（与process1相同）
    std::lock_guard<std::mutex> lock1(mutex1);
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    std::lock_guard<std::mutex> lock2(mutex2);
    
    std::cout << "Process 2 completed" << std::endl;
}

// 或者使用std::lock同时获取多个锁
void safeProcess() {
    std::unique_lock<std::mutex> lock1(mutex1, std::defer_lock);
    std::unique_lock<std::mutex> lock2(mutex2, std::defer_lock);
    std::lock(lock1, lock2); // 同时获取两个锁，避免死锁
    
    std::cout << "Safe process completed" << std::endl;
}

int main() {
    std::thread t1(process1);
    std::thread t2(process2);
    std::thread t3(safeProcess);
    
    t1.join();
    t2.join();
    t3.join();
    
    return 0;
}
```

### 2. 使用线程局部存储

```cpp
#include <iostream>
#include <thread>
#include <mutex>

// 线程局部变量
thread_local int threadLocalValue = 0;
std::mutex coutMutex;

void incrementThreadLocal(int id) {
    threadLocalValue += id;
    
    {
        std::lock_guard<std::mutex> lock(coutMutex);
        std::cout << "Thread " << id << ": threadLocalValue = " << threadLocalValue << std::endl;
    }
    
    // 每个线程有自己的threadLocalValue副本
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    threadLocalValue += 10;
    
    {
        std::lock_guard<std::mutex> lock(coutMutex);
        std::cout << "Thread " << id << ": after sleep, threadLocalValue = " << threadLocalValue << std::endl;
    }
}

int main() {
    std::thread t1(incrementThreadLocal, 1);
    std::thread t2(incrementThreadLocal, 2);
    std::thread t3(incrementThreadLocal, 3);
    
    t1.join();
    t2.join();
    t3.join();
    
    // 主线程的threadLocalValue仍然是0
    std::cout << "Main thread: threadLocalValue = " << threadLocalValue << std::endl;
    
    return 0;
}
```

## 七、性能考虑

### 1. 测量多线程性能

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>
#include <cmath>

// 计算密集型任务
void calculateSines(std::vector<double>& results, int start, int end) {
    for (int i = start; i < end; ++i) {
        results[i] = std::sin(i * 0.001);
    }
}

int main() {
    const int size = 10000000;
    std::vector<double> results(size);
    
    // 单线程版本
    auto startSingle = std::chrono::high_resolution_clock::now();
    calculateSines(results, 0, size);
    auto endSingle = std::chrono::high_resolution_clock::now();
    
    std::chrono::duration<double> singleDuration = endSingle - startSingle;
    std::cout << "Single-threaded time: " << singleDuration.count() << " seconds" << std::endl;
    
    // 多线程版本
    const int numThreads = std::thread::hardware_concurrency();
    std::vector<std::thread> threads;
    std::vector<std::vector<double>> threadedResults(numThreads, std::vector<double>(size / numThreads));
    
    auto startMulti = std::chrono::high_resolution_clock::now();
    
    for (int i = 0; i < numThreads; ++i) {
        int start = i * (size / numThreads);
        int end = (i == numThreads - 1) ? size : (i + 1) * (size / numThreads);
        
        threads.emplace_back([&, i, start, end]() {
            calculateSines(threadedResults[i], start, end);
        });
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    // 合并结果
    for (int i = 0; i < numThreads; ++i) {
        int start = i * (size / numThreads);
        int chunkSize = (i == numThreads - 1) ? size - start : size / numThreads;
        
        for (int j = 0; j < chunkSize; ++j) {
            results[start + j] = threadedResults[i][j];
        }
    }
    
    auto endMulti = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double> multiDuration = endMulti - startMulti;
    
    std::cout << "Multi-threaded time (" << numThreads << " threads): " 
              << multiDuration.count() << " seconds" << std::endl;
    std::cout << "Speedup: " << singleDuration.count() / multiDuration.count() << "x" << std::endl;
    
    return 0;
}
```

## 总结

C++ 多线程编程提供了强大的工具来利用现代多核处理器的能力：

1. **基本线程管理**：使用 `std::thread` 创建和管理线程
2. **线程同步**：使用互斥锁、条件变量等机制保护共享资源
3. **原子操作**：使用 `std::atomic` 进行无锁编程
4. **异步编程**：使用 `std::async`, `std::future` 和 `std::promise` 进行异步操作
5. **高级模式**：实现线程池等高级并发模式

多线程编程需要注意：
- 避免数据竞争和死锁
- 合理使用同步机制，避免过度同步
- 考虑性能影响，特别是在创建和销毁线程时
- 使用线程局部存储减少同步需求
- 利用硬件并发性，但不要过度使用线程

正确使用多线程可以显著提高程序性能，但也增加了复杂性。现代 C++ 提供了丰富的工具来帮助编写安全、高效的多线程代码。