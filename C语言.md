



# C语言

## 一、指针

### 1、指针简单性质

```c
#include<stdio.h>
int main(){
    int a;
    int *p = &a;
    return 0;
}
```

① *p 的值等于a的值，p的值等于存储a的地址

② p+1 等于p的值+sizeof(int);

③ p的所占字节和机械位数有关，如64位p的大小为8字节



### 2、函数的应用

```c
int swap(int x,int y){
	int t;
	t = y;
	y = x;
	x = t;
    return 0;
}
```

交换的是形参x，y，实际值并没有变化

```c
int swap(int *x,int *y){
	int t;
	t = *y;
	*y = *x;
	*x = t;
    return 0;
}
int main(){
    int a=1,b=2;
    swap(&a,&b);//使用时需要传输地址
    return 0;
}
```

### 3、指针的运算

```c
#include <stdio.h>

int main()
{
	int a = 1;
	int *pa = &a;
	printf("*pa = %d\n",*pa + 5);
	printf("*pa + 5 = %d\n",*pa + 5);
	printf("(*pa) + 5 = %d\n",(*pa) + 5);
    printf("++*pa + %d\n",++*pa);
	printf("*pa ++ = %d\n",*pa++);//输出*pa后执行pa++
    printf("(*pa) ++ = %d\n",(*pa)++);// 输出地址的值 + 1 
	printf("pa = %d\n",pa);
	printf("pa + 5 = %d\n",pa + 5); //pa + 1 与 (*pa) ++ 不同
	
    return 0;
}
```

输出结果：

```c
*pa = 6
*pa + 5 = 6
(*pa) + 5 = 6
*pa ++ = 1
(*pa) ++ = -1162752648 
##由于*pa ++返回*pa初始值之后，指针向后移动一位，导致指针指向一个没有存储数据的内存导致出现无法预料的结果
pa = -1162752647
pa + 5 = -1162752627
```

### 4、空指针与野指针

```c
int *p = null;//空指针
int *q;//野指针，指向随机地址，不一定存在（*q = 100不一定能执行）
```

避免野指针

```c
int *p = (int*) malloc(sizeof(int));//(int*)强转类型
free(p);//释放内存
```

### 5、指针与数组

a[i]等价于*(a + i)

### 6、指针与函数

① int *p()是指针型函数，返回值是指针

② int (*p)()是指向函数入口的指针变量，返回值是int型

### 7、回调函数

通过函数指针调用函数

```c
#include<stdio.h>
#include<time.h>
#include <stdlib.h>

int lesser(int x,int y){
    return x<y ? 1:0;
}
int sort(int *p,int len,int (*p)(int,int)){
    ......
        return 0;
}
//调用
int main(){
    int len;
    printf("请输入数组大小：");
    scanf("%d",&len);
    srand(time(0));
    
    int *nums = (int*) malloc(len * sizeof(int));
    for(int i=0;i<len;i++){//生成len长度的0-10随机大小的数组
        *(nums + i) = rand()%11;
    }
    sort(nums,len,lesser);
}


```

## 二、深拷贝和浅拷贝

### 1、浅拷贝

仅复制结构体本身（包括指针的值），不复制指针指向的数据。

- 原对象和拷贝对象的指针成员**指向同一块内存**
- 修改任一对象的数据会影响另一个对象
- 释放内存时可能导致重复释放（Double Free）或野指针

### 2、深拷贝

不仅复制结构体本身，还**递归复制指针指向的所有数据**。

- 原对象和拷贝对象**完全独立**，互不影响
- 需手动为新指针分配内存并复制数据
- 释放时需分别释放各自的内存

## 三、输入输出

### 1、printf与scanf的格式化处理

#### （1）printf格式化输出

```c
%[flags][width][.precision][length]type
```

##### ①标志(flags)

| 标志 | 名称   | 适用函数 | 作用描述                                                     | 示例                   |
| :--- | :----- | :------- | :----------------------------------------------------------- | :--------------------- |
| `-`  | 减号   | printf   | **左对齐**（默认右对齐）                                     | `%-10d`                |
| `+`  | 加号   | printf   | **强制显示符号**（正数显示`+`，负数显示`-`）                 | `%+d` → `+42`          |
| ` `          |空格|printf|**正数前加空格**（负数仍显示`-`）|`% d` → `42`|
| `#`  | 井号   | printf   | **添加前缀**： - `%o`：八进制加`0` - `%x`/`%X`：十六进制加`0x`/`0X` - `%f`/`%e`/`%g`：强制显示小数点 | `%#x` → `0x1a`         |
| `0`  | 零     | printf   | **用零填充宽度**（默认用空格填充）                           | `%05d` → `00042`       |
| `'`  | 单引号 | printf   | **千位分隔符**（C99标准，依赖本地化设置）                    | `%'d` → `1,000,000`    |
| `I`  | 大写i  | printf   | **使用本地化数字格式**（Windows扩展）                        | `%I64d`                |
| `*`  | 星号   | 两者     | **动态指定宽度/精度**（通过参数传递）                        | `printf("%*d", 5, 42)` |

示例

```c
#左右对齐#
printf("|%-10s|", "Hello");  // 输出 "|Hello     |"
printf("|%10s|", "Hello");   // 输出 "|     Hello|"
```

```c
#0填充#
printf("%05d", 42);     // 输出 "00042"
printf("%08.2f", 3.14); // 输出 "00003.14"
```

##### ②宽度（width）

示例

```c
printf("%5d", 10);   // 输出 "   10"（右对齐）
printf("%-5d", 10);  // 输出 "10   "（左对齐）
```

##### ③精度（Precision）

示例

```c
printf("%.2f", 3.14159); // 输出 "3.14"
printf("%.5s", "Hello World"); // 输出 "Hello"
```

##### ④长度修饰符（Length）

示例

```c
long big = 1234567890L;
printf("%ld", big);   // 输出长整型

double d = 3.14159;
printf("%Lf", d);     // 输出长双精度浮点
```

##### ⑤类型（type）

| 说明符 | 类型             | 示例输出         |
| :----- | :--------------- | :--------------- |
| `%d`   | 有符号十进制整数 | `123`            |
| `%u`   | 无符号十进制整数 | `456`            |
| `%f`   | 浮点数           | `3.140000`       |
| `%c`   | 单个字符         | `'A'`            |
| `%s`   | 字符串           | `"Hello"`        |
| `%p`   | 指针地址         | `0x7ffeeb0b4d28` |
| `%x`   | 十六进制整数     | `1a`             |
| `%o`   | 八进制整数       | `12`             |
| `%%`   | 输出百分号       | `%`              |

#### （2）scanf格式化输入

##### ①宽度限制

示例

```c
char str[10];
scanf("%9s", str); // 最多读取9个字符
```

##### ②忽略输入

示例

```c
int a, b;
scanf("%d %*d %d", &a, &b); // 忽略中间的输入
```

##### ③正则表达式

示例

```c
char vowels[10];
scanf("%[aeiou]", vowels); // 只读取元音字母

char line[100];
scanf("%[^\n]", line); // 读取整行直到换行符
```

### 2、字符（串）输入输出

#### （1）getchar() 和putchar() 函数

```c
#include <stdio.h>
int main( )
{
   int c;
 
   printf( "Enter a value :");
   c = getchar( );//获取
 
   printf( "\nYou entered: ");
   putchar( c );//输出
   printf( "\n");
   return 0;
}
```



#### （2）gets() 和 fgets() 函数

gets() 函数用于从标准输入设备读取一行字符串，但不推荐使用，因为它容易导致缓冲区溢出，推荐使用 **fgets()** 函数。

##### ①gets（）

```c
gets(char *str);
#成功返回`str`，失败或EOF返回NULL
```

##### ②fgets（）基本用法

```c
char *fgets(char *str, int n, FILE *stream);
#str：存储输入的字符数组
#n：最大读取字符数（包含结尾的\0）
#stream：输入流（stdin表示标准输入）
```

#### （3）puts()与fputs()函数

##### ①puts

```c
puts(char *str);
#将一个字符串输出到标准输出设备，并自动在末尾添加换行符
#成功时返回非负值，失败时返回EOF
```

##### ②fputs

```c
int fputs(const char *str, FILE *stream);
#将字符串输出到指定的流（如标准输出、文件等），但不会自动在字符串末尾添加换行符。
#str：要输出的字符串（以空字符 \0 结尾的字符数组）。
#stream：指定输出的流，可以是标准输出（stdout）、文件流等。
#成功时返回一个非负值（通常是输出的字符数）,失败时返回 EOF。
```

