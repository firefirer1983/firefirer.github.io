---
title: A Tour in C++
date: 2018-05-28 09:45:06
tags:
---
#  A Tour in C++
## 基本知识
### 声明，类型，对象，变量，值
- 声明(declaration): 一条语句，为本文件引入一个新的名字 
- 类型(type): 类型定义了一组可能的值，以及(此类型的对象上) 可以实施的操作
- 对象(object): 存放某类型值的内存空间
- 变量(variable): 已命名的对象
- 值(value): 一组二进制位，具体含义由其类型负责解释

### 变量初始化
C++ 提供两种初始化的方式：
1. "="
2. {}: 这种方式是以花括号，将一个初始化值列表包括起来

### 作用域和生命周期：
声明语句把一个名字引入他的作用域中：
- 局部作用域：域大小是由 {} 包括起来的块，函数参数也属于局部作用域。
- 类作用域：一个名字定义在类内部，同时位于任何函数，lambda，enum class的外部
- 命名空间作用域：如果一个名字定义在命名空间内部，同事位于任何函数，lambda，enum class的外部。
- 声明在上述所有的结构之外的名字为全局作用域

### 常量
- const： 编译器负责确认并执行不变的承诺。
- constexpr： “在编译时求值” 。 允许把数据置于只读内存中，以防被破坏，以及提升性能

如果某个函数被用在常量表达式( const expression )中，该表达式在编译时求值，则这个函数必须定义为constexpr
例如：

        constexpr double square(double x) { return x*x; }
        
想要定义为constexpr，函数必须非常简单：函数中仅有一条计算某个值的return 语句。

### 指针，数组，引用
- xxx[]表示 xxx的数组
- xxx* 表示 指向 xxx
- xxx& 表示 xxx的引用，一个引用初始化后不能再引用其他对象了。

数组里的元素遍历的新方法 range-for：

    int v[]={0,1,2,3,4,5,6,7,8,9};
    for(auto x:v)
      cout << x << endl;
        
如果我们不希望将v的值拷贝到变量x中，而只是让x引用一个元素：

    int v[]={0,1,2,3,4,5,6,7,8,9};
    for(auto &x:v)
      cout << x << endl;


func 通过把参数 r 类型定义为 int 引用，在调用 func(int &r) 函数的时候就不必把实参拷贝给形参。
在调用 func(my_val) 的是，就直接在 my_val上执行操作。

如果既不想修改实参内容，由希望节省参数拷贝的的代价，可以用 const 引用

## 用户自定义类型

### 内置类型
由const，基本类型 (int, char, short , double) ，声明运算符([], * , &) 三种构造出来的类型，称为内置类型(build-in type)

### 自定义类型
由c++的抽象机制构建的新类型，称为用户自定义类型(user-defined type)，诸如 class 和 enum

### new / delete 操作符
new 运算符从一块名为(动态内存 dynamic memory，或叫 堆 heap) 的区域中分配一段内存空间。分配的内存独立于它所在
的作用域，会一直" 存活 "到使用 delete运算符销毁它为止。
    
    
    int *m = new int;   // 从堆中分配大小为sizeof(int)的内存空间，并且将内存空间的地址值赋值给m
    int *a = new int[10];// 从堆中分配大小为 10*sizeof(int)的内存空间，并且将内存空间的地址值赋值给 a
    int &m = *(new int); // m 是从堆中分配大小为sizeof(int)的内存空间的引用
    int &a = *(new int[10]); // a 是从堆中分配的大小为 10*sizeof(int)大小空间的首个元素的引用
    a = 5;
    m = 5;
    static_assert(sizeof(*(new int)) == sizeof(*(new int[10])), "is the same");

### 自定义类型 : struct, class , enum

#### 与所属类同名的函数，称为 构造函数。构造函数是用来构造类对象的。
