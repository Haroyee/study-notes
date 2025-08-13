



# C语言

## 一、指针

1、

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



2、函数的应用

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

3、指针的运算

```c
#include <stdio.h>

int main()
{
	int a = 1;
	int *pa = &a;
	printf("*pa = %d\n",*pa + 5);
	printf("*pa + 5 = %d\n",*pa + 5);
	printf("(*pa) + 5 = %d\n",(*pa) + 5);
	printf("*pa ++ = %d\n",*pa++);
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
pa = -1162752647
pa + 5 = -1162752627
```

4、空指针与野指针

```c
int *p = null;//空指针
int *q;//野指针，指向随机地址，不一定存在（*q = 100不一定能执行）
```

避免野指针

```c
int *p = (int*) malloc(sizeof(int));//(int*)强转类型
free(p);//释放内存
```

5、指针与数组

a[i]等价于*(a + i)

6、指针与函数

① int *p()是指针型函数，返回值是指针

② int (*p)()是指向函数入口的指针变量，返回值是int型

