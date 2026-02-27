 

# 一、C++基础

## 1、vscode配置

tasks.json文件（第六行command切换gcc/g++）

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc.exe 生成活动文件",
            "command": "D:\\Soft\\mingw64\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

## 2、输入输出

### (1)基本IO

- `std::cerr`：标准错误流（无缓冲），用于输出错误信息。
- `std::clog`：标准日志流（有缓冲），用于输出日志信息。

输出

```cpp
std::cout<<"hello world!"<<std::endl;
/*endl为回车*/
```

输入

```cpp
/*接收单个*/
std::cin>>a;
/*接收多个*/
std::cin>>a>>b>>c;
```

### (2)格式化输出

需添加头文件<iomanip>

- `std::setw(int n)`：设置下一个输出项的字段宽度。

- `std::setprecision(int n)`：设置浮点数的精度。

- `std::fixed` / `std::scientific`：设置浮点数的输出格式（定点/科学计数法）。

- `std::boolalpha`：将布尔值输出为 `true`/`false` 而非 `1`/`0`。

  ```cpp
  #include <iostream>
  #include <iomanip> //需要添加的头文件
  int main()
  {
      double f = 9854.153684846484;
      std::cout << "科学计数法：" << std::setw(20) << std::setprecision(5) << std::scientific << f << std::endl;
      std::cout << "定点：" << std::setw(20) << std::setprecision(5) << std::fixed << f << std::endl;
  
      return 0;
  }
  ```

### (3)字符串流

需添加头文件<sstream>

- `std::istringstream`：字符串输入流，用于从字符串读取数据。

- `std::ostringstream`：字符串输出流，用于将数据格式化成字符串。

- `std::stringstream`：兼具输入和输出功能。

  ```cpp
  #include <iostream>
  #include <sstream>
  
  int main()
  {
  
      /*数字转字符串*/
  
      std::ostringstream oss;
      oss << 12 << 85 << "45" << std::endl;
      std::cout << oss.str() << std::endl;
  
      /*字符串解析*/
      std::istringstream iss("12 85 jack 0");
      int a;
      int b;
      std::string str;
      bool flag;
      iss >> a >> b >> str >> flag;
      std::cout << a << " " << b << " " << str << " " << flag << std::endl;
  
      return 0;
  }
  ```

### (4)文件流

需要头文件<fstream>

- `std::ifstream`：文件输入流，用于从文件读取数据。
- `std::ofstream`：文件输出流，用于向文件写入数据。
- `std::fstream`：兼具文件输入和输出功能。

**①打开文件**

```cpp
/*使用输入流*/
std::ifstream infile("data.txt");

/*使用输出流.open()函数*/
std::ofstream outfile;
outfile.open("output.txt");

/*检查文件是否打开*/
if (!infile.is_open()) { // 或者直接用 if (infile)
    std::cerr << "Failed to open file!" << std::endl;
    return -1;
}

/*文件打开模式*/
/*
std::ios::in(只写), std::ios::out(只读), std::ios::app (追加), std::ios::binary (二进制模式) 等。可以用 | 组合。
*/
std::ofstream appfile("log.txt", std::ios::app); // 以追加模式打开
```

**②读写文件**

```cpp
/*逐词读取*/
std::string word;
int i = 0
while(infile>>word){
    ++i;
    std::cout<<word<<" ";
    if(i%30==0) std::cout<<std::setw(10)<<std::endl;//每30词换行
}

/*逐行读取*/
std::string line;
while (std::getline(infile, line)) {
    std::cout << line << std::endl;
}
/*写文件同理*/
```

二进制读写

使用 `.read()` 和 `.write()` 方法，处理非文本数据（如图片、结构体）。

```cpp
struct Data { int a; double b; };
Data d = {10, 20.5};

// 写
std::ofstream binfile("data.bin", std::ios::binary);
binfile.write(reinterpret_cast<char*>(&d), sizeof(d));
binfile.close();

// 读
std::ifstream binin("data.bin", std::ios::binary);
Data d2;
binin.read(reinterpret_cast<char*>(&d2), sizeof(d2));
```

③关闭文件

使用 `.close()` 方法。虽然流对象析构时会自动关闭，但显式关闭是好习惯。

### (5)流的状态

- 流对象有状态标志位，用来指示流当前是否正常。
- `eof()`：是否到达文件末尾（End-Of-File）。
- `fail()`：最近的操作是否失败（例如试图将 `"hello"` 读入 `int` 变量）。
- `bad()`：流是否发生严重错误（如无法读取文件）。
- `good()`：流是否一切正常。
- **清除状态**：`clear()` 方法，在重试操作前需要先清除错误状态。
- 理解流是有缓冲的，数据可能不会立即写入设备，而是先存储在缓冲区。
- `std::endl` 会刷新缓冲区，`\n` 则不会。
- 手动刷新：`std::flush` 操纵器或 `flush()` 方法。

### (6)io重载

- 可以通过重载 `<<` 和 `>>` 运算符，让自己的类也能像内置类型一样方便地进行I/O操作。

```cpp
class Person {
public:
    std::string name;
    int age;
    // ... 其他成员
};

// 输出重载
std::ostream& operator<<(std::ostream& os, const Person& p) {
    os << "Name: " << p.name << ", Age: " << p.age;
    return os;
}

// 输入重载
std::istream& operator>>(std::istream& is, Person& p) {
    is >> p.name >> p.age;
    return is;
}

// 使用
Person me;
std::cin >> me; // 现在可以直接输入了
std::cout << me; // 也可以直接输出
```

### (7)getline的使用

用于从键盘和文件等逐行读取数据

使用实例

```cpp
std::getline(数据流,line)//数据流中的数据被读取到line中
/*有以下例子*/
/*从键盘获取*/
std::getline(std::cin,line)
/*从文件获取*/
std::ifstream infile("data.txt");
std::getline(infile,line);
/*从字符串流获取*/
std::istringstream iss(str);
std::getline(iss,line);
```

逐行读取示例

```cpp
#include <iostream>
#include <sstream>
#include <vector>
#include <iomanip>
/*getline函数的使用*/

int main()
{
    std::vector<int> numbers;
    std::string line;
    int num = 0;

    while (std::getline(std::cin, line))
    { // 在输入EOF前循环执行，即ctrl+z后+enter
        std::istringstream iss(line);
        while (iss >> num)
            numbers.push_back(num);
    }
    int i = 0;
    for (int n : numbers)
    {
        ++i;
        std::cout << std::setw(5) << n << " ";
        if (i % 10 == 0)
            std::cout << std::endl;
    }
    return 0;
}
```

## 3、指针与引用

### （1）指针

详情见c语言指针

### （2）引用

**引用**（Reference）是一种别名，它为已存在的对象提供了另一个名称，即指向地址还是已存在的对象的地址。

●引用必须初始化，而且不可重新绑定，通常不占用额外存储

●提高代码效率（避免不必要的拷贝）

●使函数能够修改调用者的变量

```cpp
int a = 5;
int &ref = a;
ref = 7;//修改ref就是修改a;
```

```cpp
void swap(int &a,int &b)//引用用于交互俩个变量的值
{
    int temp;
    temp = a;
    a = b;
    b = temp;
}
```

**右值引用**

右值引用主要实现对资源的窃取

```cpp
int a = 100;
int &&ref = std::move(a);//std::move将a转化为右值引用付给ref；
/*对于int类型来说，实际上此操作仍是将a取了个别名ref，并没有实现资源的窃取，对ref的操作就是对a的操作*/
```

```cpp
vector<int> numbers = {1,2,3,4,5};
int *p = std::move(numbers);
std::cout<<numbers.size();//输出结果为0
```



# 二、C++中阶

## 1、函数的重载

通过函数重载，多个函数可以具有相同的名称和不同的参数，只要参数的数量和/或类型不同，多个函数可以具有相同的名称。

```cpp
int add(int a,int b)
    return a+b;
}
int add(int a, int b,int c){
    return a+b+c;
}
char add(char a,char b){
    return a+b;
}
```

## 2、面向对象

### (1)面对对象编程（opp）

**面向对象**：对象是指具体的某一个事物，这些事物的抽象就是类，类中包含数据（成员变量）和动作（成员方法）。

**面向过程编程：**一种以执行程序操作的过程或函数为中心编写软件的方法。程序的数据通常存储在变量中，与这些过程是分开的。所以必须将变量传递给需要使用它们的函数。缺点：随着程序变得越来越复杂，程序数据与运行代码的分离可能会导致问题。例如，程序的规范经常会发生变化，从而需要更改数据的格式或数据结构的设计。当数据结构发生变化时，对数据进行操作的代码也必须更改为接受新的格式。查找需要更改的所有代码会为程序员带来额外的工作，并增加了使代码出现错误的机会。

**面向对象编程（Object-Oriented Programming, OOP）**：以创建和使用对象为中心。一个对象（Object）就是一个软件实体，它将数据和程序在一个单元中组合起来。对象的数据项，也称为其属性，存储在成员变量中。对象执行的过程被称为其成员函数。将对象的数据和过程绑定在一起则被称为封装。

面向对象编程将数据成员和成员函数封装到一个类中，并声明数据成员和成员函数的访问级别（public、private、protected），以便控制类对象对数据成员和函数的访问，对数据成员起到一定的保护作用。而且在类的对象调用成员函数时，只需知道成员函数的名、参数列表以及返回值类型即可，无需了解其函数的实现原理。当类内部的数据成员或者成员函数发生改变时，不影响类外部的代码。

### (2)构造函数与虚函数

构造函数分为无参构造，有参构造，全参构造、拷贝构造、析构函数、移动函数

```cpp
/*无参构造(默认有)*/
    Rectangle()
    {
        length = 0;
        breadth = 0;
    }
/*全参构造*/
    Rectangle(double len, double bre)
    {
        length = len;
        breadth = bre;
    }
```

①**虚函数**：在基类中使用`virtual`关键字声明。实现运行时多态，允许在派生类中重写基类的函数。

②**析构函数**：类的特殊成员函数，它在对象生命周期结束时自动调用，用于执行清理操作，使用波浪号~声明。

确保资源在对象生命周期结束时被正确释放，即类被销毁时释放动态分配的资源，默认的析构函数不会对动态的分配资源进行释放容易导致内存泄露。

③**虚析构函数：**当作为基类被继承时，销毁派生类只会调用基类的析构函数，需要将基类的析构函数变成虚析构函数，以便在派生类中重写

```cpp
class fa{//基类
    
    protected:
    int *p;
    
    public:
    fa(int q[]){//构造函数
        p = new int[sizeof(q)];
        std::copy(std::begin(q),std::end(q),std::(p));//拷贝
    }
    virtual ~fa(){//虚析构函数
        delete[] p;
        std::cout<<"fa析构函数"<<std::endl;
    }
    
}
class ch{//派生类
    public:
    /*
    using fa::fa;
    */
    ch(int q[]):fa(q[]){}//构造函数
    
    ~ch() override{//重写虚函数
        delete[] p;
        std::cout<<"ch析构函数"<<std::endl;
    }
}
```

**④拷贝构造：**是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象。拷贝构造函数通常用于：

- 通过使用另一个同类型的对象来初始化新创建的对象。
- 复制对象把它作为参数传递给函数。
- 复制对象，并从函数返回这个对象。

**浅拷贝**：类的默认拷贝函数就是浅拷贝，对于类中的基本数据类型是直接复制，对于指针是复制指针值（而不是指针指向的内容）

**深拷贝**：复制指针值（而不是指针指向的内容）

```cpp
class R
{
protected:
    int *ptr;

public:
    R(int p)
    {
        ptr = new int;
        *ptr = p;
    }
    R(const R &obj)
    { // 深拷贝函数
        ptr = new int;
        *ptr = *obj.ptr;
        std::cout << "执行R的深拷贝函数" << std::endl;
    }
    R &operator=(const R &obj) // 深拷贝赋值函数
    {
        if (this != &obj)
        {               // 处理自我赋值
            delete ptr; // 释放当前资源
            ptr = new int;
            *ptr = *obj.ptr;
            std::cout << "执行R的深拷贝赋值函数" << std::endl;
        }
        return *this;
    }
    void setPtr(int p)
    {
        *ptr = p;
    }
    int *getPtr()
    {
        return ptr;
    }
};
```



### (3)面向对象三大特征

**①封装**

将具体的实现过程和数据封装成一个函数，只能通过接口进行访问，降低耦合性。

**②继承**

子类继承父类的特征和行为，子类有父类的非 private 方法或成员变量，子类可以对父类的方法进行重写，增强了类之间的耦合性，但是当**父类或者虚函数被final修饰**时,修饰的**类**不能继承,修饰的**虚函数不能被重新和修改**（c++中final只能修饰类本身和虚函数）

java：当父类中的成员变量、成员函数或者类本身被 **final 关键字**修饰时，**修饰的类不能继承**，**修饰的成员不能重写或修改**。

```cpp
/*无法被继承*/
class R final{//被final修饰的类
    ...
  virtual func() final{//被final修饰的虚函数
        ...
    }
}
```

c++的类支持多继承，就是指子类可以继承多个父类；但在java中类并不支持多继承

- **公有继承（public）：**当一个类派生自**公有**基类时，基类的**公有**成员也是派生类的**公有**成员，基类的**保护**成员也是派生类的**保护**成员，基类的**私有**成员不能直接被派生类访问，但是可以通过调用基类的**公有**和**保护**成员来访问。
- **保护继承（protected）：** 当一个类派生自**保护**基类时，基类的**公有**和**保护**成员将成为派生类的**保护**成员。
- **私有继承（private）：**当一个类派生自**私有**基类时，基类的**公有**和**保护**成员将成为派生类的**私有**成员。

```cpp
/*公有多继承*/
class Ra:public R,public T,public Y{
    ...
}
```

**公有类成员（public）**：可以被类内外部访问，也可以被子类类外部访问；

**保护类成员（protected）**：可以被类和子类内部访问，但不可在类外部访问；

**私有类成员（private）**：可以在自身类（同类）内部访问，不可在在子类内外部访问，也不可在类外部访问；

```cpp
class R{
public:
   	int func(){
        return a + b + c;
    };
protected:
    int c;
private:
    int a;
    int b;
}

```

父类的构造函数无法被子类继承，需要自行在子类中添加

```cpp
class RectangleA : public Rectangle{
    public:
    using Rectangle::Rectangle; // 依据父类生成所有构造函数(无法继承构造函数与析构函数)

        /*
        等效于：
        RectangleA(double len,double bre):Rectangle(len,bre){}
        RectangleA():Rectangle(){}
        */
}
```

**Tip：类与结构体最本质的区别就是默认访问权限不同，类中可以设定成员的访问权限，而结构体默认成员访问权限均为public。**

**③多态**

多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。

多态允许使用基类指针或引用来调用子类的重写方法，从而使得同一接口可以表现不同的行为。

多态使得代码更加灵活和通用，程序可以通过基类指针或引用来操作不同类型的对象，而不需要显式区分对象类型。

多态主要通过虚函数以及纯虚函数实现，使用虚函数可以在派生类中对基类中的函数进行重写，而纯虚函数在基类实现接口，在派生类中实现接口的实际操作。

```cpp
class fa{
    protected:
    int a;
    int b;
    public:
    virtual int add() = 0;//纯虚函数
    virtual int mul(){//虚函数
        return a* b;
    }
}
class ch:public fa{
    public:
    int add() override{
        return a+ b;
    }
    int mul() override{
        return 2*a*b;
    }
}
```

### (4)友员

友元是 C++ 中一种特殊的机制，它允许一个类或函数访问另一个类的私有（private）和保护（protected）成员。友元关系打破了封装性，因此应该谨慎使用。

### (5)类的单例模式

**类的单例模式**是一种创建型设计模式，确保一个类只有一个实例，并提供一个全局访问点。

常见的单例模式实现有以下几种方式：

1. 懒汉式（Lazy Initialization）：在第一次被使用时才初始化。
2. 饿汉式（Eager Initialization）：在程序启动时就初始化。
3. 双检锁（Double-Checked Locking）：使用锁来保证线程安全，且减少性能开销。
4. 使用局部静态变量（Meyers' Singleton）：利用局部静态变量的特性，在C++11及以上保证线程安全。

#### ①懒汉式与饿汉式

```cpp
/*经典单例模式*/
class Classic_Singleton
{
private:
    static Classic_Singleton *instance;
    Classic_Singleton() {}; // 构造函数私有化，防止外部实例化

    Classic_Singleton(const Classic_Singleton &R) = delete;           // 禁用拷贝函数
    Classic_Singleton operator=(const Classic_Singleton &R) = delete; // 禁用拷贝赋值函数
public:
    static Classic_Singleton *getInstence()
    {
        /*下方被框选的代码删除即为饿汉式单例模式*/
        /*-----------------------*/
        if (instance == nullptr)
        {
            instance = new Classic_Singleton;
        }
        /*----------------------*/
        return instance;
    }
    ~Classic_Singleton()
    {
        if (!instance)
            delete instance;
    }

    static void log(const std::string &str) // 静态方法
    {
        std::cout << "log : " << str << std::endl;
    }
    static void destroy()
    {
        if (instance != nullptr)
        {
            delete instance;
            instance = nullptr;
        }
        std::cout << "this class has destroyed" << std::endl;
    }
    void show()
    {
        std::cout << "this just a show!" << std::endl;
    }
};
```

初始化方式

```cpp
/*懒汉式-初始化方式*/
Classic_Singleton *Classic_Singleton::instance = nullptr;
/*饿汉式-初始化方式*/
Classic_Singleton *Classic_Singleton::instance = new Classic_Singleton();
```

使用方式

```cpp
Classic_Singleton::log("hello world!");
Classic_Singleton::getInstence()->show();
Classic_Singleton::destroy();
```

#### ②使用局部静态变量(推荐)

```cpp
class Meyer_Singleton
{
private:
    Meyer_Singleton() {};
    Meyer_Singleton(const Meyer_Singleton &m) = delete;
    Meyer_Singleton operator=(const Meyer_Singleton &m) = delete;

public:
    static Meyer_Singleton *getInstance()
    {
        static Meyer_Singleton instance;
        return &instance;
    }
    // 不提供销毁函数，因为静态资源由系统自动管理
    void show()
    {
        std::cout << "this just a show!" << std::endl;
    }
};
```

#### ③双检锁

[](#③call_once在类的单例中的使用)







# 三、C++进阶

## 1、命名空间

### (1)定义

C++命名空间（Namespace）是一种用来组织代码的机制，它可以帮助避免命名冲突，尤其是在大型项目中使用多个库时。命名空间将相关的代码（如变量、函数、类等）放置在一个命名的作用域内。

### (2)简单示例

定义俩个命名空间

```cpp
#include <iostream>
// 定义命名空间
namespace Math {
    int add(int a, int b) {
        return a + b;
    }
}

namespace Physics {
    int add(int a, int b) {
        return a + b + 10; // 假设物理中的加法规则不同
    }
}
```

单独使用

```cpp
int main() {
    // 使用命名空间中的函数
    std::cout << "Math::add(2, 3): " << Math::add(2, 3) << std::endl;
    std::cout << "Physics::add(2, 3): " << Physics::add(2, 3) << std::endl;
    return 0;
}
```

全局声明（不建议）

```cpp
int main() {

    // 使用using声明
    using Math::add;
    std::cout << "add(2, 3): " << add(2, 3) << std::endl; // 使用Math的add

    // 使用using指令
    using namespace Physics;
    std::cout << "add(2, 3): " << add(2, 3) << std::endl; // 现在使用Physics的add，因为using指令引入了整个Physics命名空间
    return 0;
}
```

### (3)标准命名空间std

所有 C++ 标准库类型和函数都在 `std` 命名空间或嵌套在 `std` 内的命名空间中进行声明

[C++ 标准库 - cppreference.cn - C++参考手册](https://cppreference.cn/w/cpp/standard_library)

### (4)创建命名空间

```cpp
namespace nsp{
/*code*/
    void func1(){
        /*code*/
    }
    void func2(){
        /*code*/
    }
};
```

使用命名空间

```cpp
/*应用nsp中的全部函数*/
using namespace nsp;

/*只应用func1函数*/
/*第一种*/
using nsp::func1;
func1();

/*第二种*/
nsp::func1();

```

### (5)嵌套命名空间

创建嵌套命名空间

```cpp
namespace nspParents//父命名空间
{
    void helloWorld()
    {
        std::cout << "hello world!" << std::endl;
        return;
    }
    namespace nspChild//子命名空间
    {
        void helloWorld()
        {
            std::cout << "hello everybody!" << std::endl;
            return;
        }
    }
}
```

使用被嵌套的命名空间

```cpp
nspParents::helloWorld();
namespace nsp = nspParents::nspChild; // 调用子命名空间
nsp::helloWorld();
```



## 2、动态内存管理

**动态内存管理**主要涉及使用`new`和`delete`操作符（或它们的数组版本`new[]`和`delete[]`）来手动管理内存。然而，手动管理内存容易出错，因此现代C++更推荐使用智能指针和容器来管理内存，以减少内存泄漏和悬空指针的风险。

### (1)**手动内存管理方式**

**①常规数据类型内存管理**

new对应delete，new[]对应delete[]

```cpp
int* ptr = new int(10);//一个整数
delete ptr; // 释放内存
int* arr = new int[5];多个整数
delete[] arr;
```

**②类的内存管理**（析构函数）

```cpp
class R
{
    private:
    int *ptr;
    public:
    R(int d)
    {
        ptr = new int(d);
    }
    ~R()//析构函数
    {
        delete ptr;
    }
}
```

### (2)智能指针管理内存

#### **①独占智能指针：unique_ptr**

独占所有权的智能指针,**无拷贝函数与拷贝赋值函数**，**有移动函数和移动赋值函数**；

意味着unique_ptr可以窃取其他智能指针的原始指针，但是无法直接赋值拷贝；

另外指针指针不支持指针的运算（+，-，++，--）；

```cpp
std::unique_ptr<int> up1(new int(10));
std::unique_ptr<int> up2 = up1;//非法操作，无法拷贝
std::unique_ptr<int> up3 = std::move(up1);//有效操作，可移动
/*--------------------------------------------------------*/
std::unique_ptr<int> up4 = std::unique_ptr<int>(new int(10));
//有效操作，此操作无需拷贝函数，‘unique_ptr<int>(new int(10))’为临时的右值
/*---------------------------------------------------------*/
std::unique_ptr<int> func(){
   std:: unique_ptr<int> pp(new int(10))
        return pp;
}
up4 = func();//有效操作，此操作调用了移动函数
```

初始化方式

```cpp
class R
{
    public:
    int num;
    R(int n)
    {
        this->num = n;
    }
};
/*方式一(推荐)c++11*/
std::unique_ptr<int> up11(new int(10));
std::unique_ptr<R> up12(new R(10));
/*方式二c++14*/
std::unique_ptr<int> up21 = std::make_unique<int>(10);
std::unique_ptr<R> up21 = std::make_unique<R>(10);
/*方式三*/
int *p = new int(10);
R *r = new R(10);
std::unique_ptr<int> up11(p);
std::unique_ptr<R> up12(R);
```

不建议多个智能指针指向同一原始指针

```cpp
int *p = new int(10);
std::unique_ptr<int> up1(p);//当程序结束时释放p
std::unique_ptr<int> up2(p);//由于与up1指向统一指针，程序结束时再次释放p，但p已经被释放
```

**unique_ptr常用操作**

```cpp
std::unique_ptr up(new int(5));

/*重置指针*/
up.reset(new int(10)); // 释放当前管理的对象，并接管新指针的所有权
up.reset(); // 释放当前对象，并将up置为空

/*释放所有权*/
int* raw_ptr = up.release(); // 返回裸指针，并释放所有权，up变为空，不会销毁对象

/*交换*/
std::unique_ptr up1(new int(8));
up.swap(up1);
std::swap(up,up1);

/*检查是否为空*/
if (up) 
if (up != nullptr)
    
/*访问对象*/
up[n];//拥有数组的具体化版本
up->member;
(*up);
up.get();// get()返回裸指针，但不释放所有权

/*转换为shared_ptr*/
std::shared_ptr<int> sp = std::move(up); // 将unique_ptr移动转换为shared_ptr
```

#### ②共享智能指针：shared_ptr

共享所有权的智能指针，使用引用计数机制,有拷贝和拷贝赋值函数

初始化方式

```cpp
/*方式一*/
std::shared_ptr<int> sp0(new int(10));
/*方式二(推荐)*/
std::shared_ptr<int> sp1 = std::make_shared<int>(10);
/*方式三*/
int *nums = new int(10);
std::shared_ptr<int> sp2(nums);
```

**shared_ptr常用操作**

```cpp
std::shared_ptr sp(new int(5));

/*重置指针*/
sp.reset(new int(10)); // 释放当前管理的对象，并接管新指针的所有权
sp.reset(); // 释放当前对象，并将up置为空

/*交换*/
std::shared_ptr sp1(new int(8));
sp.swap(sp1);
std::swap(sp,sp1);

/*检查是否为空*/
if (sp) 
if (sp != nullptr)
    
/*访问对象*/
sp[n];//拥有数组的具体化版本
sp->member;
(*up);
up.get();// get()返回裸指针，但不释放所有权

/*返回计数*/
sp.use_count();

/*检查唯一性*/
sp.unique();//唯一返回ture，反正false;
/*将unique_ptr转换为shared_ptr*/
std::unique_ptr up(new int(10));
std::shared_ptr<int> sp2 = std::move(up); // 将unique_ptr移动转换为shared_ptr
```

#### ③自定义删除器

对于特定格式的数据类型，用户可以根据自己的需求自定义删除器，只释放自己想要释放的内容

```cpp
void deleter(R *r)
{
    delete r;
    std::cout << "调用自定义删除器" << std::endl;
}
int main()
{
    /*unique_ptr自定义删除器的使用*/
    std::unique_ptr<R, decltype(deleter) *> up(new R(10), deleter);
    std::unique_ptr<R, void (*)(R *)> up1(new R(10), deleter); // 函数指针
    /*shared_ptr自定义删除器的使用*/
    //std::shared_ptr<R> sp = std::make_shared<R>(10, deleter);//错误用法make_share无法用删除器
    std::shared_ptr<R> sp1(new R(10), deleter);

    return 0;
}
```

#### ④弱智能指针：weak_ptr

`weak_ptr` 是一种不控制所指向对象生命周期的智能指针，它指向由一个或多个 `shared_ptr` 管理的对象。`weak_ptr` 不会增加引用计数，因此不会阻止所指向对象的销毁。

`weak_ptr` 无所有权，无法直接访问对象（避免对象已释放时的 “悬空引用”），**必须通过 `lock()` 方法转换为 `shared_ptr`** 后才能安全访问。

无法直接赋值`nullptr`，初始化时便已经为空，若想赋值`nullptr`直接调用`reset()`方法

`shared_ptr`可以赋值给`weak_ptr`,而`weak_ptr`无法赋值给`shared_ptr`,即`shared_ptr`可以强转为`weak_ptr`，而`weak_ptr`无法强转为`shared_ptr`

使用`weak_ptr`主要**解决循环引用问题**：当两个或多个 `shared_ptr` 相互引用时，会导致内存泄漏，导致不会销毁对象

```cpp
#include <iostream>
#include <memory>

class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed" << std::endl; }
};

class B {
public:
    std::shared_ptr<A> a_ptr; // 这里使用 shared_ptr 会导致循环引用
    ~B() { std::cout << "B destroyed" << std::endl; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a; // 循环引用
    
    std::cout << "A use count: " << a.use_count() << std::endl; // 输出: 2
    std::cout << "B use count: " << b.use_count() << std::endl; // 输出: 2
    
    return 0;
    // A 和 B 不会被销毁，因为存在循环引用
}
```

解决办法

```cpp
#include <iostream>
#include <memory>

class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed" << std::endl; }
};

class B {
public:
    std::weak_ptr<A> a_ptr; // 使用 weak_ptr 打破循环引用
    ~B() { std::cout << "B destroyed" << std::endl; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a; // 使用 weak_ptr，不会增加引用计数
    
    std::cout << "A use count: " << a.use_count() << std::endl; // 输出: 1
    std::cout << "B use count: " << b.use_count() << std::endl; // 输出: 2
    
    // 访问 weak_ptr
    if (auto shared_a = b->a_ptr.lock()) {
        std::cout << "A is still alive" << std::endl;
    } else {
        std::cout << "A has been destroyed" << std::endl;
    }
    
    return 0;
    // A 和 B 会被正确销毁
}
```

weak_ptr初始化（使用shared_ptr初始化）

```cpp
std::weak_ptr wp= std::make_shared<R>(10);
```

weak_ptr常用操作



```cpp
std::weak_ptr wp = std::make_shared<int>(10);
/*判断所指内容是否过期（被销毁）*/
/*判断是否过期*/
std::cout << wp.expired() << std::endl;
/*返回所指向share_ptr,资源过期返回空的share_ptr*/
auto wp0 = wp.lock();
/*返回引用次数*/
std::cout << wp.use_count() << std::endl;
/*重置指针*/
wp.reset(new int(10)); // 释放当前管理的对象，并接管新指针的所有权
sp.reset(); // 释放当前对象，并将up置为空

/*交换*/
std::weak_ptr wp1 = std::make_shared<int>(8);
wp.swap(wp1);
std::swap(wp,wp1);

/*检查是否为空*/
if (wp) 
if (wp != nullptr)
```

#### ⑤`share_ptr`与`weak_ptr`

- `weak_ptr` 是一种不控制所指向对象生命周期的智能指针，它指向由一个或多个 `shared_ptr` 管理的对象。`weak_ptr` 不会增加引用计数，因此不会阻止所指向对象的销毁。
- `weak_ptr` 无所有权，无法直接访问对象（避免对象已释放时的 “悬空引用”），**必须通过 `lock()` 方法转换为 `shared_ptr`** 后才能安全访问。
- 无法直接赋值`nullptr`，初始化时便已经为空，若想赋值`nullptr`直接调用`reset()`方法;而share_ptr可以直接赋值nullptr
- `shared_ptr`可以赋值给`weak_ptr`,而`weak_ptr`无法赋值给`shared_ptr`,即`shared_ptr`可以强转为`weak_ptr`，而`weak_ptr`无法强转为`shared_ptr`
- 使用`weak_ptr`主要**解决循环引用问题**：当两个或多个 `shared_ptr` 相互引用时，会导致内存泄漏，导致不会销毁对象

## 3、运算符号重载

重定义或重载大部分 C++ 内置的运算符，这样就能使用自定义类型的运算符。

| 双目算术运算符 | + (加)，-(减)，*(乘)，/(除)，% (取模)                        |
| -------------- | ------------------------------------------------------------ |
| 关系运算符     | ==(等于)，!= (不等于)，< (小于)，> (大于)，<=(小于等于)，>=(大于等于) |
| 逻辑运算符     | \|\|(逻辑或)，&&(逻辑与)，!(逻辑非)                          |
| 单目运算符     | + (正)，-(负)，*(指针)，&(取地址)                            |
| 自增自减运算符 | ++(自增)，--(自减)                                           |
| 位运算符       | \| (按位或)，& (按位与)，~(按位取反)，^(按位异或),，<< (左移)，>>(右移) |
| 赋值运算符     | =, +=, -=, *=, /= , % = , &=, \|=, ^=, <<=, >>=              |
| 空间申请与释放 | new, delete, new[ ] , delete[]                               |
| 其他运算符     | **()**(函数调用)，**->**(成员访问)，**,**(逗号)，**[]**(下标) |

下面是不可重载的运算符列表：

- **.**：成员访问运算符
- **.\***, **->\***：成员指针访问运算符
- **::**：域运算符
- **sizeof**：长度运算符
- **?:**：条件运算符
- **#**： 预处理符号

```cpp
#include <iostream>
#include <cmath>
/*运算符的重载*/
class Rectangle
{
private:
    double length;
    double breadth;

public:
    /*全参构造*/
    Rectangle(double len, double bre)
    {
        length = len;
        breadth = bre;
    }
    /*无参构造(默认有)*/
    Rectangle()
    {
        length = 0;
        breadth = 0;
    }
    void setLength(double len)
    {
        length = len;
    }
    void setBreadth(double bre)
    {
        breadth = bre;
    }
    double getLength()
    {
        return length;
    }
    double getBreadth()
    {
        return breadth;
    }
    double getArea()
    {
        return length * breadth;
    }
    /*重载运算符*/
    Rectangle operator+(Rectangle &r)
    {
        // 创建一个新矩形，其面积是两个矩形面积之和
        double totalArea = this->getArea() + r.getArea();

        // 保持长宽比例与当前矩形相同
        double ratio = length / breadth;
        double newBreadth = std::sqrt(totalArea / ratio);
        double newLength = totalArea / newBreadth;

        return Rectangle(newLength, newBreadth);
    }
};

int main()
{
    Rectangle R1 = Rectangle(8, 6);
    Rectangle R2 = Rectangle(7, 2);
    /*执行重载后的操作*/
    Rectangle R3 = R1 + R2;
    std::cout << "this Rectangle's length is " << R3.getLength() << std::endl;
    std::cout << "this Rectangle's breadth is " << R3.getBreadth() << std::endl;
    return 0;
}
```

 运算符重载实例

| 序号 | 运算符和实例                                                 |
| :--- | :----------------------------------------------------------- |
| 1    | [一元运算符重载](https://www.runoob.com/cplusplus/unary-operators-overloading.html) |
| 2    | [二元运算符重载](https://www.runoob.com/cplusplus/binary-operators-overloading.html) |
| 3    | [关系运算符重载](https://www.runoob.com/cplusplus/relational-operators-overloading.html) |
| 4    | [输入/输出运算符重载](https://www.runoob.com/cplusplus/input-output-operators-overloading.html) |
| 5    | [++ 和 -- 运算符重载](https://www.runoob.com/cplusplus/increment-decrement-operators-overloading.html) |
| 6    | [赋值运算符重载](https://www.runoob.com/cplusplus/assignment-operators-overloading.html) |
| 7    | [函数调用运算符 () 重载](https://www.runoob.com/cplusplus/function-call-operator-overloading.html) |
| 8    | [[下标运算符 [\] 重载]](https://www.runoob.com/cplusplus/subscripting-operator-overloading.html) |
| 9    | [类成员访问运算符 -> 重载](https://www.runoob.com/cplusplus/class-member-access-operator-overloading.html) |

## 4、“::”的妙用

（1）**访问命名空间中的成员**：用于指定一个标识符属于哪个命名空间。

```cpp
std::cout << "Hello"; // 使用std命名空间中的cout
```

（2）**访问类的静态成员**：用于访问类的静态成员，因为静态成员属于类而不是对象。

```cpp
class MyClass {
public:
    static int static_var;
};
int MyClass::static_var = 10; // 定义并初始化静态成员
```

（3）**在类外部定义成员函数**：当成员函数在类外部定义时，需要使用作用域解析运算符来指定该函数属于哪个类。

```cpp
class MyClass {
public:
    void myFunction(); // 声明
};
void MyClass::myFunction() { // 定义
    // 函数体
}
```

（4）**访问全局变量**：当局部作用域中有一个与全局变量同名的变量时，可以使用作用域解析运算符（前面不加任何命名空间）来访问全局变量。

```cpp
int var = 20;
int main() {
    int var = 30;
    cout << var;        // 输出30，局部变量
    cout << ::var;      // 输出20，全局变量
}
```

（5）**在多重继承中解决二义性**：当多个基类中有相同名称的成员时，可以使用作用域解析运算符指定从哪个基类中访问。

```cpp
class Base1 {
public:
    void func() { cout << "Base1"; }
};
class Base2 {
public:
    void func() { cout << "Base2"; }
};
class Derived : public Base1, public Base2 {
public:
    void callFunc() {
        Base1::func(); // 明确调用Base1的func
        Base2::func(); // 明确调用Base2的func
    }
};
```

（6）**嵌套类的访问**：当有一个类嵌套在另一个类中时，使用作用域解析运算符来访问嵌套类。

```cpp
class Outer {
public:
    class Inner {
    public:
        static void message() { cout << "Inner"; }
    };
};
Outer::Inner::message(); // 访问嵌套类的静态成员
```

（7）**访问枚举值**：在C++11中引入了有作用域的枚举（enum class），访问其枚举值必须使用作用域解析运算符。

```
enum class Color { Red, Green, Blue };
Color c = Color::Red;
```

（8）**指向成员函数的指针**：当使用指向成员函数的指针时，需要用到作用域解析运算符来指定该成员函数属于哪个类。

```cpp
class MyClass {
public:
    void func() {}
};
void (MyClass::*ptr)() = &MyClass::func;
```

## 5、模版

模板是泛型编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

模板是创建泛型类或函数的蓝图或公式。库容器，比如迭代器和算法，都是泛型编程的例子，它们都使用了模板的概念。

每个容器都有一个单一的定义，比如 **向量**，我们可以定义许多不同类型的向量，比如 **vector <int>** 或 **vector <string>**。

### (1)自动推导类型

**auto关键字：**由auto声明的变量可根据赋予的值自动推导出类型

```cpp
#include <iostream>
int main()
{
    auto a = 1;
    auto b = "a";
    auto c = {1, 2, 3, 4, 5};
    std::cout << "a's type is " << typeid(a).name() << std::endl;
    std::cout << "b's type is " << typeid(b).name() << std::endl;
    std::cout << "c's type is " << typeid(c).name() << std::endl;
    return 0;
}
```

```powershell
a's type is i
b's type is PKc
c's type is St16initializer_listIiE
```

自动推导函数指针

```cpp
#include<iostream>
int add(int a, int b, int c)
{
    return a + b + c;
}
int main(){
    
    auto pf = add;
    std::cout<<"pf(1,2,3) = "<<pf(1,2,3)<<std::endl;
    
    return 0;
}
```

```cpp
pf(1,2,3) = 6
```

### (2)函数模版

单个数据类型的函数模版

```cpp
template <typename T> // 声明数据类型

void Swap(T &a, T &b) // 在函数中的使用
{
    T temp = a;
    a = b;
    b = temp;
}

```

多个数据类型的函数模版

```cpp
template <typename T, typename Y> // 不同类型函数模版

void Message(T number, Y message)
{
    std::cout << number << ":" << message << std::endl;
}
```

类的成员函数模版(不能是析构函数和虚函数)

```cpp
template <typename C>
class tempClass
{

protected:
    C data;

public:
    tempClass(C d)
    {
        this->data = d;
    }
    void setData(C d)
    {
        this->data = d;
    }
    C getData()
    {
        return this->data;
    }
};
```

函数模版覆写与函数具体化(具体化优先级高于覆写)

```cpp
/*函数模版*/
template <typename T> // 声明数据类型

void Swap(T &a, T &b) // 单个数据类型函数模版
{
    T temp = a;
    a = b;
    b = temp;
}
/*函数模版覆写*/
template <typename T>
void Swap(tempClass<T> &a, tempClass<T> &b)
{

    T temp = a.getData();
    a.setData(b.getData());
    b.setData(temp);
}
/*函数模版具体化*/
template <typename T>
void Swap(int *a, int *b)
{
    int *temp;
    temp = a;
    a = b;
    b = temp;
}
```

返回值未知时的函数模版

**decltype关键字**

与auto不同的是使用auto需要执行函数，但decltype不会执行函数

```cpp
#include<iostream>
template<typename T>
auto func(T x, T y) ->decltype(x+y){
    return x+y;
}
```

*ps：分文件编写时，函数模版的具体化在头文件声明，主文件实现*

### (3)类模版

类模版与函数模版类似

```cpp
#include <iostream>
#include <string>
/*类模版*/
template <class T1, class T2>

class R
{
protected:
    T1 data1;
    T2 data2;

public:
    R(T1 d1, T2 d2)
    {
        this->data1 = d1;
        this->data2 = d2;
    }
    void setData1(T1 d1)
    {
        this->data1 = d1;
    }
    void setData2(T2 d2)
    {
        this->data2 = d2;
    }
    T1 getData1()
    {
        return this->data1;
    }
    T2 getData2()
    {
        return this->data2;
    }
};
int main()
{
    R<int, std::string> r = R<int, std::string>(1, "15684");//实例化写法
    return 0;
}
```

类模版的具体化

```cpp
template <>//类模版具体化
class R<int, std::string>
{
protected:
    int data1;
    std::string data2;

public:
    R(int d1, std::string d2)
    {
        this->data1 = d1;
        this->data2 = d2;
    }
    void setData1(int d1)
    {
        this->data1 = d1;
    }
    void setData2(std::string d2)
    {
        this->data2 = d2;
    }
    int getData1()
    {
        return this->data1;
    }
    std::string getData2()
    {
        return this->data2;
    }
};
```

类模版的继承

```cpp
class R1:public R<int,std::string>//类模版具体化继承
{
    public:
    using R<int,std::string>::R;
}

template <class T1, class T2>
class R2 : public R<T1, T2> // 类模版继承
{
public:
    using R<T1, T2>::R;
};

template <class T>//模版类继承模版参数给出的基类
class R3 : public T
{
public:
    R3(int a, std::string b) : T(a, b) {}
};
/*使用方式
R3<R1> r3 = R3<R1>(2, "58");
R3<R<int, std::string>> r4 = R3<R<int, std::string>>(3, "58445");
*/
```

## 

## 6、线程池

多线程允许程序同时执行多个任务，从而提高性能和响应能力。

***注：线程无法拷贝但可以使用移动语义转移线程所有权***

### (1)线程创建

#### ①使用 `std::thread` 类创建线程：

```cpp
void show(const std::string &str)
{
    std::cout << str << std::endl;
}

/*方式一*/
std::thread t1 = std::thread(show, "hello world!");
/*方式二*/
std::thread t2(show, "hello everybody!");
t1.join();
t2.join();
```

```txt
输出结果可能是：
1:
hello world!hello everybody!
2
hello everybody!hello world!
3:
hello world!
hello everybody!
4:
hello everybody!
hello world!
```

#### ②**创建关于类的成员函数线程**

大致格式为`std::thread t(&classname::memberFun,&classInstance,param)`

calssname:类名

memberFun：成员函数

&classInstance：类的实例指针

param：成员函数所需函数

**方式一：在类内部实现**

```cpp
#include <iostream>
#include <thread>
#include <string>
class R
{
public:
    R() {}
    void fun(const std::string &str)
    {
        std::cout << "执行R的成员函数fun:" << str << std::endl;
    }
    void startThread()
    {
        std::thread t(&R::fun, this, "内部执行");//this为该类的指针
        t.join();
    }
};
int main(){
    R r;
    r.startThread();
    return 0;
}
```

**方式二：在类的外部直接实现**

```cpp
#include <iostream>
#include <thread>
#include <string>
class R
{
public:
    R() {}
    void fun(const std::string &str)
    {
        std::cout << "执行R的成员函数fun:" << str << std::endl;
    }
    void startThread()
    {
        std::thread t(&R::fun, this, "内部执行");
        t.join();
    }
};
int main(){
    R r;
    std::thread t1(&R::fun, &r, "外部执行方式二");
    t1.join();
    return 0;
}
```

**方式三：使用`std::bind`绑定对象和参数**

```cpp
#include <iostream>
#include <thread>
#include <string>
#include <functional> // 用于std::bin
/*线程创建关于类的成员函数*/
class R
{
public:
    R() {}
    void fun(const std::string &str)
    {
        std::cout << "执行R的成员函数fun:" << str << std::endl;
    }
    void startThread()
    {
        std::thread t(&R::fun, this, "内部执行");
        t.join();
    }
};
int main()
{
    R r;
    auto bindFun = std::bind(&R::fun, &r, "外部执行方式三");
    std::thread t2(bindFun);
    t2.join();

    return 0;
}
```



#### ③**创建线程时的异常处理**

下面的线程函数`func`接受一个int的引用，但在创建线程时，传递的是`num`的值（即按值传递）。因此，线程函数实际上操作的是**原始数据的副本**，而不是引用。这会导致编译错误，因为线程函数期望的是一个引用，但传递的是一个值。

要传递引用，可以使用`std::ref`来包装参数，以确保按引用传递，并且`std::ref`所包装的值不能是临时参数。

```cpp
#include <iostream>
#include <thread>

void func(int &x)
{
    x++;
}

int main()
{
    int num = 1;
    std::thread t1 = std::thread(func, std::ref(num);
    t1.join();
    return 0;
}
```





### (2)线程常用方法

#### ①join()

等待线程完成,阻塞当前线程，直到目标线程执行完毕,不使用否则主线程执行完毕可能提前结束该线程

```cpp
std::thread t();
t.join();
```

#### ②detach()

分离线程，让线程在后台独立运行（无法再控制），主线程结束该线程不受主线程影响继续运行

```cpp
std::thread t();
t.decach();
```

#### ③joinable()

判断线程是否正在执行且可被join/detach,返回bool值

```cpp
std::thread t();
std::cout<<std::boolalpha<<t.joinable()<<std::endl;
```

#### ④get_id()

每个线程有唯一标识符，可通过 `get_id()` 获取

```
std::thread t();
std::cout<<t.get_id()<<std::endl;
```

### (3)线程同步

#### ①互斥锁（mutex）

**互斥锁**是一种同步原语，用于防止多个线程同时访问共享资源。当一个线程需要访问共享资源时，它首先需要锁定（lock）互斥量。如果互斥量已经被其他线程锁定，那么请求锁定的线程将被阻塞，直到互斥量被解锁（unlock）。

`std::mutex` 是 C++ 标准库中用于实现线程同步的基本工具，它提供了互斥访问的机制，确保多个线程不会同时访问共享资源，从而防止数据竞争和不一致的状态。

**互斥锁的声明与使用**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
/*互斥锁-mutex*/

std::mutex m; // 声明互斥锁
void selfAdd(int *a)
{
    m.lock(); // 上锁
    /*临界区*/
    (*a)++;
    m.unlock(); // 解锁
}
void func(int *a)
{
    for (int i = 0; i < 100000; i++)
        selfAdd(a);
}

int main()
{
    int *num = new int(0);
    std::thread t1(func, num);
    std::thread t2(func, num);
    t1.join();
    t2.join();
    std::cout << *num << std::endl;

    return 0;
}
```

#### ②`lock_guard`与`unique_lock`

**`std::lock_guard`**

手动调用 `lock()` 和 `unlock()` 容易出错（比如忘记解锁导致死锁）,推荐使用 RAII（资源获取即初始化）方式,可以使用最简单的 RAII 包装器`lock_guard`,构造时加锁，析构时自动解锁。

**原理：**在函数局部作用域，`lock_guard`再被构造时加锁，函数执行完毕后会自动调用`lock_guard`的析构函数进行解锁。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
std::mutex m;
void func_lock_guard(int *a)
{
    std::lock_guard<std::mutex> lock(m); // 初始化调用构造函数加锁
    // std::lock_guard<std::mutex> lock(m,std::adopt_lock);//不加锁
    /*临界区*/
    (*a)++;
    std::cout << "执行func_lock_guard\n";
    /*离开作用于lock调用析构函数解锁*/
}

```

**`std::unique_lock`**

可以延迟锁定、手动解锁和**转移所有权**(移动语义)；当互斥锁所有权在其他线程时，会尝试延迟锁定。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
std::mutex m;
std::timed_mutex tm; // 允许时间操作的互斥锁
void func_unique_lock(int *a)
{
    std::unique_lock<std::mutex> lock(m);
    // std::unique_lock<std::mutex> lock(m,std::defer_lock);//不加锁
    (*a)++;
    std::cout << "执行func_unique_lock\n";
}
void func_unique_lock_time(int *a)
{
    std::unique_lock<std::timed_mutex> lock(tm);
    (*a)++;
    lock.unlock();

    lock.lock();
    (*a)++;
    lock.unlock();

    lock.try_lock();
    (*a)++;
    lock.unlock();

    lock.try_lock_for(std::chrono::seconds(2)); // 在指定时间内获取到锁则返回true，反之flase
    (*a)++;
    lock.unlock();
    std::cout << "执行func_unique_lock_time\n";
}
```

#### ③call_once在类的单例中的使用

常规方式实现线程安全的类单例时使用`call_once`更加方便

**常规双检锁实现**

```cpp
class ThreadSafeSingleton
{
private:
    static ThreadSafeSingleton *instance;
    static std::mutex mtx;

    ThreadSafeSingleton()
    {
        std::cout << "ThreadSafeSingleton initialized" << std::endl;
    }

    ThreadSafeSingleton(const ThreadSafeSingleton &) = delete;
    ThreadSafeSingleton &operator=(const ThreadSafeSingleton &) = delete;

public:
    static ThreadSafeSingleton *getInstance()
    {
        // 第一次检查：避免每次调用都加锁
        if (instance == nullptr)
        {
            std::lock_guard<std::mutex> lock(mtx);
            // 第二次检查：确保只有一个线程创建实例
            if (instance == nullptr)
            {
                instance = new ThreadSafeSingleton();
            }
        }
        return instance;
    }

    void doSomething()
    {
        std::cout << "Doing something with ThreadSafeSingleton" << std::endl;
    }

    static void destroy()
    {
        std::lock_guard<std::mutex> lock(mtx);
        if (instance != nullptr)
        {
            delete instance;
            instance = nullptr;
        }
    }
};
// 初始化静态成员变量
ThreadSafeSingleton *ThreadSafeSingleton::instance = nullptr;
std::mutex ThreadSafeSingleton::mtx;

```

**call_once实现**

```cpp
class CallOnceSingleton
{
private:
    static CallOnceSingleton *instance;
    static std::once_flag flag;
    CallOnceSingleton()
    {
        std::cout << "CallOnceSingleton initialized" << std::endl;
    };
    CallOnceSingleton(const CallOnceSingleton &c) = delete;
    CallOnceSingleton &operator=(const CallOnceSingleton &c) = delete;

    static void createInstance()
    {
        instance = new CallOnceSingleton();
    }

public:
    static CallOnceSingleton *getInstance()
    {
        std::call_once(flag, createInstance);
        return instance;
    }
    void doSomething()
    {
        std::cout << "Doing something with CallOnceSingleton" << std::endl;
    }
    static void destroy()
    {
        if (instance != nullptr)
        {
            delete instance;
            instance = nullptr;
        }
    }
};
CallOnceSingleton *CallOnceSingleton::instance = nullptr;
std::once_flag CallOnceSingleton::flag;
```

### (4)条件变量 (Condition Variable)

使用 `std::condition_variable` 进行线程间通信

#### **①基本用法**

`cv.wait(lock,predicate);`

`cv.wait_for(lock,time_clock,predicate);`

其中`predicate`为可选项，通常为`Lambda`表达式，返回布尔值，开发者可据此增加线程唤醒的附加条件;

`lock`为互斥锁;

`time_clock`为时间，`std::chrono` 类型;

#### **②以生产者消费者问题为例使用条件变量Condition Variable进行线程通信**

头文件以及全局变量

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <fstream>
#include <string>
/*条件变量 (Condition Variable)-以生产者消费者问题为例*/
std::ofstream outfile("data.txt");//日志文件
std::queue<int> buffer;//任务池
std::queue<std::string> strBuffer;//日志池
std::mutex m;//互斥锁
std::condition_variable cv_producer, cv_consumer;//条件变量，cv_producer用于阻塞和唤醒生产者，cv_consumer同理消费者
bool finished = false;//生产者生产任务完成标志
const int max_size = 10000;//任务池最大容纳任务数
int au = 0;//多个生产者时标志所有任务是否完成
```

生产者

```cpp
void producer(int count)
{
    {
        std::unique_lock<std::mutex> lg(m);
        ++au;
    }

    for (int i = 0; i < count; ++i)
    {

        std::unique_lock<std::mutex> lg(m);
        cv_producer.wait(lg, []()
                         { return buffer.size() < max_size; }); // 当buffer不满时且被唤醒时执行以下代码
        /*生产商品*/
        buffer.push(i);
        /*----------写文件----------*/
        std::string line;
        line.reserve(sizeof('生') * 18);
        line.append("producer : 生产了");
        line.append(std::to_string(i));
        line.append("号商品\n");
        strBuffer.push(line);
        if (outfile.is_open() && i % 20 == 0)
        {

            while (!strBuffer.empty())
            {
                outfile << strBuffer.front();
                strBuffer.pop();
            }
        }
        /*------------------------------*/

        lg.unlock();
        cv_consumer.notify_one(); // 唤醒一个消费者
    }

    {
        std::unique_lock<std::mutex> lg(m);
        if (!--au)
        {
            finished = true;
            std::cout << "生产者生成完成！" << std::endl;
        }
    }
    cv_consumer.notify_all();
}
```

消费者

```cpp
void consumer(int id)
{
    int i = 0;

    while (true)
    {

        std::unique_lock<std::mutex> lg(m);

        cv_consumer.wait(lg, []
                         { return !buffer.empty() || finished; }); // 当buffer不为空或者生产任务已完成且被唤醒时执行以下代码
        if (buffer.empty() && finished)
            break;
        /*处理商品*/
        if (!buffer.empty())
        {
            ++i;
            std::string line;
            line.reserve(sizeof('生') * 18);
            line.append("consumer ");
            line.append(std::to_string(id));
            line.append(" : 消费了");
            line.append(std::to_string(buffer.front()));
            line.append("号商品\n");
            strBuffer.push(line);
            buffer.pop();
            if (outfile.is_open() && i % 20 == 0)
            {
                while (!strBuffer.empty())
                {
                    outfile << strBuffer.front();
                    strBuffer.pop();
                }
            }
        }
        lg.unlock();
        cv_producer.notify_one(); // 通知所有生产者
        // std::this_thread::sleep_for(std::chrono::milliseconds(50));//休眠50ms
    }
    std::cout << "consumer " << id << " : 消费完成！\n"
              << std::endl;
}
```

运行代码

```cpp
int main()
{
    auto start = std::chrono::system_clock::now();

    std::thread t1(producer, 1000);
    std::thread t6(producer, 1000);

    std::thread t2(consumer, 0);
    std::thread t3(consumer, 1);
    std::thread t4(consumer, 2);
    std::thread t5(consumer, 3);

    t1.join();
    t2.join();
    t3.join();
    t4.join();
    t5.join();
    t6.join();

    auto end = std::chrono::system_clock::now();
    std::chrono::duration<double> elapsed_seconds = end - start;
    /*处理多余缓冲区*/
    if (!strBuffer.empty())
    {
        while (!strBuffer.empty())
        {
            outfile << strBuffer.front();
            strBuffer.pop();
        }
    }
    std::cout << "总耗时: " << elapsed_seconds.count() << "s\n";
    outfile << "总耗时: " << elapsed_seconds.count() << "s\n";
    outfile.close();
    return 0;
}
```

#### **③`wait_for()`函数**

**`wait_for()`**可以使线程等待指定一段时间，在段时间内若满足条件则提前唤醒，反之线程取消等待继续执行后续代码，开发者可以依据**`wait_for()`**的返回值决定线程是否继续执行下去.

不带谓词版本的**`wait_for()`**函数会返回`std::cv_status`类型值，未超时返回`std::cv_status::no_timeout`，超时则返回`std::cv_status::timeout`

带谓词版本的`wait_for()`函数会返回`bool`类型值，未超时返回`true`，超时返回`false`

```cpp
/*非谓词版本*/
if(std::cv_status::timeout==cv.wait_for(lock,std::chrono::seconds(1)))//超时返回`std::cv_status::timeout`
    std::cout<<"timeout!"<<std::endl;
else//未超时返回std::cv_status::no_timeout
    std::cout<<"no timeout!"<<std::endl;


/*谓词版本*/
int a = 1;
int b = 2;
auto lambda=[=](){return a>b}
if(!cv.wait_for(lock,std::chrono::seconds(1),lambda))//超时返回false
    std::cout<<"timeout!"<<std::endl;
else//未超时返回ture
    std::cout<<"no timeout!"<<std::endl;

```

`wait_until()`函数会等待值指定时间，但该时间不好掌握故不常用.

### (5)异步编程

异步编程的头文件是`<future>`,里面包含了诸如std::future、std::asyns、std::promise等异步线程管理工具

#### ①异步执行函数(`std::asyns`)

`std::async` 是启动异步操作的最简单方式，它返回一个 `std::future` 对象，用于获取异步操作的结果。

**基本语法**

```cpp
std::async( Function&& f, Args&&... args );
std::async( std::launch policy, Function&& f, Args&&... args );
```

其中，f为函数，args是f所需参数，`std::lanuch policy`是启动策略主要有以下三种：

1. **`std::launch::async`** - 直接在新线程中异步执行函数，使用`get()` 或 `wait()`获取执行结果
2. **`std::launch::deferred`** - 延迟执行，直到调用 `get()` 或 `wait()` 时在当前线程执行
3. **`std::launch::async | std::launch::deferred`** - 默认策略，由实现决定

**`std::async`的简单使用**

```cpp
#include <iostream>
#include <thread>
#include <future>
/*异步编程*/
int add(int x, int y)
{
    std::cout << "task - add start!" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    return x+y
    std::cout << "task - add end!" << std::endl;
}
int main()
{
    /*std::launch::async协议的async异步函数*/
    std::cout << "std::launch::async协议" << std::endl;
    std::future<int> fu1 = std::async(std::launch::async, add, 10, 20);//用future类型接收
    std::this_thread::sleep_for(std::chrono::milliseconds(5000));
    fu1.wait(); // 获取状态是否阻塞？
    std::cout << fu1.get() << std::endl; // 获取结果
    
    /*std::launch::deferred协议*/
    std::cout << "std::launch::deferred协议" << std::endl;
    std::future<int> fu2 = std::async(std::launch::deferred, add, 10, 30);
    std::this_thread::sleep_for(std::chrono::milliseconds(5000));
    fu2.wait_for(std::chrono::milliseconds(500)); // 若阻塞等待500ms
    std::cout << fu2.get() << std::endl; // 获取结果
    return 0;
}
```

#### ②异步结果传递函数(`std::promise`)

**`std::promise`**主要用于在线程之间传递值，promise声明的变量没有拷贝函数只有移动函数，确保只有一个线程可以访问。

**`std::promise`的简单使用**

```cpp
#include <iostream>
#include <thread>
#include <future>

void add_prom(std::promise<int> &&sum, int x, int y)
{
    std::cout << "task - add_prom start!" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    sum.set_value(x + y);//promise赋值
    std::cout << "task - add_prom end!" << std::endl;
}

int main()
{

    /*std::launch::deferred协议的async传递promise值的异步函数*/
    std::promise<int> prom;
    std::future<int> fu_prom = prom.get_future();
    std::cout << "std::launch::deferred协议" << std::endl;
    std::future<void> fu2 = std::async(std::launch::deferred, add_prom, std::move(prom), 10, 30);//使用移动语义传值
    /*由于执行移动语义，此时prom已经为空*/
    std::this_thread::sleep_for(std::chrono::milliseconds(5000));
    fu2.wait_for(std::chrono::milliseconds(500)); // 若阻塞等待500ms
    fu2.get();
    std::cout << fu_prom.get() << std::endl;
}
```

#### ③`std ::share_future`

对于future类型变量，可以使用多次wait()函数，但只能执行一次get()函数，将future类型转化为share_future可以使用多次get(),便于在多个线程等待同一个异步结果

```cpp
#include <iostream>
#include <thread>
#include <future>
/*异步编程*/
int add(int x, int y)
{
    std::cout << "task - add start!" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    std::cout << "task - add end!" << std::endl;
    return x + y;
}

int main()
{
    /*std::share_future*/
    std::future<int> fu3 = std::async(std::launch::async, add, 10, 20);
    std::shared_future<int> share_fu = fu3.share();
    std::cout << share_fu.get() << std::endl;
    std::cout << share_fu.get() << std::endl;
    std::cout << share_fu.get() << std::endl;
}
```

#### ④`std::packaged_task`

`std::packaged_task` 可以将任何可调用对象包装成一个异步任务。同样，`std::packaged_task` 也没有拷贝函数只有移动函数。

`std::packaged_task` 使用实例

```cpp
#include <iostream>
#include <thread>
#include <future>

int add(int x, int y)
{
    std::cout << "task - add start!" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    std::cout << "task - add end!" << std::endl;
    return x + y;
}

int main()
{
    /*std::packaged_task的使用*/
    std::packaged_task<int(int, int)> p_task(add);
    std::future<int> fu4 = p_task.get_future();

    std::thread async_t(std::move(p_task), 10, 20);
    async_t.join();
    std::cout << fu4.get() << std::endl;
}
```

#### ⑤`wait_for()`

`std::future`类型的`wait_for()`函数同理条件变量 (Condition Variable)的`wait_for()`函数[](#③`wait_for()`函数)

只是谓词版本超时返回的是`std::future_status::timeout`,未超时返回的是`std::future_status::no_timeout`

### (6)原子操作

**原子操作**可以在多线程环境中安全地操作数据，而无需使用互斥锁。原子操作是不可中断的操作，即在执行过程中不会被其他线程打断，从而避免了数据竞争，而且比互斥锁更高效。

**原子操作**通常只适用于简单的读、写、算术和位运算等操作。对于复杂的数据结构或需要多个操作组合的场景，原子操作可能无法直接满足需求。

**原子操作**并不能解决所有的并发问题。对于复杂的同步需求，仍然需要使用锁或其他同步机制。

#### ①原子操作的常用操作

```cpp
#include <iostream>
#include <mutex>
#include <atomic>
/*初始化*/
std::atomic<int> atomic_int(0);
/*存贮值*/
atomic_int.store(15);
/*加载值*/
int load = atomic_int.load();
/*交换值*/
int exchange = atomic_int.exchange(100);
/*比较交换*/
// 如果atomic_int等于exchange，则将其设置为desired，否则将exchange设置为atomic_int的当前值
int desired = 100;
atomic_int.compare_exchange_weak(exchange, desired);
atomic_int.compare_exchange_strong(exchange, desired);
/*读-修改-写操作*/
// 加法并返回旧值
exchange = atomic_int.fetch_add(5);
// 减法并返回旧值
exchange = atomic_int.fetch_sub(5);
// 按位与并返回旧值
exchange = atomic_int.fetch_and(0x0F);
// 按位或并返回旧值
exchange = atomic_int.fetch_or(0xF0);
// 按位异或并返回旧值
exchange = atomic_int.fetch_xor(0xFF);
```

#### ②内存顺序

操作系统在执行指令时会尝试对指令进行重排，通过在atomic相关操作第二位参数填写以下内容可实现指定得到指令重排效果.

`std::memory_order_relaxed`:当前线程里，编译器和 CPU 可以随便重排这条原子操作的前后指令.
`std::memory_order_release`:保证这条原子写之前的所有普通读写操作，都不能被重排到它后面.
`std::memory_order_acquire`:保证这条原子读之后的所有普通读写操作，都不能被重排到它前面.
`std::memory_order_acq_rel`:既阻止前面的操作被重排到后面，也阻止后面的操作被重排到前面.

`std::memory_order_seq_cst`:顺序一致性，最严格的内存顺序，也是默认的内存顺序.

```cpp
#include <iostream>
#include <mutex>
#include <atomic>

std::atomic<int> atomic_int;
atomic_int.store(10,std::memory_order_acq_rel);
```



## 7、预处理器和信号处理

### (1)预处理器

### ① `#include` - 文件包含

用于包含头文件或其它源文件：

```cpp
// 包含系统头文件
#include <iostream>
#include <vector>
#include <string>

// 包含用户自定义头文件
#include "myheader.h"
#include "../includes/utils.h"
```

### ②`#define` - 宏定义

用于定义常量、函数宏或特殊宏：

```cpp
// 定义常量宏
#define PI 3.14159
#define MAX_SIZE 100
#define APPLICATION_NAME "MyApp"

// 定义函数宏
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define PRINT(msg) std::cout << msg << std::endl

// 特殊用法
#define DEBUG // 定义空宏，用于条件编译
```

### ③ `#undef` - 取消宏定义

用于取消已定义的宏：

```cpp
#define TEMP_MACRO 42
// 使用 TEMP_MACRO...
#undef TEMP_MACRO
// 这里 TEMP_MACRO 不再定义
```

### ④条件编译指令

用于根据条件包含或排除代码：

- `#if`：如果条件为真，则编译后续代码。
- `#ifdef`：如果宏已定义，则编译后续代码。
- `#ifndef`：如果宏未定义，则编译后续代码。
- `#elif`：前面的条件不满足时，检查新的条件。
- `#else`：前面的条件都不满足时，编译后续代码。
- `#endif`：结束一个条件编译块。

**头文件保护**

头文件保护是条件编译最常见的用法，防止头文件被多次包含。

```cpp
/*
当预处理器第一次遇到这个头文件时，`MYHEADER_H` 未定义，所以 `#ifndef` 条件为真，预处理器会定义 `MYHEADER_H` 并包含头文件内容。如果同一个编译单元中再次包含这个头文件，`MYHEADER_H` 已经被定义，所以条件为假，头文件内容会被跳过。
*/
// myheader.h
#ifndef MYHEADER_H
#define MYHEADER_H

// 头文件的内容

#endif // MYHEADER_H
```

**跨平台代码**

假设你正在编写一个跨平台的程序，在Windows和Linux上需要不同的代码。

```cpp
/*
预处理器会根据平台定义不同的宏（例如，在Windows上会定义 _WIN32，在Linux上会定义 __linux__），从而选择编译相应的代码。如果平台不被支持，则会产生一个编译错误。
*/
#ifdef _WIN32
    // Windows特定的代码
    #include <windows.h>
#elif __linux__
    // Linux特定的代码
    #include <unistd.h>
#else
    #error "Unsupported platform"
#endif
```

```cpp
// 基本条件编译
#ifdef DEBUG
    std::cout << "Debug mode enabled" << std::endl;
#endif

#ifndef RELEASE
    std::cout << "Not in release mode" << std::endl;
#endif

// 多条件判断
#if defined(WIN32) || defined(_WIN32)
    std::cout << "Windows platform" << std::endl;
#elif defined(__linux__)
    std::cout << "Linux platform" << std::endl;
#elif defined(__APPLE__)
    std::cout << "macOS platform" << std::endl;
#else
    std::cout << "Unknown platform" << std::endl;
#endif

// 数值比较
#if __cplusplus >= 201703L
    std::cout << "C++17 or later" << std::endl;
#else
    std::cout << "Older C++ standard" << std::endl;
#endif
```

### ⑤ `#error` - 生成错误

用于在预处理阶段生成错误消息：

```cpp
#ifndef REQUIRED_MACRO
#error "REQUIRED_MACRO must be defined"
#endif

#if __cplusplus < 201103L
#error "This code requires C++11 or later"
#endif
```

### ⑥ `#pragma` - 编译器指令

用于向编译器发送特定指令：

```cpp
// 确保头文件只被包含一次（非标准但广泛支持）
#pragma once

// 禁止特定警告（编译器特定）
#pragma warning(disable: 4996) // MSVC
#pragma GCC diagnostic ignored "-Wdeprecated-declarations" // GCC

// 内存对齐
#pragma pack(push, 1) // 1字节对齐
struct PackedStruct {
    char a;
    int b;
    short c;
};
#pragma pack(pop) // 恢复默认对齐
```

### ⑦ `#line` - 行号控制

用于改变编译器报告的行号和文件名：

```cpp
#line 100 "special_file.cpp"
// 从这里开始，编译器会报告行号从100开始，文件名为special_file.cpp

void someFunction() {
    // 如果这里有错误，编译器会报告在special_file.cpp的第103行
}

#line 200 // 只改变行号
// 现在行号从200开始
```

### (2)信号处理

**信号处理**是 C++ 中与操作系统交互的重要机制，它允许程序响应外部事件（如用户中断、程序错误等）。C++ 继承了 C 语言的信号处理机制，并提供了 `<csignal>` 头文件来支持信号处理功能。

## 8、异常处理

## 9、Lambda表达式

C++ Lambda 表达式是 C++11 引入的匿名函数机制，旨在简化代码中短小函数的定义与使用，尤其适合作为算法的谓词（predicate）或回调函数。它的核心优势是**无需单独定义函数名**，可直接在需要的地方内联编写，使代码更紧凑、逻辑更清晰。

### （1）基本语法

Lambda 表达式的完整语法结构如下：

```
[捕获列表](参数列表) mutable(可选) noexcept(可选) -> 返回类型(可选) { 函数体 }
```

各部分的作用：

- **捕获列表**：控制 Lambda 与外部作用域变量的交互（如值传递、引用传递）。

- **参数列表**：与普通函数的参数列表类似，可省略（无参数时）。

- **mutable**：可选关键字，允许修改值捕获的变量（默认值捕获变量为const）。

- **noexcept**：可选说明符，表示 Lambda 不会抛出异常（用于优化）。

- **返回类型**：可选，若函数体仅有一个return语句，编译器可自动推导，否则需显式指定。

- **函数体**：Lambda 的执行逻辑，与普通函数体一致。

### （2）核心部分详解

#### 1. 捕获列表（关键）

捕获列表用于指定 Lambda 如何访问外部作用域的变量，是 Lambda 与外部交互的核心。常见形式如下：

| 捕获方式    | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| []          | 不捕获任何外部变量                                           |
| [x]         | **值捕获**：复制变量x的副本（Lambda 内修改不影响外部）       |
| [&x]        | **引用捕获**：引用变量x（Lambda 内修改会影响外部）           |
| [=]         | **隐式值捕获**：值捕获所有 Lambda 中使用的外部变量           |
| [&]         | **隐式引用捕获**：引用捕获所有 Lambda 中使用的外部变量       |
| [=, &x]     | 除x用引用捕获外，其余变量隐式值捕获（&x需放在=后）           |
| [&, x]      | 除x用值捕获外，其余变量隐式引用捕获（x需放在&后）            |
| [this]      | 捕获当前类的this指针（类成员函数中使用，可访问成员变量 / 函数） |
| [y = x + 1] | **初始化捕获**（C++14）：捕获表达式x+1的结果，命名为y（副本） |
| [*this]     | 复制捕获当前对象（C++20）：避免this指针悬垂（如返回 Lambda 时） |

**示例：捕获方式对比**

```
#include <iostream>
using namespace std;

int main() {
    int a = 10, b = 20;

    // 1. 不捕获任何变量
    auto f1 = []() { cout << "无捕获" << endl; };
    f1(); // 输出：无捕获

    // 2. 值捕获a，引用捕获b
    auto f2 = [a, &b]() { 
        // a = 30; // 错误：值捕获变量默认是const（需加mutable）
        b = 30;   // 引用捕获可修改
        cout << "a=" << a << ", b=" << b << endl; 
    };
    f2(); // 输出：a=10, b=30（外部b已被修改）
    cout << "外部b=" << b << endl; // 输出：外部b=30

    // 3. 隐式值捕获所有使用的变量（此处为a）
    auto f3 = [=]() { cout << "隐式值捕获a=" << a << endl; };
    a = 100; // 外部修改不影响Lambda内的副本
    f3(); // 输出：隐式值捕获a=10

    // 4. 初始化捕获（C++14）：捕获a+b的结果，命名为sum
    auto f4 = [sum = a + b]() { cout << "sum=" << sum << endl; };
    f4(); // 输出：sum=130（a=100, b=30）

    return 0;
}
```

#### 2. 参数列表

与普通函数的参数列表规则基本一致，支持类型推导（C++14 起）：

- 无参数时可省略（如[] { ... }）。

- C++14 起支持**泛型参数**（auto），此时 Lambda 会被推导为带模板的闭包类型。

**示例：泛型 Lambda（C++14）**

```
// 泛型Lambda：支持任意类型的参数a和b
auto add = [](auto a, auto b) { return a + b; };
cout << add(1, 2) << endl;       // 输出：3（int）
cout << add(1.5, 2.5) << endl;   // 输出：4.0（double）
cout << add("hello", " world") << endl; // 输出：hello world（const char*）
```

#### 3. mutable 关键字

默认情况下，**值捕获的变量在 Lambda 内是****const****的，不可修改**。若需修改值捕获的变量，需加mutable（但修改仅影响副本，不影响外部变量）。

**示例：mutable 的作用**

```
int x = 5;
// 不加mutable：值捕获的x是const，无法修改
// auto f = [x]() { x++; }; // 错误

// 加mutable：允许修改值捕获的副本
auto f = [x]() mutable { 
    x++; 
    return x; 
};
cout << f() << endl; // 输出：6（副本被修改）
cout << x << endl;   // 输出：5（外部x不变）
```

#### 4. 返回类型

若函数体仅有一个return语句，编译器可自动推导返回类型，此时-> 返回类型可省略；若有多个return语句或类型复杂，需显式指定。

**示例：返回类型推导与显式指定**

```
// 单个return：自动推导为int
auto f1 = []() { return 10; };

// 多个return，类型不同：需显式指定返回类型
auto f2 = []() -> double { 
    if (true) return 1; 
    else return 2.5; 
};
cout << f2() << endl; // 输出：1.0（double）
```

### （3）Lambda 的类型与存储

Lambda 表达式的类型是**匿名的闭包类型（closure type）**，每个 Lambda 的类型唯一，无法直接通过类型名声明变量。但可通过以下方式存储或传递：

- 用auto自动推导（推荐，效率最高）。

- 用std::function（需包含<functional>）存储（适合作为函数参数或容器元素）。

**示例：Lambda 的存储与传递**

```
#include <functional>
#include <vector>

// 用std::function接收Lambda
void callFunc(std::function<void()> func) {
    func();
}

int main() {
    // 1. auto存储Lambda
    auto print = []() { cout << "Lambda" << endl; };
    print(); // 输出：Lambda

    // 2. 存储到std::function容器
    std::vector<std::function<int(int)>> funcs;
    funcs.push_back([](int x) { return x * 2; });
    funcs.push_back([](int x) { return x + 10; });
    cout << funcs[0](5) << endl; // 输出：10
    cout << funcs[1](5) << endl; // 输出：15

    // 3. 作为参数传递
    callFunc([]() { cout << "作为参数" << endl; }); // 输出：作为参数

    return 0;
}
```

### （4）C++ 版本对 Lambda 的扩展

Lambda 在后续 C++ 标准中不断增强：

- **C++14**：支持泛型参数（auto）、初始化捕获（[y = x + 1]）。

- **C++17**：支持constexpr Lambda（可在编译期执行）、Lambda 捕获*this（复制当前对象）。

- **C++20**：支持模板 Lambda（显式模板参数，如[]<typename T>(T t) { ... }）、Lambda 作为非类型模板参数。

**示例：constexpr Lambda（C++17）**

```
// constexpr Lambda：可在编译期计算
constexpr auto square = [](int x) { return x * x; };
constexpr int s = square(5); // 编译期计算，s=25（无需运行时开销）
cout << s << endl; // 输出：25
```

### （5）常见使用场景

Lambda 最常用于简化 STL 算法的谓词或操作：

- 排序（std::sort）、遍历（std::for_each）、查找（std::find_if）等。

**示例：STL 算法中使用 Lambda**

```
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};

    // 1. 排序：降序（默认是升序）
    std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
    // 此时v = {5,4,3,1,1}

    // 2. 遍历：打印所有元素
    std::for_each(v.begin(), v.end(), [](int x) { cout << x << " "; });
    // 输出：5 4 3 1 1 

    // 3. 查找：第一个大于3的元素
    auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 3; });
    if (it != v.end()) cout << "\n找到：" << *it << endl; // 输出：找到：5

    return 0;
}
```

### （6）注意事项

1. **引用捕获的生命周期**：若 Lambda 的生命周期超过引用捕获变量的生命周期，会导致悬垂引用（未定义行为）。

```
auto badFunc() {
    int x = 10;
    return [&x]() { return x; }; // 错误：x在函数返回后销毁，Lambda引用无效
}
```

1. **重复捕获**：同一变量不能被重复捕获（如[x, x]是错误的）。

1. **默认参数**：Lambda 不支持默认参数（如[](int x = 0) { ... }是错误的）。

# 四、常用库函数

## 1、容器类库



### 前言

C++ 标准库中的容器类（Containers）主要用于存储和管理对象集合，主要有以下分类

#### ①**序列容器**

**特点**：元素按照插入顺序存储，位置由插入顺序决定（而非元素值），支持通过索引或迭代器访问元素。

- `vector`：动态数组，元素连续存储，尾部插入 / 删除效率高（O (1)），中间插入 / 删除效率低（O (n)），支持随机访问。
- `deque`：双端队列，元素分段连续存储，两端插入 / 删除效率高（O (1)），支持随机访问。
- `list`：双向链表，元素不连续存储，任意位置插入 / 删除效率高（O (1)），但不支持随机访问（只能双向遍历）。
- `forward_list`：单向链表（C++11 新增），比`list`更节省空间，仅支持单向遍历，任意位置插入 / 删除效率高（O (1)）。
- `array`：固定大小数组（C++11 新增），封装了原生数组，大小编译期确定，支持随机访问，比原生数组更安全。

#### ②**关联容器**

**特点**：元素按 “键（key）” 排序存储（默认升序），底层通常基于红黑树实现，查找、插入、删除操作效率为 O (log n)，不支持随机访问。

- `set`：存储唯一键（key 即元素），元素不可重复，自动排序。
- `multiset`：与`set`类似，但允许键重复（支持多个相同元素）。
- `map`：存储键值对（key-value），键唯一，自动按 key 排序，可通过 key 快速查找 value。
- `multimap`：与`map`类似，但允许键重复（一个 key 可对应多个 value）。

#### ③**无序关联容器**

**特点**：C++11 新增，元素不排序，底层基于哈希表实现，查找、插入、删除的平均效率为 O (1)（最坏 O (n)），适合需要快速查找但不关心顺序的场景。

- `unordered_set`：存储唯一键，不排序，基于哈希表。
- `unordered_multiset`：与`unordered_set`类似，允许键重复。
- `unordered_map`：存储键值对，键唯一，不排序，基于哈希表。
- `unordered_multimap`：与`unordered_map`类似，允许键重复。

#### ④**容器适配器**

**特点**：不直接存储元素，而是对现有容器（序列容器）进行封装，提供特定的接口（如栈、队列），隐藏底层容器的部分功能。

包含的容器：

- `stack`：栈，遵循 “后进先出（LIFO）”，默认基于`deque`实现，仅支持从顶部插入 / 删除 / 访问元素。
- `queue`：队列，遵循 “先进先出（FIFO）”，默认基于`deque`实现，仅支持从尾部插入、头部删除。
- `priority_queue`：优先队列，每次出队的是 “优先级最高” 的元素（默认最大元素），默认基于`vector`实现，底层通过堆结构维护优先级。

### 序列容器

### （1）动态数组`vector`

`vector` 使用的是**单一、连续的内存块**，`vector` 实现动态数组是因为内部维护着一个指向连续内存块的指针，当这个内存块的空间不够时，`vector` 会申请一块**更大**的新内存，将旧内存中的所有元素**复制或移动**到新内存中，然后释放旧的内存块。

这并不意味着每添加或每删除一个元素都需要重新申请新内存块，一个 `std::vector` 对象内部至少包含三个关键的指针（或类似指针的成员）,一个指向第一个元素（`begin()`），一个指向最后一个元素的后面一个位置（`end()`），还有一个指向内存块的末尾。每当添加末尾添加新元素时，直接将`end()`位置内存赋值，然后`end()`指向下一个位置。

```cpp
#include <iostream>
#include <vector>
#include <string>
/*函数库-vector*/
class R
{
private:
    std::string str;

public:
    R(const std::string &s) : str(s) {}

    std::string &getStr()
    {
        return str;
    }
};

```



#### ①初始化

```cpp
std::vector<R> vr;
```

#### ②插入元素

```cpp
/*添加新元素*/
// 尾部插入元素
// 调用移动函数
vr.push_back(R("0"));
// 调用构造函数
vr.emplace_back("1");
// 任意位置插入元素
vr.insert(vr.begin() + 2, R("2"));
```

#### ③删除元素

```cpp
/*删除元素*/
// 删除尾部元素
vr.pop_back();
// 删除指定位置元素
vr.erase(vr.end());
```

#### ④访问元素

```cpp
/*访问元素*/
// 不检查边界
std::cout << "第一个元素:" << vr[0].getStr() << std::endl;
// 检查边界
std::cout << "第二个元素:" << vr.at(1).getStr() << std::endl;
// 首位元素
std::cout << "首位元素:" << vr.front().getStr() << std::endl;
// 尾部元素
std::cout << "尾部元素:" << vr.back().getStr() << std::endl;
```

#### ⑤遍历元素

```cpp
/*遍历*/
// 迭代器遍历
std::cout << "使用迭代器: ";
for (auto i = vr.begin(); i != vr.end(); i++)
    std::cout << (*i).getStr() << " ";
std::cout << std::endl;
// 范围for循环(c++11)
std::cout << "范围for循环: ";
for (R i : vr)
    std::cout << i.getStr() << " ";
std::cout << std::endl;
```

### （2）双端队列`deque`

与`vector`相比`deque` 使用的是**多个不连续的、固定大小的内存块（通常称为 "缓冲区" 或 "chunk"）**，并通过一个**中央指针数组（或索引数组）** 来管理这些内存块。

每当添加的元素填满已有内存块后，就会申请新的内存块，并添加新的指针管理新内存块，因此`deque`的迭代对比`vector`要麻烦的多

注:`deque`没有`capacity()`和`reserve()`方法，与`vector`不同

#### ①初始化

```cpp
/*初始化*/
std::deque<int> dq1;                   // 空deque
std::deque<std::string> dq2;           // 字符串deque
std::deque<double> dq3(10);            // 包含10个默认初始化的double
std::deque<int> dq4(5, 42);            // 包含5个值为42的int
std::deque<int> dq5 = {1, 2, 3, 4, 5}; // 初始化列表 (C++11)
```

#### ②插入元素

```cpp
/*插入元素*/
std::deque<int> dq_int;
// 头部操作
dq_int.push_front(0);    // 头部添加
dq_int.emplace_front(1); // 头部构造添加
// 尾部
dq_int.push_back(2);    // 头部添加
dq_int.emplace_back(4); // 头部构造添加
// 指定位置插入
//  在指定位置插入单个元素
auto it = dq_int.begin() + 1;
dq_int.insert(it, 20); // 在it位置之前插入20

// 在指定位置插入多个相同元素
it = dq_int.begin() + 2;
dq_int.insert(it, 3, 25); // 插入3个25

// 在指定位置插入范围
std::vector<int> vec = {35, 36, 37};
it = dq_int.end() - 1;
dq_int.insert(it, vec.begin(), vec.end()); // 将vec元素插入dq_int

// 在指定位置原地构造元素 (C++11)
it = dq_int.begin() + 1;
dq_int.emplace(it, 15);
```

#### ③删除元素

```cpp
/*删除元素*/
dq_int.pop_front(); // 删除头
dq_int.pop_back();  // 删除尾
// 删除指定位置的元素
it = dq_int.begin() + 2;
it = dq_int.erase(it); // 删除it位置元素，it现在指向原来it位置的元素

// 删除范围内的元素
auto first = dq_int.begin() + 1;
auto last = dq_int.begin() + 4;
dq_int.erase(first, last); // 删除1-4位置的元素
```



#### ④访问元素

```cpp
/*访问元素*/
// 使用下标操作符（不检查边界）
std::cout << "q_int[2] = " << dq_int[2] << std::endl; // 30

// 使用at()方法（带边界检查）
std::cout << "dq_int.at(3) = " << dq_int.at(3) << std::endl; // 40
try
{
    std::cout << dq_int.at(10) << std::endl; // 抛出std::out_of_range异常
}
catch (const std::out_of_range &e)
{
    std::cerr << "Out of range: " << e.what() << std::endl;
}
```

#### ⑤遍历元素

```cpp
for (auto it = dq_int.begin(); it != dq_int.end(); ++it) // 迭代器
    {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    for (auto &it : dq_int) // 引用范围for循环
    {
        std::cout << it << " ";
    }
    std::cout << std::endl;
```



### （3）双向链表`list`

### （4）单向链表`forward_list`

### （5）固定大小数组`array`

### 有序关联容器

### （1）`set`

存储唯一键（key 即元素），元素不可重复，自动排序。

### （2）`multiset`

与`set`类似，但允许键重复（支持多个相同元素）。

### （3）`map`

注：存储键值对（key-value），键唯一，自动按 key 排序，可通过 key 快速查找 value。

`std::map` 是 C++ 标准模板库 (STL) 中的一个关联容器，它存储键值对 (key-value pairs)，并且按键的升序自动排序。底层通常实现为红黑树（一种自平衡的二叉搜索树）。

#### ①初始化

```
std::map<std::string, int> map;
```

#### ②插入元素

**使用 `insert()` 成员函数**

`insert` 方法在键不存在时插入，返回一个 `pair<iterator, bool>`，其中 `bool` 表示插入是否成功。

```cpp
std::map<std::string, int> map;

// 方法 1: 使用 make_pair
auto result1 = map.insert(std::make_pair("Alice", 25));
// result1.first 是指向元素的迭代器，result1.second 是 bool (true)

// 方法 2: 使用花括号初始化 (C++11)
auto result2 = map.insert({"Bob", 30});

// 方法 3: 使用 pair 的构造函数
auto result3 = map.insert(std::pair<const std::string, int>("Charlie", 35));

// 检查插入是否成功
if (result2.second) {
    std::cout << "Insertion of Bob succeeded.\n";
}
```

**使用 `emplace()` 成员函数 (C++11)**

`emplace` 直接在容器内构造元素，避免创建临时对象，通常更高效。

```cpp
// 原地构造，传入构造键和值所需的参数
auto result = map.emplace("David", 40);
// 等同于 map.insert({"David", 40})，但可能更高效
```

**使用下标操作符 `[]`**

这是最直观的插入和更新方式。

```cpp
// 插入新键值对
map["Eve"] = 45;

// 更新已存在的键对应的值
map["Alice"] = 26; // 现在 Alice 的值是 26

// 注意：如果键不存在，operator[] 会值初始化一个值并插入
// 对于 int，初始化为 0；对于 string，初始化为空字符串等。
int value = map["Unknown"]; // 插入 {"Unknown", 0}，并返回 0
```

**使用 `try_emplace()` 成员函数 (C++17)**

更安全和高效的插入方法。如果键已存在，**不构造**值对象。

```cpp
std::string key = "Alice";
// 只有当 key 不存在时，才会构造 std::string(5, 'A') 这个值
auto [iter, success] = map.try_emplace(key, 5, 'A'); // 尝试插入 {"Alice", "AAAAA"}
// 如果 key 已存在，则不会发生任何构造，success 为 false
```

#### ③删除元素

**使用 `erase()` 成员函数**

```cpp
// 方法 1: 通过键删除。返回删除的元素个数（0 或 1）
size_t count = map.erase("Bob"); // count 为 1

// 方法 2: 通过迭代器删除。返回被删除元素之后元素的迭代器
auto it = map.find("Charlie");
if (it != map.end()) {
    umap.erase(it); // 删除 Charlie
}

// 方法 3: 通过迭代器范围删除 (C++11)
// map.erase(start_iterator, end_iterator);
```

#### ④访问元素

**使用下标操作符 `[]`（不推荐）**

```cpp
// 访问存在的键
std::cout << map["Alice"]; // 输出: 26

// 访问不存在的键（危险！会插入新元素）
std::cout << map["Ghost"]; // 插入 {"Ghost", 0} 并输出 0
```

**使用 `at()` 成员函数**

**安全访问**。如果键不存在，抛出 `std::out_of_range` 异常。

```cpp
try {
    std::cout << map.at("Alice"); // 输出: 26
    std::cout <<umap.at("Nonexistent"); // 抛出 std::out_of_range
} catch (const std::out_of_range& e) {
    std::cerr << "Key not found: " << e.what() << '\n';
}
```

**使用 `find()` 成员函数**

**最常用且安全的访问方法**。返回一个迭代器，如果键不存在则返回 `end()`。

```cpp
auto it = map.find("Bob");
if (it != map.end()) {
    // it->first 是键，it->second 是值
    std::cout << "Found: " << it->first << " -> " << it->second << '\n';
} else {
    std::cout << "Key Bob not found.\n";
}
```

#### ⑤边界查找

```cpp
std::map<int, std::string> numMap = {
    {10, "ten"},
    {20, "twenty"},
    {30, "thirty"},
    {40, "forty"}
};

// lower_bound - 第一个不小于key的元素
auto lb = numMap.lower_bound(25); // 指向30
std::cout << lb->first << ": " << lb->second << std::endl; // 输出: 30: thirty

// upper_bound - 第一个大于key的元素
auto ub = numMap.upper_bound(25); // 也指向30
std::cout << ub->first << ": " << ub->second << std::endl; // 输出: 30: thirty

// equal_range - 返回匹配键的范围（对于map，范围最多一个元素）
auto range = numMap.equal_range(20);
for (auto it = range.first; it != range.second; ++it) {
    std::cout << it->first << ": " << it->second << std::endl; // 输出: 20: twenty
}
```

#### ⑥遍历

```cpp
// 方法 1: 使用迭代器
for (auto it = map.begin(); it != map.end(); ++it) {
    std::cout << it->first << ": " << it->second << '\n';
}

// 方法 2: 基于范围的 for 循环 (C++11)
for (const auto& pair : map) {
    std::cout << pair.first << ": " << pair.second << '\n';
}

// 方法 3: 使用结构化绑定 (C++17) - 最清晰
for (const auto& [key, value] : map) {
    std::cout << key << ": " << value << '\n';
}
```



#### ⑦自定义比较函数

默认情况下，`std::map` 使用 `std::less<Key>` 对键进行排序。你可以提供自定义比较函数：

```cpp
// 自定义比较函数：按键的降序排序
struct CompareDescending {
    bool operator()(const int& a, const int& b) const {
        return a > b; // 降序
    }
};

std::map<int, std::string, CompareDescending> descMap = {
    {1, "one"},
    {3, "three"},
    {2, "two"}
};

// 遍历将按 3, 2, 1 的顺序
for (const auto& [key, value] : descMap) {
    std::cout << key << ": " << value << std::endl;
}
```



### （4）`multimap`

与`map`类似，但允许键重复（一个 key 可对应多个 value）。

### 无序关联容器

### （1）`unordered_set`

存储唯一键，不排序，基于哈希表。

### （2）`unordered_multiset`

与`unordered_set`类似，允许键重复。

### （3）unordered_map

注：存储键值对，不排序，基于哈希表

`std::unordered_map` 是一个关联容器，存储键值对（key-value pairs）。与 `std::map` 不同，它**不根据键的顺序**存储元素，而是使用**哈希表**实现，通过键的**哈希值**来组织元素。

**工作原理**

1.  **哈希函数**：对键进行计算，生成一个 `size_t` 类型的哈希值。
2.  **映射到表**：哈希值通过与表长取模，决定键值对存储在哪个位置中。
3.  **处理冲突**：不同的键可能产生相同的哈希值（哈希冲突）。标准要求使用**链地址法**（每个桶是一个链表）来解决冲突。在实践中，高性能实现可能会在链表过长时将其转换为树结构。

#### ①初始化

格式`std::unordered_map<KeyType, ValueType> myMap;`

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

#### ②插入元素

**使用 `insert()` 成员函数**

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

**使用 `emplace()` 成员函数 (C++11)**

`emplace` 直接在容器内构造元素，避免创建临时对象，通常更高效。

```cpp
// 原地构造，传入构造键和值所需的参数
auto result = umap.emplace("David", 40);
// 等同于 umap.insert({"David", 40})，但可能更高效
```

**使用下标操作符 `[]`**

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

**使用 `try_emplace()` 成员函数 (C++17)**

更安全和高效的插入方法。如果键已存在，**不构造**值对象。

```cpp
std::string key = "Alice";
// 只有当 key 不存在时，才会构造 std::string(5, 'A') 这个值
auto [iter, success] = umap.try_emplace(key, 5, 'A'); // 尝试插入 {"Alice", "AAAAA"}
// 如果 key 已存在，则不会发生任何构造，success 为 false
```

#### ③删除元素

**使用 `erase()` 成员函数**

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

#### ④访问元素

**使用下标操作符 `[]`（不推荐）**

```cpp
// 访问存在的键
std::cout << umap["Alice"]; // 输出: 26

// 访问不存在的键（危险！会插入新元素）
std::cout << umap["Ghost"]; // 插入 {"Ghost", 0} 并输出 0
```

**使用 `at()` 成员函数**

**安全访问**。如果键不存在，抛出 `std::out_of_range` 异常。

```cpp
try {
    std::cout << umap.at("Alice"); // 输出: 26
    std::cout << umap.at("Nonexistent"); // 抛出 std::out_of_range
} catch (const std::out_of_range& e) {
    std::cerr << "Key not found: " << e.what() << '\n';
}
```

**使用 `find()` 成员函数**

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

#### ⑤查询元素是否存在

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

#### ⑥遍历

```
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

#### ⑦哈希表特定操作

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

#### ⑧使用自定义类型作为键

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

### (4)`unordered_multimap`

### 容器适配器

### （1）栈`stack`

### （2）队列`queue`

### （3）优先队列`priority_queue`

## 2、Algorithm

C++ 标准库中的 <algorithm> 头文件是容器操作的 “瑞士军刀”，提供了大量通用算法，用于处理容器中的元素（如查找、排序、修改、转换等）。这些算法基于迭代器工作，与具体容器类型无关（只要容器提供符合要求的迭代器），因此具有极强的通用性和复用性。

### 核心特点

1. **基于迭代器**：算法通过迭代器操作元素，无需关心底层容器（如 vector、list、array 等），只需迭代器满足算法要求（如输入迭代器、随机访问迭代器等）。

1. **范围约定**：几乎所有算法的参数都采用 **[first, last)** 左闭右开区间，表示操作的元素范围（即从 first 指向的元素开始，到 last 指向的元素前结束）。

1. **不修改容器结构**：算法仅操作元素的值或位置，不会直接增加 / 删除容器元素（若需修改容器大小，需配合插入迭代器如 back_inserter）。

1. **可定制行为**：许多算法支持通过 “谓词”（predicate，即返回 bool 的函数 /lambda）自定义逻辑（如比较、过滤条件）。

### （1）非修改性序列操作（仅查询 / 遍历，不改变元素）

这类算法仅读取元素，不修改容器内容，适用于查询、统计、遍历等场景。

| 函数                           | 功能描述                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| for_each                       | 对范围 [first, last) 中的每个元素执行指定函数（如打印、计算）。 |
| count                          | 统计范围中等于 value 的元素个数。                            |
| count_if                       | 统计范围中满足一元谓词 pred 的元素个数（如统计偶数：count_if(v.begin(), v.end(), [](int x){return x%2==0;})）。 |
| find                           | 查找范围中第一个等于 value 的元素，返回其迭代器（未找到返回 last）。 |
| find_if                        | 查找范围中第一个满足一元谓词 pred 的元素（如查找第一个大于 10 的元素）。 |
| find_end                       | 在范围中查找子序列 [s_first, s_last) 最后一次出现的位置。    |
| search                         | 在范围中查找子序列 [s_first, s_last) 第一次出现的位置。      |
| equal                          | 判断两个范围 [first1, last1) 和 [first2, first2 + (last1-first1)) 是否所有元素相等（可自定义比较函数）。 |
| all_of/any_of/none_of（C++11） | 检查范围中是否**所有元素**满足 pred / **任一元素**满足 pred / **无元素**满足 pred。 |

非修改性序列操作的核心是 “只读不修改”，主要用于查询、统计、遍历等场景。以下是这类操作中常用函数的具体用法示例，每个示例都包含代码实现和效果说明：

#### a. for_each

**功能**：对范围 [first, last) 中的每个元素执行指定函数（可自定义操作，如打印、累加等）。

**示例**：遍历打印元素 + 计算总和

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    
    // 1. 打印每个元素（用lambda表达式）
    std::cout << "元素列表：";
    std::for_each(nums.begin(), nums.end(), 
        [](int x) { std::cout << x << " "; });  // 输出：1 2 3 4 5
    
    // 2. 计算总和（用外部变量捕获）
    int sum = 0;
    std::for_each(nums.begin(), nums.end(), 
        [&sum](int x) { sum += x; });  // 累加每个元素到sum
    std::cout << "\n总和：" << sum;  // 输出：15
    
    return 0;
}
```

#### b. count

**功能**：统计范围中**等于** **value** 的元素个数。

**示例**：统计 vector 中 “3” 出现的次数

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {3, 1, 3, 5, 3, 7};
    int target = 3;
    
    // 统计等于3的元素个数
    int cnt = std::count(nums.begin(), nums.end(), target);
    std::cout << "数字" << target << "出现的次数：" << cnt;  // 输出：3
    
    return 0;
}
```

#### c. count_if

**功能**：统计范围中**满足一元谓词** **pred** 的元素个数（比 count 更灵活，支持自定义条件）。

**示例**：统计偶数的个数

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};
    
    // 统计偶数（谓词：x % 2 == 0）
    int even_cnt = std::count_if(nums.begin(), nums.end(),
        [](int x) { return x % 2 == 0; });  // 2、4、6是偶数
    
    std::cout << "偶数的个数：" << even_cnt;  // 输出：3
    
    return 0;
}
```

#### d. find

**功能**：查找范围中**第一个等于** **value** 的元素，返回其迭代器（未找到返回 last）。

**示例**：查找第一个 “5” 的位置

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {2, 4, 5, 7, 5};
    int target = 5;
    
    // 查找第一个等于5的元素
    auto it = std::find(nums.begin(), nums.end(), target);
    
    if (it != nums.end()) {
        // 计算索引（距离begin的偏移量）
        int index = std::distance(nums.begin(), it);
        std::cout << "第一个" << target << "在索引" << index << "处";  // 输出：索引2
    } else {
        std::cout << "未找到" << target;
    }
    
    return 0;
}
```

#### e. find_if

**功能**：查找范围中**第一个满足一元谓词** **pred** 的元素（支持自定义查找条件）。

**示例**：查找第一个大于 10 的元素

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {5, 8, 12, 3, 15};
    
    // 查找第一个大于10的元素（谓词：x > 10）
    auto it = std::find_if(nums.begin(), nums.end(),
        [](int x) { return x > 10; });
    
    if (it != nums.end()) {
        std::cout << "第一个大于10的元素是：" << *it;  // 输出：12
    } else {
        std::cout << "没有大于10的元素";
    }
    
    return 0;
}
```

#### f. find_end

**功能**：在主范围中查找**子序列最后一次出现**的位置（返回子序列第一个元素的迭代器）。

**示例**：查找子序列 [3,4] 最后一次出现的位置

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> main_seq = {1, 3, 4, 2, 3, 4, 5};  // 主序列
    std::vector<int> sub_seq = {3, 4};  // 子序列
    
    // 查找子序列最后一次出现的位置
    auto it = std::find_end(main_seq.begin(), main_seq.end(),
                            sub_seq.begin(), sub_seq.end());
    
    if (it != main_seq.end()) {
        int index = std::distance(main_seq.begin(), it);
        std::cout << "子序列最后一次出现在索引" << index << "处";  // 输出：索引4（对应元素3）
    } else {
        std::cout << "未找到子序列";
    }
    
    return 0;
}
```

#### g. search

**功能**：在主范围中查找**子序列第一次出现**的位置（与 find_end 相反）。

**示例**：查找子序列 [3,4] 第一次出现的位置

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> main_seq = {1, 3, 4, 2, 3, 4, 5};
    std::vector<int> sub_seq = {3, 4};
    
    // 查找子序列第一次出现的位置
    auto it = std::search(main_seq.begin(), main_seq.end(),
                          sub_seq.begin(), sub_seq.end());
    
    if (it != main_seq.end()) {
        int index = std::distance(main_seq.begin(), it);
        std::cout << "子序列第一次出现在索引" << index << "处";  // 输出：索引1（对应元素3）
    } else {
        std::cout << "未找到子序列";
    }
    
    return 0;
}
```

#### h. equal

**功能**：判断两个范围是否**所有元素都相等**（可自定义比较函数）。

**示例**：比较两个 vector 是否相等 + 自定义比较（忽略大小写）

```
#include <algorithm>
#include <vector>
#include <string>
#include <cctype>  // 用于tolower
#include <iostream>

int main() {
    // 示例1：比较整数vector
    std::vector<int> a = {1, 2, 3};
    std::vector<int> b = {1, 2, 3};
    std::vector<int> c = {1, 2, 4};
    
    bool a_eq_b = std::equal(a.begin(), a.end(), b.begin());  // true
    bool a_eq_c = std::equal(a.begin(), a.end(), c.begin());  // false
    std::cout << "a与b是否相等：" << std::boolalpha << a_eq_b << "\n";  // true
    std::cout << "a与c是否相等：" << a_eq_c << "\n";  // false
    
    // 示例2：自定义比较（字符串忽略大小写）
    std::string s1 = "Hello";
    std::string s2 = "hElLo";
    // 自定义谓词：转换为小写后比较
    bool str_eq = std::equal(s1.begin(), s1.end(), s2.begin(),
        [](char c1, char c2) { 
            return std::tolower(c1) == std::tolower(c2); 
        });
    std::cout << "字符串忽略大小写是否相等：" << str_eq;  // true
    
    return 0;
}
```

#### i. all_of / any_of / none_of（C++11）

- all_of：检查**所有元素**是否满足谓词；

- any_of：检查**任一元素**是否满足谓词；

- none_of：检查**没有元素**满足谓词。

**示例**：检查数组的元素特性

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {2, 4, 6, 8};  // 全是偶数、正数
    
    // 1. 所有元素都是偶数吗？
    bool all_even = std::all_of(nums.begin(), nums.end(),
        [](int x) { return x % 2 == 0; });  // true
    std::cout << "所有元素都是偶数：" << std::boolalpha << all_even << "\n";  // true
    
    // 2. 存在元素大于10吗？
    bool has_gt10 = std::any_of(nums.begin(), nums.end(),
        [](int x) { return x > 10; });  // false（最大是8）
    std::cout << "存在大于10的元素：" << has_gt10 << "\n";  // false
    
    // 3. 没有元素是负数吗？
    bool none_negative = std::none_of(nums.begin(), nums.end(),
        [](int x) { return x < 0; });  // true（全是正数）
    std::cout << "没有负数：" << none_negative;  // true
    
    return 0;
}
```

#### 总结

非修改性序列操作的核心是 “只读”，通过迭代器访问元素并执行查询逻辑。使用时需注意：

- 范围均为 [first, last) 左闭右开区间；

- 未找到元素时，find/find_if/find_end/search 均返回 last；

- 自定义谓词（lambda / 函数）需保证无副作用（相同输入返回相同结果）。

### （2）修改性序列操作（修改元素值或位置）

这类算法会修改元素的值、位置或复制到其他容器，适用于转换、替换、复制等场景。

| 函数             | 功能描述                                                     |
| ---------------- | ------------------------------------------------------------ |
| copy             | 将范围 [first, last) 中的元素复制到 [dest, ...)（需确保目标有足够空间，或用 back_inserter 自动插入）。 |
| copy_if（C++11） | 复制范围中满足 pred 的元素到目标范围。                       |
| move（C++11）    | 将范围中元素 “移动” 到目标范围（源元素会变为 “可析构状态”，避免复制开销）。 |
| swap_ranges      | 交换两个范围 [first1, last1) 和 [first2, first2 + (last1-first1)) 中的元素。 |
| transform        | 对范围元素应用函数（或二元操作），结果存入目标范围（如 transform(v.begin(), v.end(), v.begin(), [](int x){return x*2;}) 实现所有元素乘 2）。 |
| replace          | 将范围中等于 old_val 的元素替换为 new_val。                  |
| replace_if       | 将范围中满足 pred 的元素替换为 new_val（如替换所有负数为 0）。 |
| fill             | 将范围中所有元素设为 value（如 fill(v.begin(), v.end(), 0) 清空容器）。 |
| generate         | 用函数 gen 的返回值填充范围（如 generate(v.begin(), v.end(), rand) 生成随机数）。 |

修改性序列操作的核心是 “修改元素值或位置”（但不直接改变容器的大小，除非配合插入迭代器），主要用于复制、转换、替换、填充等场景。以下是这类操作中常用函数的具体用法示例，包含代码实现和关键说明：

#### a. copy

**功能**：将范围 [first, last) 中的元素**复制**到目标范围 [dest, ...)（需确保目标有足够空间，或用 back_inserter 自动插入）。

**示例**：复制 vector 元素到另一个容器

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> src = {1, 2, 3, 4};
    std::vector<int> dest1(src.size());  // 提前分配与src相同的空间
    std::vector<int> dest2;  // 空容器，需用back_inserter自动插入
    
    // 1. 复制到已分配空间的dest1（直接用迭代器）
    std::copy(src.begin(), src.end(), dest1.begin());
    // 2. 复制到空容器dest2（用back_inserter自动扩容）
    std::copy(src.begin(), src.end(), std::back_inserter(dest2));
    
    // 输出结果
    std::cout << "dest1: ";
    for (int x : dest1) std::cout << x << " ";  // 1 2 3 4
    std::cout << "\ndest2: ";
    for (int x : dest2) std::cout << x << " ";  // 1 2 3 4
    
    return 0;
}
```

**关键**：若目标容器未提前分配空间，必须用 std::back_inserter（或 front_inserter/inserter）避免越界。

#### b. copy_if（C++11）

**功能**：仅复制范围中**满足一元谓词** **pred** 的元素到目标范围（条件复制）。

**示例**：复制所有偶数到新容器

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> src = {1, 2, 3, 4, 5, 6};
    std::vector<int> dest;
    
    // 复制满足“x是偶数”的元素到dest
    std::copy_if(src.begin(), src.end(), std::back_inserter(dest),
        [](int x) { return x % 2 == 0; });  // 筛选2、4、6
    
    std::cout << "复制的偶数：";
    for (int x : dest) std::cout << x << " ";  // 2 4 6
    
    return 0;
}
```

#### c. move（C++11）

**功能**：将范围 [first, last) 中的元素**移动**到目标范围（源元素会变为 “可析构状态”，避免复制开销，适合大型对象）。

**示例**：移动字符串 vector 的元素

```
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>

int main() {
    std::vector<std::string> src = {"apple", "banana", "cherry"};
    std::vector<std::string> dest;
    
    // 移动src的元素到dest（src会变为空或元素处于可析构状态）
    std::move(src.begin(), src.end(), std::back_inserter(dest));
    
    std::cout << "dest中的元素：";
    for (const auto& s : dest) std::cout << s << " ";  // apple banana cherry
    std::cout << "\nsrc的大小（移动后）：" << src.size();  // 3（大小不变，但元素已失效）
    std::cout << "\nsrc的第一个元素（失效）：" << src[0];  // 未定义（可能为空）
    
    return 0;
}
```

**关键**：移动后源容器的元素不应再使用（虽大小不变，但内容已被 “取走”）。

#### d. swap_ranges

**功能**：交换两个范围 [first1, last1) 和 [first2, first2 + (last1-first1)) 中的元素（两个范围需等长）。

**示例**：交换两个 vector 的前 3 个元素

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2, 3, 4};
    std::vector<int> b = {10, 20, 30, 40};
    
    // 交换a和b的前3个元素（范围长度为3）
    std::swap_ranges(a.begin(), a.begin() + 3, b.begin());
    
    std::cout << "a: ";
    for (int x : a) std::cout << x << " ";  // 10 20 30 4
    std::cout << "\nb: ";
    for (int x : b) std::cout << x << " ";  // 1 2 3 40
    
    return 0;
}
```

#### e. transform

**功能**：对范围元素应用函数（或二元操作），结果存入目标范围（支持 “一元转换” 或 “二元合并”）。

**示例**：一元转换（所有元素乘 2）+ 二元合并（两数组元素相加）

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    // 示例1：一元转换（原地修改）
    std::vector<int> nums = {1, 2, 3};
    std::transform(nums.begin(), nums.end(), nums.begin(),
        [](int x) { return x * 2; });  // 每个元素乘2
    std::cout << "一元转换后：";
    for (int x : nums) std::cout << x << " ";  // 2 4 6
    
    // 示例2：二元合并（a[i] + b[i] 存入c[i]）
    std::vector<int> a = {1, 2, 3};
    std::vector<int> b = {10, 20, 30};
    std::vector<int> c(3);  // 提前分配空间
    std::transform(a.begin(), a.end(), b.begin(), c.begin(),
        [](int x, int y) { return x + y; });  // 对应元素相加
    std::cout << "\n二元合并后：";
    for (int x : c) std::cout << x << " ";  // 11 22 33
    
    return 0;
}
```

#### f. replace

**功能**：将范围中**等于** **old_val** 的元素替换为 new_val（原地修改）。

**示例**：替换所有 “0” 为 “-1”

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {0, 2, 0, 4, 0};
    
    // 替换所有0为-1
    std::replace(nums.begin(), nums.end(), 0, -1);
    
    std::cout << "替换后：";
    for (int x : nums) std::cout << x << " ";  // -1 2 -1 4 -1
    
    return 0;
}
```

#### g. replace_if

**功能**：将范围中**满足一元谓词** **pred** 的元素替换为 new_val（条件替换）。

**示例**：替换所有负数为 0

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {-1, 2, -3, 4, -5};
    
    // 替换所有负数（x < 0）为0
    std::replace_if(nums.begin(), nums.end(),
        [](int x) { return x < 0; }, 0);
    
    std::cout << "替换后：";
    for (int x : nums) std::cout << x << " ";  // 0 2 0 4 0
    
    return 0;
}
```

#### h. fill

**功能**：将范围中**所有元素**设置为 value（批量赋值）。

**示例**：将 vector 的前 3 个元素填充为 9

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums(5);  // 初始化5个元素（默认0）
    
    // 填充前3个元素为9
    std::fill(nums.begin(), nums.begin() + 3, 9);
    
    std::cout << "填充后：";
    for (int x : nums) std::cout << x << " ";  // 9 9 9 0 0
    
    return 0;
}
```

#### i. generate

**功能**：用函数 gen 的返回值**填充范围**（适合生成有规律的序列，如随机数、自增序列）。

**示例**：生成自增序列和随机数

```
#include <algorithm>
#include <vector>
#include <iostream>
#include <cstdlib>  // rand
#include <ctime>    // time

int main() {
    // 示例1：生成自增序列（从1开始）
    std::vector<int> seq(5);
    int n = 1;
    std::generate(seq.begin(), seq.end(),
        [&n]() { return n++; });  // 每次调用返回n并自增
    std::cout << "自增序列：";
    for (int x : seq) std::cout << x << " ";  // 1 2 3 4 5
    
    // 示例2：生成随机数（1-100）
    std::vector<int> rand_nums(3);
    std::srand(std::time(0));  // 随机数种子
    std::generate(rand_nums.begin(), rand_nums.end(),
        []() { return rand() % 100 + 1; });  // 1-100的随机数
    std::cout << "\n随机数：";
    for (int x : rand_nums) std::cout << x << " ";  // 如：35 72 18（每次运行不同）
    
    return 0;
}
```

#### 总结

修改性序列操作的核心是 “改变元素”，使用时需注意：

- 目标范围的空间：copy、transform 等需确保目标有足够空间（或用 back_inserter 自动插入）；

- 原地修改 vs 复制：replace、fill 是原地修改，copy、move 是复制 / 移动到其他容器；

- 移动后的源元素：std::move 后源元素应视为 “无效”，避免再次使用；

- 谓词的灵活性：copy_if、replace_if 等通过谓词支持自定义条件，大幅提升复用性。

### （3）排序及相关操作（针对有序 / 无序范围）

排序是 <algorithm> 中最核心的功能之一，这类算法依赖随机访问迭代器（因此仅支持 vector、array、deque 等，list 需用自身的 sort 成员函数）。

| 函数          | 功能描述                                                     |
| ------------- | ------------------------------------------------------------ |
| sort          | 对范围 [first, last) 进行**不稳定排序**（默认升序，可自定义比较函数 comp）。 |
| stable_sort   | 对范围进行**稳定排序**（相等元素的相对顺序保持不变）。       |
| partial_sort  | 对范围部分排序：使前 n 个元素为范围中最小的 n 个，且已排序（其余元素无序）。 |
| nth_element   | 使范围中第 n 个元素（n 从 0 开始）处于 “正确位置”（左边元素 ≤ 它，右边元素 ≥ 它），常用于快速找中位数。 |
| lower_bound   | 在**已排序范围**中查找第一个 ≥ value 的元素（返回迭代器）。  |
| upper_bound   | 在**已排序范围**中查找第一个 > value 的元素（返回迭代器）。  |
| binary_search | 检查**已排序范围**中是否存在 value（返回 bool）。            |
| merge         | 合并两个**已排序范围**为一个新的有序范围（需目标范围足够大）。 |

排序及相关操作的核心是 “对元素进行排序或基于有序范围做高效查询 / 处理”，这类算法大多依赖**随机访问迭代器**（仅支持 vector、array、deque 等，list 需用自身 sort 成员函数），且二分查找、合并等操作必须基于 “已排序范围” 才能保证正确性。以下是常用函数的具体用法示例：

#### ①排序核心函数（无序→有序）

##### a. sort

`std::sort` 的核心目标是**高效排序**（平均时间复杂度 O (n log n)），但**不保证稳定性**。主流实现采用 **内省排序（Introsort）**，这是一种混合算法，结合了三种排序的优势：

- **快速排序（Quicksort）**：作为基础算法，因为它在平均情况下效率极高（缓存友好、常数因子小）。
- **堆排序（Heapsort）**：当快速排序的递归深度超过阈值（通常为 `2×log2(n)`）时切换，避免快速排序在最坏情况下（如已排序数组）退化为 O (n²) 复杂度。
- **插入排序（Insertion sort）**：当子数组长度小于阈值（通常为 16~20）时使用，因为插入排序在小规模数据上效率更高（减少递归开销）。

**功能**：对范围 [first, last) 进行**不稳定排序**（默认升序，可自定义比较规则）。

- 不稳定：相等元素的相对顺序可能改变；

- 效率高：平均时间复杂度 O (n log n)。

**示例**：默认升序 + 自定义降序 + 自定义结构体排序

```
#include <algorithm>
#include <vector>
#include <iostream>
#include <string>

// 自定义结构体：学生（姓名+分数）
struct Student {
    std::string name;
    int score;
};

int main() {
    // 示例1：默认升序排序（int类型）
    std::vector<int> nums1 = {3, 1, 4, 1, 5};
    std::sort(nums1.begin(), nums1.end());
    std::cout << "默认升序：";
    for (int x : nums1) std::cout << x << " ";  // 1 1 3 4 5

    // 示例2：自定义降序排序（用lambda）
    std::vector<int> nums2 = {3, 1, 4, 1, 5};
    std::sort(nums2.begin(), nums2.end(),
        [](int a, int b) { return a > b; });  // 降序规则：a>b时a在前
    std::cout << "\n自定义降序：";
    for (int x : nums2) std::cout << x << " ";  // 5 4 3 1 1

    // 示例3：自定义结构体排序（按分数升序，分数相同按姓名升序）
    std::vector<Student> students = {
        {"Bob", 85}, {"Alice", 90}, {"Bob", 80}
    };
    std::sort(students.begin(), students.end(),
        [](const Student& a, const Student& b) {
            if (a.score != b.score) {
                return a.score < b.score;  // 分数升序
            } else {
                return a.name < b.name;    // 分数相同，姓名升序
            }
        });
    std::cout << "\n结构体排序：\n";
    for (const auto& s : students) {
        std::cout << s.name << " " << s.score << "\n";  // Bob80 → Alice90 → Bob85（分数升序）
    }

    return 0;
}
```

##### b. stable_sort

`std::stable_sort` 的核心目标是**稳定排序**（相等元素的相对顺序不变），同时保证 O (n log n) 的时间复杂度。主流实现采用 **归并排序（Mergesort）**，或其变种（如原地归并排序优化）。

归并排序的稳定性来源于其 “合并” 步骤：当两个子数组中出现相等元素时，优先选取前一个子数组的元素，从而保留原始顺序。

**功能**：对范围进行**稳定排序**（相等元素的相对顺序保持不变），适合需要保留原始顺序的场景（如先按班级排序，再按分数排序，相同分数的学生仍保持原班级顺序）。

**示例**：稳定排序 vs 不稳定排序（对比相等元素顺序）

```
#include <algorithm>
#include <vector>
#include <iostream>

// 自定义结构体：值+原始索引（用于观察顺序）
struct Pair {
    int val;
    int index;  // 原始索引，标记插入顺序
};

int main() {
    std::vector<Pair> vec = {{2, 0}, {1, 1}, {2, 2}, {1, 3}};

    // 示例1：sort（不稳定排序，相等val的元素顺序可能变）
    auto vec_sort = vec;
    std::sort(vec_sort.begin(), vec_sort.end(),
        [](const Pair& a, const Pair& b) { return a.val < b.val; });
    std::cout << "sort（不稳定）：\nval 索引\n";
    for (const auto& p : vec_sort) {
        std::cout << p.val << "   " << p.index << "\n";  // 可能输出：1(1)、1(3)、2(2)、2(0)（2的索引顺序变了）
    }

    // 示例2：stable_sort（稳定排序，相等val的元素保持原索引顺序）
    auto vec_stable = vec;
    std::stable_sort(vec_stable.begin(), vec_stable.end(),
        [](const Pair& a, const Pair& b) { return a.val < b.val; });
    std::cout << "\nstable_sort（稳定）：\nval 索引\n";
    for (const auto& p : vec_stable) {
        std::cout << p.val << "   " << p.index << "\n";  // 固定输出：1(1)、1(3)、2(0)、2(2)（2的索引顺序不变）
    }

    return 0;
}
```

#### ②部分排序函数（无需全量排序）

##### a. partial_sort

**功能**：对范围进行**部分排序**—— 使前 n 个元素成为范围中 “最小的 n 个元素” 且已升序排列，其余元素无序（比全量排序更高效，适合只需前 n 个有序元素的场景）。

**示例**：取前 3 个最小元素并排序

```
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {5, 3, 1, 4, 2, 6};
    int n = 3;  // 要排序的前3个元素（最小的3个）

    // 语法：partial_sort(范围开始, 前n个元素的结束位置, 范围结束)
    std::partial_sort(nums.begin(), nums.begin() + n, nums.end());

    std::cout << "部分排序后（前3个最小且有序）：";
    for (int x : nums) std::cout << x << " ";  // 1 2 3 5 4 6（前3个是最小的1、2、3，后3个无序）

    return 0;
}
```

##### b. nth_element

**功能**：使范围中 “第 n 个元素”（索引从 0 开始）处于 “正确的排序位置”—— 所有小于它的元素在左边，大于它的元素在右边（左右元素无需有序），适合快速找 “第 k 小 / 大元素” 或中位数。

**示例**：找第 2 个（索引 2）元素，使其左边≤它、右边≥它

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {5, 3, 1, 4, 2, 6};
    int n = 2;  // 目标元素的索引（找第3小元素，因为索引从0开始）

    // 语法：nth_element(范围开始, 目标元素位置, 范围结束)
    std::nth_element(nums.begin(), nums.begin() + n, nums.end());

    std::cout << "nth_element后（索引2元素在正确位置）：";
    for (int x : nums) std::cout << x << " ";  // 如：1 2 3 5 4 6（索引2是3，左边1、2≤3，右边5、4、6≥3）
    std::cout << "\n第" << n+1 << "小元素：" << nums[n];  // 第3小元素是3

    return 0;
}
```

#### ③二分查找函数（需基于已排序范围）

**核心前提**：输入范围必须是**已排序**的（升序，自定义比较需对应），否则结果未定义。

##### a. lower_bound

**功能**：查找范围中 “第一个 ≥ value” 的元素，返回其迭代器（未找到返回 last）。

##### b. upper_bound

**功能**：查找范围中 “第一个> value” 的元素，返回其迭代器（未找到返回 last）。

##### c. binary_search

**功能**：检查范围中是否存在 value，返回 bool（本质是基于 lower_bound 实现，效率 O (log n)）。

**示例**：三者对比使用（在升序 vector 中找 3）

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 3, 4, 5};  // 已升序排序
    int target = 3;

    // 1. lower_bound：第一个≥3的元素（索引2）
    auto it_lower = std::lower_bound(nums.begin(), nums.end(), target);
    if (it_lower != nums.end()) {
        std::cout << "lower_bound：值=" << *it_lower << "，索引=" << std::distance(nums.begin(), it_lower) << "\n";  // 值=3，索引=2
    }

    // 2. upper_bound：第一个>3的元素（索引4）
    auto it_upper = std::upper_bound(nums.begin(), nums.end(), target);
    if (it_upper != nums.end()) {
        std::cout << "upper_bound：值=" << *it_upper << "，索引=" << std::distance(nums.begin(), it_upper) << "\n";  // 值=4，索引=4
    }

    // 3. binary_search：是否存在3
    bool exists = std::binary_search(nums.begin(), nums.end(), target);
    std::cout << "binary_search：" << (exists ? "存在" : "不存在") << target;  // 存在3

    // 扩展：统计target出现的次数（upper_bound - lower_bound）
    int count = it_upper - it_lower;
    std::cout << "\n" << target << "出现的次数：" << count;  // 3出现2次

    return 0;
}
```

#### ④合并函数（需基于已排序范围）

##### a. merge

**功能**：合并两个**已排序范围**为一个新的有序范围（默认升序，保持稳定性），适合将两个有序数组 /vector 合并为一个有序容器。

**示例**：合并两个升序 vector

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    // 两个已升序排序的范围
    std::vector<int> a = {1, 3, 5};
    std::vector<int> b = {2, 4, 6};
    std::vector<int> merged;  // 目标容器（存储合并结果）

    // 提前分配空间（可选，提升效率，避免频繁扩容）
    merged.reserve(a.size() + b.size());

    // 语法：merge(范围1开始, 范围1结束, 范围2开始, 范围2结束, 目标容器迭代器)
    std::merge(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(merged));

    std::cout << "合并后的有序范围：";
    for (int x : merged) std::cout << x << " ";  // 1 2 3 4 5 6

    return 0;
}
```

### 总结

排序及相关操作的关键注意事项：

1. **迭代器要求**：sort、stable_sort、partial_sort、nth_element 需**随机访问迭代器**，仅支持 vector、array、deque，不支持 list（list 用 list::sort()）。

1. **排序前提**：lower_bound、upper_bound、binary_search、merge 必须基于**已排序范围**（默认升序，自定义比较需统一规则）。

1. **稳定性选择**：需保留相等元素相对顺序用 stable_sort，否则用 sort（效率更高）。

1. **效率差异**：全量排序用 sort，只需前 n 个有序用 partial_sort，只需第 k 个元素位置用 nth_element（效率最高，O (n)）。

### （4）集合操作（针对有序范围）

这类算法用于处理两个**已排序范围**，模拟数学中的集合运算（需确保输入范围已排序）。

| 函数                     | 功能描述                                                     |
| ------------------------ | ------------------------------------------------------------ |
| set_union                | 求两个范围的并集（元素存在于任一范围，去重，结果有序）。     |
| set_intersection         | 求两个范围的交集（元素同时存在于两个范围，结果有序）。       |
| set_difference           | 求两个范围的差集（元素存在于第一个范围但不在第二个，结果有序）。 |
| set_symmetric_difference | 求两个范围的对称差集（元素存在于其中一个但不同时存在于两个，结果有序）。 |

**示例**：求两个有序数组的交集：

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2, 3, 4}, b = {3, 4, 5, 6}, res;
    // 求交集（需 a 和 b 已排序）
    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(res));
    // res 为 {3,4}
    return 0;
}
```

### （5）排列算法（生成序列的排列）

用于生成序列的字典序排列（需序列已排序，否则从当前状态开始生成）。

| 函数             | 功能描述                                                     |
| ---------------- | ------------------------------------------------------------ |
| next_permutation | 将序列转换为**下一个字典序更大**的排列（成功返回 true，否则返回 false，表示已为最大排列）。 |
| prev_permutation | 将序列转换为**上一个字典序更小**的排列（类似 next_permutation）。 |

**示例**：生成所有排列：

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::sort(v.begin(), v.end()); // 先排序，确保从最小排列开始
    do {
        for (int x : v) std::cout << x << " ";
        std::cout << "\n";
    } while (std::next_permutation(v.begin(), v.end()));
    // 输出所有 6 种排列
    return 0;
}
```

### （6）注意事项

1. **迭代器类型要求**：不同算法对迭代器类型有要求（如 sort 需要随机访问迭代器，find 仅需输入迭代器），使用时需确保容器迭代器满足要求（如 list 的迭代器是双向的，不能用 sort）。

1. **目标范围大小**：copy、transform 等算法不会自动扩容目标容器，需提前预留空间（或用 back_inserter 等插入迭代器自动插入）。

1. **自定义谓词**：谓词需满足 “纯函数” 特性（相同输入返回相同结果，无副作用），否则可能导致未定义行为。

1. **C++ 标准扩展**：C++11 及以上新增了大量算法（如 all_of、copy_if），C++20 引入了 ranges 版本的算法（更简洁，支持管道语法），可根据编译器版本选择使用。

## 3、Iterator

在 C++ 中，**迭代器（Iterator）** 是连接容器（Container）与算法（Algorithm）的核心桥梁，它提供了一种统一的方式来访问容器中的元素，而无需暴露容器的内部实现细节。迭代器的设计思想类似于指针，但比指针更通用 —— 它可以适配各种不同存储结构的容器（如数组、链表、树等）。

### （1）迭代器的核心作用

迭代器的本质是 **“元素访问工具”**，主要解决两个问题：

1. **统一接口**：无论容器内部是连续存储（如vector）还是非连续存储（如list），迭代器都提供一致的操作方式（如++移动到下一个元素，*获取元素值）。

1. **解耦容器与算法**：STL 中的算法（如sort、find）通过迭代器操作元素，无需关心具体是哪种容器，极大提升了代码复用性。

### （2）迭代器的分类与特性

C++ 标准根据迭代器支持的操作能力，将其分为**5 类**，每类迭代器支持的操作不同，且存在 “继承” 关系（如随机访问迭代器同时也是双向迭代器）。

| 迭代器类型                               | 核心能力（支持的操作）                                       | 典型对应容器 / 场景                     |
| ---------------------------------------- | ------------------------------------------------------------ | --------------------------------------- |
| 输入迭代器（Input Iterator）             | 只读访问，支持++（单向移动）、*（读值）、==/!=（比较），不可重复读取同一元素 | istream_iterator（从输入流读数据）      |
| 输出迭代器（Output Iterator）            | 只写访问，支持++（单向移动）、*（写值），不可重复写入同一元素 | ostream_iterator（向输出流写数据）      |
| 前向迭代器（Forward Iterator）           | 可读可写，支持输入 / 输出迭代器的所有操作，且可重复访问元素（可多次++遍历） | forward_list（单向链表）、unordered_set |
| 双向迭代器（Bidirectional Iterator）     | 支持前向迭代器的所有操作，额外支持--（反向移动）             | list（双向链表）、set、map              |
| 随机访问迭代器（Random Access Iterator） | 支持双向迭代器的所有操作，额外支持随机访问：+=n/-=n（移动 n 步）、[]（访问第 n 个元素）、</>（距离比较） | vector、array、deque、原生指针          |

### （3）迭代器的基本操作

不同类型的迭代器支持的操作不同，以下是通用基础操作（具体支持取决于类型）：

| 操作      | 含义                                                     | 支持的迭代器类型                        |
| --------- | -------------------------------------------------------- | --------------------------------------- |
| *it       | 获取迭代器指向的元素（读 / 写，取决于迭代器是否为const） | 除输出迭代器外（输出迭代器*it仅用于写） |
| it->mem   | 等价于(*it).mem（访问元素的成员）                        | 输入、前向、双向、随机访问迭代器        |
| ++it      | 移动到下一个元素（前置递增）                             | 所有迭代器                              |
| it++      | 移动到下一个元素（后置递增，返回旧值）                   | 所有迭代器                              |
| --it      | 移动到上一个元素（前置递减）                             | 双向、随机访问迭代器                    |
| it--      | 移动到上一个元素（后置递减，返回旧值）                   | 双向、随机访问迭代器                    |
| it + n    | 移动 n 个元素（返回新迭代器）                            | 随机访问迭代器                          |
| it - n    | 反向移动 n 个元素（返回新迭代器）                        | 随机访问迭代器                          |
| it1 - it2 | 计算两个迭代器之间的距离（元素个数）                     | 随机访问迭代器                          |
| it[n]     | 等价于*(it + n)（访问第 n 个元素）                       | 随机访问迭代器                          |
| it1 < it2 | 比较迭代器位置（是否 it1 在 it2 前）                     | 随机访问迭代器                          |

### （4）迭代器的获取与使用

容器通过成员函数提供迭代器，最常用的是：

- begin()：返回指向容器**第一个元素**的迭代器；

- end()：返回指向容器**最后一个元素的下一个位置**的迭代器（“尾后迭代器”，作为遍历结束标志）；

- cbegin()/cend()：返回const迭代器（只能读元素，不能修改）；

- rbegin()/rend()：返回反向迭代器（++实际是向前移动，用于逆序遍历）。

#### 示例：用迭代器遍历容器

```
#include <vector>
#include <list>
#include <iostream>
using namespace std;

int main() {
    vector<int> vec = {1, 2, 3, 4};
    list<int> lst = {5, 6, 7, 8};

    // 遍历vector（随机访问迭代器）
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        cout << *it << " "; // 输出：1 2 3 4
    }

    // 逆序遍历list（双向迭代器 + 反向迭代器）
    for (auto it = lst.rbegin(); it != lst.rend(); ++it) {
        cout << *it << " "; // 输出：8 7 6 5
    }

    return 0;
}
```

### （5）特殊迭代器

除了容器自带的迭代器，C++ 还提供了一些 “辅助迭代器” 用于特定场景：

1. **反向迭代器（reverse_iterator）**：

对正向迭代器的封装，++操作实际是正向迭代器的--，用于逆序遍历。例如rbegin()返回的就是反向迭代器，等价于reverse_iterator(end())。

1. **插入迭代器（Insert Iterator）**：

用于在容器中插入元素（而非覆盖），常见的有：

示例：用back_inserter向 vector 插入元素

```
#include <algorithm> // for copy
#include <vector>
using namespace std;

int main() {
    vector<int> src = {1, 2, 3};
    vector<int> dest;

    // 用back_inserter自动扩容dest并插入元素
    copy(src.begin(), src.end(), back_inserter(dest)); 
    // dest现在为{1,2,3}
    return 0;
}
```

- - back_inserter(c)：在容器c的末尾插入（需容器支持push_back，如vector、list）；

- - front_inserter(c)：在容器c的头部插入（需容器支持push_front，如list、deque）；

- - inserter(c, it)：在迭代器it指向的位置前插入（需容器支持insert）。

1. **流迭代器（Stream Iterator）**：

关联输入 / 输出流与迭代器，方便从流中读写数据。例如：

- - istream_iterator<T>(cin)：从标准输入流读T类型数据；

- - ostream_iterator<T>(cout, " ")：向标准输出流写T类型数据，间隔为空格。

### （6）迭代器失效问题

当容器进行**插入**或**删除**操作时，迭代器可能会 “失效”（即指向错误的位置或已释放的内存），使用失效的迭代器会导致未定义行为（如崩溃）。不同容器的迭代器失效规则不同：

| 容器类型          | 插入操作后迭代器失效规则                                     | 删除操作后迭代器失效规则                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| vector            | 若插入后重新分配内存（容量不足），**所有迭代器失效**；否则仅插入位置后的迭代器失效。 | 仅删除位置后的迭代器失效。                                   |
| list/forward_list | 所有迭代器均不失效（节点独立存储，插入不影响其他节点）。     | 仅被删除节点的迭代器失效，其他迭代器不受影响。               |
| deque             | 头部 / 尾部插入：若未重新分配内存，仅可能失效首尾迭代器；中间插入则**所有迭代器失效**。 | 头部 / 尾部删除：可能失效首尾迭代器；中间删除则**所有迭代器失效**。 |
| set/map           | 所有迭代器均不失效（树结构，插入不影响其他节点）。           | 仅被删除元素的迭代器失效，其他迭代器不受影响。               |

### （7）迭代器与指针的关系

- 迭代器可以看作 “广义指针”：对于连续存储的容器（如vector），其迭代器通常直接封装原生指针（T*），因此性能与指针一致；

- 对于非连续存储的容器（如list），迭代器需要通过自定义++/--操作实现节点跳转（本质是对节点指针的封装）。

## 4、Functional



