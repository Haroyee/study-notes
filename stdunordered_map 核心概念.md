好的，我们来深入探讨 C++ `std::unordered_map` 的操作。这是一个极其有用的容器，以其平均常数时间复杂度的查找性能而闻名。

## `std::unordered_map` 核心概念

`std::unordered_map` 是一个关联容器，存储键值对（key-value pairs）。与 `std::map` 不同，它**不根据键的顺序**存储元素，而是使用**哈希表**实现，通过键的**哈希值**来组织元素。

### 底层实现：哈希表
其工作原理如下：
1.  **哈希函数**：对键进行计算，生成一个 `size_t` 类型的哈希值。
2.  **映射到桶**：哈希值通过与桶的数量取模，决定键值对存储在哪个“桶”（bucket）中。
3.  **处理冲突**：不同的键可能产生相同的哈希值（哈希冲突）。标准要求使用**链地址法**（每个桶是一个链表）来解决冲突。在实践中，高性能实现可能会在链表过长时将其转换为树结构。

---

## 头文件与声明

```cpp
#include <unordered_map>

// 基本声明
std::unordered_map<KeyType, ValueType> myMap;

// 示例：键是字符串，值是整数
std::unordered_map<std::string, int> wordCount;
std::unordered_map<int, std::string> idToName;

// 可以指定哈希函数和相等比较函数的类型（通常使用默认即可）
// std::unordered_map<KeyType, ValueType, Hash, KeyEqual> myMap;
```

---

## 关键操作详解

### 1. 插入操作 (Insertion)

#### 使用 `insert()` 成员函数
`insert` 方法在键不存在时插入，返回一个 `pair<iterator, bool>`，其中 `bool` 表示插入是否成功。

```cpp
std::unordered_map<std::string, int> umap;

// 方法 1: 使用 make_pair
auto result1 = umap.insert(std::make_pair("Alice", 25));
// result1.first 是指向元素的迭代器，result1.second 是 bool (true)

// 方法 2: 使用花括号初始化 (C++11)
auto result2 = umap.insert({"Bob", 30});

// 方法 3: 使用 pair 的构造函数
auto result3 = umap.insert(std::pair<const std::string, int>("Charlie", 35));

// 检查插入是否成功
if (result2.second) {
    std::cout << "Insertion of Bob succeeded.\n";
}
```

#### 使用 `emplace()` 成员函数 (C++11)
`emplace` 直接在容器内构造元素，避免创建临时对象，通常更高效。

```cpp
// 原地构造，传入构造键和值所需的参数
auto result = umap.emplace("David", 40);
// 等同于 umap.insert({"David", 40})，但可能更高效
```

#### 使用下标操作符 `[]`
这是最直观的插入和更新方式。

```cpp
// 插入新键值对
umap["Eve"] = 45;

// 更新已存在的键对应的值
umap["Alice"] = 26; // 现在 Alice 的值是 26

// 注意：如果键不存在，operator[] 会值初始化一个值并插入
// 对于 int，初始化为 0；对于 string，初始化为空字符串等。
int value = umap["Unknown"]; // 插入 {"Unknown", 0}，并返回 0
```

#### 使用 `try_emplace()` 成员函数 (C++17)
更安全和高效的插入方法。如果键已存在，**不构造**值对象。

```cpp
std::string key = "Alice";
// 只有当 key 不存在时，才会构造 std::string(5, 'A') 这个值
auto [iter, success] = umap.try_emplace(key, 5, 'A'); // 尝试插入 {"Alice", "AAAAA"}
// 如果 key 已存在，则不会发生任何构造，success 为 false
```

### 2. 访问操作 (Access)

#### 使用下标操作符 `[]`
```cpp
// 访问存在的键
std::cout << umap["Alice"]; // 输出: 26

// 访问不存在的键（危险！会插入新元素）
std::cout << umap["Ghost"]; // 插入 {"Ghost", 0} 并输出 0
```

#### 使用 `at()` 成员函数
**安全访问**。如果键不存在，抛出 `std::out_of_range` 异常。

```cpp
try {
    std::cout << umap.at("Alice"); // 输出: 26
    std::cout << umap.at("Nonexistent"); // 抛出 std::out_of_range
} catch (const std::out_of_range& e) {
    std::cerr << "Key not found: " << e.what() << '\n';
}
```

#### 使用 `find()` 成员函数
**最常用且安全的访问方法**。返回一个迭代器，如果键不存在则返回 `end()`。

```cpp
auto it = umap.find("Bob");
if (it != umap.end()) {
    // it->first 是键，it->second 是值
    std::cout << "Found: " << it->first << " -> " << it->second << '\n';
} else {
    std::cout << "Key Bob not found.\n";
}
```

### 3. 删除操作 (Deletion)

#### 使用 `erase()` 成员函数
```cpp
// 方法 1: 通过键删除。返回删除的元素个数（0 或 1）
size_t count = umap.erase("Bob"); // count 为 1

// 方法 2: 通过迭代器删除。返回被删除元素之后元素的迭代器
auto it = umap.find("Charlie");
if (it != umap.end()) {
    umap.erase(it); // 删除 Charlie
}

// 方法 3: 通过迭代器范围删除 (C++11)
// umap.erase(start_iterator, end_iterator);
```

### 4. 查找与判断操作 (Lookup)

#### 检查元素是否存在
```cpp
// 方法 1: 使用 count()。返回匹配键的数量（0 或 1）
if (umap.count("Alice") > 0) {
    std::cout << "Alice exists.\n";
}

// 方法 2: 使用 find() (更常用，见上文)

// 方法 3: 使用 contains() - C++20 引入，最直观的表达
#if __cplusplus >= 202002L
if (umap.contains("Alice")) {
    std::cout << "Alice exists.\n";
}
#endif
```

### 5. 容量操作 (Capacity)

```cpp
std::unordered_map<std::string, int> umap = {{"A", 1}, {"B", 2}};

std::cout << "Size: " << umap.size() << '\n';     // 元素数量: 2
std::cout << "Empty: " << umap.empty() << '\n';   // 是否为空: 0 (false)
umap.clear();                                      // 清空所有元素
std::cout << "Size after clear: " << umap.size() << '\n'; // 0
```

### 6. 哈希表特定操作 (Bucket Interface)

这是 `unordered_map` 特有的，用于观察和管理哈希表的内部状态。

```cpp
umap = {{"A", 1}, {"B", 2}, {"C", 3}, {"D", 4}, {"E", 5}};

// 获取桶的数量
size_t bucket_count = umap.bucket_count();
std::cout << "Number of buckets: " << bucket_count << '\n';

// 获取特定键所在的桶编号
size_t bucket_a = umap.bucket("A");
std::cout << "\"A\" is in bucket: " << bucket_a << '\n';

// 获取特定桶中的元素数量
size_t bucket_size = umap.bucket_size(bucket_a);
std::cout << "Elements in bucket " << bucket_a << ": " << bucket_size << '\n';

// 获取负载因子：元素数量 / 桶数量
float load_factor = umap.load_factor();
std::cout << "Current load factor: " << load_factor << '\n';

// 获取最大负载因子
float max_lf = umap.max_load_factor();
std::cout << "Max load factor: " << max_lf << '\n';

// 设置最大负载因子。超过此值，容器会自动增加桶数量（rehash）
umap.max_load_factor(0.8f);

// 手动重新哈希，设置一个指定的最小桶数量
umap.rehash(100); // 确保桶数量至少为 100

// 预留空间，为至少指定数量的元素预留空间（更高效，避免多次rehash）
umap.reserve(1000); // 确保容器可以容纳至少1000个元素而不需要rehash
```

### 7. 遍历操作 (Iteration)

**重要**：遍历顺序是**不确定的**，与插入顺序无关，取决于哈希函数和桶的布局。

```cpp
// 方法 1: 使用迭代器
for (auto it = umap.begin(); it != umap.end(); ++it) {
    std::cout << it->first << ": " << it->second << '\n';
}

// 方法 2: 基于范围的 for 循环 (C++11)
for (const auto& pair : umap) {
    std::cout << pair.first << ": " << pair.second << '\n';
}

// 方法 3: 使用结构化绑定 (C++17) - 最清晰
for (const auto& [key, value] : umap) {
    std::cout << key << ": " << value << '\n';
}
```

### 8. 使用自定义类型作为键

要让自定义类型 `MyKey` 作为 `unordered_map` 的键，你需要提供两个东西：
1.  **哈希函数**：告诉容器如何计算你的类型的哈希值。
2.  **相等比较函数**：告诉容器如何判断两个键是否相等（默认是 `operator==`）。

**方法一：在自定义类型中定义 `operator==` 并特化 `std::hash`**

```cpp
struct MyKey {
    std::string name;
    int id;

    // 1. 必须定义相等操作符
    bool operator==(const MyKey& other) const {
        return name == other.name && id == other.id;
    }
};

// 2. 为 MyKey 特化 std::hash 模板
namespace std {
    template<>
    struct hash<MyKey> {
        size_t operator()(const MyKey& k) const {
            // 组合各个成员的哈希值（常用方法）
            return hash<string>()(k.name) ^ (hash<int>()(k.id) << 1);
        }
    };
}

// 现在可以使用了
std::unordered_map<MyKey, std::string> myMap;
myMap[{"Alice", 1}] = "Engineer";
```

**方法二：将哈希函数和相等比较函数作为模板参数传入（更灵活）**

```cpp
struct MyKey { ... }; // 同上，但不一定需要 operator==

// 自定义哈希函数对象
struct MyKeyHash {
    size_t operator()(const MyKey& k) const {
        return std::hash<std::string>()(k.name) ^ std::hash<int>()(k.id);
    }
};

// 自定义相等比较函数对象
struct MyKeyEqual {
    bool operator()(const MyKey& lhs, const MyKey& rhs) const {
        return lhs.name == rhs.name && lhs.id == rhs.id;
    }
};

// 在声明时传入
std::unordered_map<MyKey, std::string, MyKeyHash, MyKeyEqual> myMap;
```

---

## 性能特点与注意事项总结

1.  **平均时间复杂度**：插入、删除、查找操作的平均情况为 **O(1)**。
2.  **最坏时间复杂度**：所有操作的最坏情况为 **O(n)**（当所有元素都哈希到同一个桶时）。
3.  **内存开销**：比 `std::map` 更高，需要维护桶数组和链表节点。
4.  **迭代器失效**：
    *   插入操作可能导致**重新哈希（rehash）**，使**所有迭代器失效**。
    *   删除操作只使**指向被删除元素的迭代器失效**，其他迭代器仍然有效。
5.  **无序性**：元素顺序不确定，并且可能随时间（插入删除操作后）而改变。
6.  **选择合适的键**：键的类型应该具有良好的哈希函数，能够将键均匀分布到各个桶中。

## 何时选择 `std::unordered_map`？

*   **需要极快的查找速度**。
*   **不关心元素的顺序**。
*   **愿意为速度牺牲更多的内存**。
*   **能够为键类型提供一个好的哈希函数**。

通过熟练掌握这些操作，你就能高效地利用 `std::unordered_map` 来提升程序的性能。