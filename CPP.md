

# 一、C++基础

## 1、vscode配置

tasks.json文件（第六行command切换gcc/g++）

```
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

# 二、C++中阶

## 1、函数的重载

通过函数重载，多个函数可以具有相同的名称和不同的参数，只要参数的数量和/或类型不同，多个函数可以具有相同的名称。

```

```



## 2、面向对象

### (1)面对对象编程（opp）

**面向对象**：对象是指具体的某一个事物，这些事物的抽象就是类，类中包含数据（成员变量）和动作（成员方法）。

**面向过程编程：**一种以执行程序操作的过程或函数为中心编写软件的方法。程序的数据通常存储在变量中，与这些过程是分开的。所以必须将变量传递给需要使用它们的函数。缺点：随着程序变得越来越复杂，程序数据与运行代码的分离可能会导致问题。例如，程序的规范经常会发生变化，从而需要更改数据的格式或数据结构的设计。当数据结构发生变化时，对数据进行操作的代码也必须更改为接受新的格式。查找需要更改的所有代码会为程序员带来额外的工作，并增加了使代码出现错误的机会。

**面向对象编程（Object-Oriented Programming, OOP）**：以创建和使用对象为中心。一个对象（Object）就是一个软件实体，它将数据和程序在一个单元中组合起来。对象的数据项，也称为其属性，存储在成员变量中。对象执行的过程被称为其成员函数。将对象的数据和过程绑定在一起则被称为封装。

面向对象编程将数据成员和成员函数封装到一个类中，并声明数据成员和成员函数的访问级别（public、private、protected），以便控制类对象对数据成员和函数的访问，对数据成员起到一定的保护作用。而且在类的对象调用成员函数时，只需知道成员函数的名、参数列表以及返回值类型即可，无需了解其函数的实现原理。当类内部的数据成员或者成员函数发生改变时，不影响类外部的代码。

### (2)构造函数与虚函数

构造函数分为无参构造，有参构造，全参构造、拷贝构造与析构函数

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

③**虚析构函数：**当作为基类被继承时，销毁派生类只会调用基类的析构函数，需要将基类的析构函数变成虚析构函数，以便在派生类中重载

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

#### ①**封装**

将具体的实现过程和数据封装成一个函数，只能通过接口进行访问，降低耦合性。

#### ②**继承**

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

**私有类成员（private）**：可以在自身类内部访问，不可在在子类内外部访问，也不可在类外部访问；

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

#### ③**多态**

多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。

多态允许使用基类指针或引用来调用子类的重写方法，从而使得同一接口可以表现不同的行为。

多态使得代码更加灵活和通用，程序可以通过基类指针或引用来操作不同类型的对象，而不需要显式区分对象类型。

多态主要通过虚函数以及纯虚函数实现，使用虚函数可以在派生类中对基类中的函数进行重载，而纯虚函数在基类实现接口，在派生类中实现接口的实际操作。

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

## 2、动态内存

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

### （1）自动推导类型

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



### 四、常用库函数