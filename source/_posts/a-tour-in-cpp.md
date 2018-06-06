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
    int *a = new int[10];// 从堆中分配大小为 10*sizeof(int)的内存空间，并且将其首个元素的内存地址值赋值给 a
    int &m = *(new int); // m 是从堆中分配大小为sizeof(int)的内存空间的引用
    int &a = *(new int[10]); // a 是从堆中分配的大小为 10*sizeof(int)大小空间的首个元素的引用
    a = 5;
    m = 5;
    static_assert(sizeof(*(new int)) == sizeof(*(new int[10])), "is the same");

### 自定义类型 : struct/class , enum, union



#### 类 class/struct  ：
与所属类同名的函数，称为 构造函数。构造函数是用来构造类对象的。
> 类的目的构造一个功能模块，实现功能的 ** 接口 ** 和 ** 实现 ** 分离。  
接口(代码可对外访问部分) : public
实现(代码不可对外访问部分，主要用于构造本功能) : private

#### 联合 union
union是一种特殊的struct，所有成员都被分配在同一块内存区域中。因此union所占的内存空间就是它最大成员所占的空间
而且，同一时刻 union只能保存一个成员的值

#### 枚举 enum
1. 裸的枚举类型

        enum {red, yellow, green

2. 类中的枚举类型：
    
        enum class Color{red, yellow, green};
        enum class TraficLigth{red, yellow, green};
        
枚举值位于class的作用域内，就可以在不同的enum class中重复使用这些枚举值而不致引起混淆例如：

    Color x = Color::red;
    Color y = TrafficLight::red;

默认情况下，enum class只定义了赋值，初始化 和比较( == 和 < ) 操作，然而，既然枚举类型是一种用户自定义类型
那么我们就可以为它定义别的运算符：

    TrafficLight& operator++(TrafficLight &t) {
      switch(t){
      case TrafficLight::green: return t=TrafficLight::yellow;
      case TrafficLight::yellow: return t=TrafficLight::red;
      case TrafficLight::red: return t=TrafficLight::green;
      }
    }
    
    TrafficLight now = TrafficLight::red;
    TrafficLight next = now ++; // next == TrafficLight::green
    
## 模块化
- 引言
- 分离编译
- 命名空间

### 引言
模块化的第一步是通过类来分离接口和实现，同时声明在*.h，定义在*.cpp

### 分离编译
用户代码(*.h)只可见类型和函数接口
头文件的作用是描述接口和强调逻辑结构的。


### 命名空间
命名空间机制，一方面表达某些声明式属于一个整体的，同时不会跟其他命名空间中的冲突。

### 错误处理
错误处理内容远超语言特性层面，而应归结为程序设计技术和工具范畴。
#### 异常(exception)
功能的作者一般不知道出现异常的时候用户(使用者)希望怎么处理，同时用户(使用者)不能保证每次都能正确使用功能
> 最佳的解决方案是由功能实现者负责检测，并且将异常情况通知使用者。

throw负责把程序的控制权从异常发生处转移到异常处理代码。
为了实现这一目标，异常实现部分需要解开(unwind)函数调用栈以便返回主调用函数的上下文。
异常处理机制把程序的控制权从当前作用域转移到处理该类型错误的代码，** 有必要的时候调用析构函数 **

把一个永远不会抛出异常的函数声明为noexcept，一旦真的发生错误，函数user还是会抛出异常，此时标准库函数
terminate()立即终止当前程序执行。

    void user(int sz) noexcept {
      std::vector(sz);
    }


不变式
> 对于类来说，一条假定某事为真的声明成为类的不变式(class invariant)，建立不变式是构造函数的任务，从而成员函数
可以依赖于该不变式，另一个作用是确保当前函数提出时不变式仍然成立。

#### 静态断言
异常和普通断言负责报告运行时错误，编译时错误可用静态断言来检查。

    static_assert(4<=sizeof(int), "integers size are too small");
    

## 类
- 引言
- 具体类型
- 抽象类型
- 虚函数
- 类层次结构
- 拷贝和移动
- 建议


C++ 最核心的语言特性就是类(class)，类有三种 ：
1. 具体类
2. 抽象类
3. 模板类
4. 类层次结构中的类

### 具体类：
- 基本思想是它们的行为"就像内置类型一样"
- 有自己确定的语义和操作集合

我们应该把简单的操作设置为内联的。也就是这些操作不应该以函数调用的方式实现。
** 定义在类内部的函数默认是内联的，我们也可以在函数声明前加上inline关键字，从而显示指定为内联。

默认构造函数：
> 无需实参就可以调用的构造函数称为默认构造函数(default constructor)

如果很多函数并不需要直接访问成员变量，可以将这些函数和类的定义分离。

#### 容器：是指一个包含若干元素的对象。

析构函数：
> 一种以确保构造函数分配的内存一档会被销毁的机制。

构造函数和析构函数的存在意义：
1. 一些值的初始化，可在构造函数内自动执行赋值行为。
2. 当类内部的成员是在堆上分配(new)，那么在构造函数new，析构函数 delete实现资源自动申请和回收。(RAII)

数据句柄模型(handle-to-data model): 用来管理在对象生命周期中大小会发生变化的数据。在构造函数中获取资源，  
然后在西沟函数中释放，这种技术称为(RAII：Resource Acquisition Is Initialization)。

试想，如果一个类里面所有的成员都是从栈上分配内存空间的，那么默认就是RAII的。
但是当类有从堆上分配资源，那么就需要用RAII的技术更好的去管理资源。(避免手动调用new/delete，或者叫裸new/delete)

#### 初始化容器
- 初始值列表构造函数(initializer-list constructor): 使用元素列表进行初始化
- push_back(): 类似函数

    class Container {
    public:
      Container(std::initializer_list<double>); // 使用一个double值列表进行初始化。
      void push_back(double); //在末尾添加一个元素，容器长度 +1
    };
    
std::initializer_list是一种标准库类型，编译器可以辨识它：
** 当我们使用 {} 列表的时候，如 {1,2,3,4}，编译器就会创建一个initializer_list类型的对象并将其提供给程序。

Container::Container(std::initializer_list<double> lst)
  :elem_(new double[lst.size], sz(static_cast<int>(lst.size)))
{
  std::copy(lst.begin(), lst.end(), elem);
}

### 抽象类型(接口类型) 含有纯虚函数的类，称为抽象类
Container类型之所以被称为具体类型，是因为他们的实现属于定义的一部分。在这点上与内置类型相似。
抽象类型则把使用者与类的实现细节完全隔离出来：

    class Container {
    public:
      virtual double& operator[](int) = 0; // 纯虚函数
      virtual int size() const = 0; // 常量成员函数
      virtual ~Container() {}    //析构函数
    };
    
> 因为我们对抽象类型一无所知，所以必须从 heap 上位对象分配空间，然后通过引用或指针访问对象。  
> 上面这个类纯粹是个接口。关键字virtual的意思是 "可能在随后的派生类中被重新定义"。  
> 纯虚函数意味着Container的派生类必须定义这个函数，因此不能单纯定义一个Container对象，Container只能作为  
> 接口出现，派生类负责具体实现size, operator[]函数，*** 含有纯虚函数的类称为抽象类 ***

- 作为一个抽象类，不需要提供构造函数，毕竟它不需要初始化数据和获取资源。
- 另一方面，抽象类必须有一个virual的析构函数，  因为需要执行派生类的具体析构函数来释放资源。



### 类层次结构
class 派生类 : 基类 {
};

class 子类： 超类 {
};


    void use(Container &c)
    {
      const int sz = c.size();
      for(int i=0; i!=sz; ++i)
        printf("%f ",c[i]);
    }
    
    class Vector : public Container {
    public:
      double &operator[](int i){ return v[i]; }
      size_t size(){ return sz_; }
    private:
      std::vector;
    };

    int main(int argc, char *argv[])
    {
      Vector v{1,2,3,4,5};
      use(vc);
    }
    
    
灵活性背后唯一不足的是我们只能通过引用或者指针操作对象。

#### 显式覆盖
如果派生类的某个函数与基类中虚函数的名字和类型都相同，则派生类的版本会覆盖基类中的版本。
可以用 override的指令显式的指明一个函数是否覆盖其基类的函数：

    void draw() const override;
    
当这个函数找不到其基类可覆盖的函数的时候，就会产生编译错误。

#### 层次结构的益处
- 接口继承：派生类对象可以用在任何使用基类对象的地方
- 实现继承：基类负责提供可以简化派生类实现的函数或者数据。

1. 具体类，尤其是表现形式不复杂的类，其行为非常类似于内置类型
> 将其定义为局部变量，通过名字访问，随意拷贝  

2. 层次结构中的类：
> 倾向于通过new来分配空间，然后通过指针或者引用访问。

#### 层次结构漫游：
如果基类指针要使用派生类的函数，就必须用dynamic_cast做转换，再检测后使用 

    Shape *ps = read_shape(cin);
    Smiley *p = dynamic_cast<Smiley*>(ps);
    if(p) {
      p.wink();
    }


#### 避免资源泄漏
- 应该用unique_ptr来管理，将new的指针存放在unique_ptr里面管理 

### 拷贝和移动
当设计一个类时候，必须仔细考虑对象是否会被拷贝以及如何拷贝的问题。

#### 拷贝容器
默认的 拷贝构造 和 拷贝赋值，都是按照每个类成员的值拷贝的。这种拷贝方式将违反资源句柄的不变式

类对象的拷贝，如果不想使用默认的每个成员的值拷贝，就必须要自定义 ** 拷贝构造函数 ** 与 ** 拷贝赋值运算符 **

    class Vector {
    public:
      // 默认构造函数
      Vector(std::initializer_list<double> lst): 
        sz_(lst.size()),
        ary_(new doulbe[sz_])
      {
      
      }
      
      ~Vector(){
        delete[] ary_;
        sz_ = 0;
      }
      
      // 拷贝构造函数
      Vector(const Vector &v) { 
        sz_ = v.sz_;  
        ary_ = new double[sz_];
        std::copy(ary_, ary_+sz_, v.ary_);
      }
      
      // 拷贝赋值运算符
      void operator=(const Vector &v) {
        sz_ = v.sz_;  
        ary_ = new double[sz_];
        std::copy(ary_, ary_+sz_, v.ary_);
      }
      
      size_t size() {
        return sz_;
      }
      
    priavate:
      double *ary_;
      size_ sz_;
    };
    
#### 基本操作
构造函数，析构函数，拷贝操作，移动操作 在逻辑上有千丝万缕的联系，在定义这些函数时我们必须考虑这种内在的
联系，否则就会遇到逻辑问题或者性能问题。
> 如果类X的析构函数执行了某些特定的任务，比如释放自 heap空间或者释放锁，则该类也应该实现所有的其他相关函数：

    class X {
      X(Sometype); // 普通构造函数
      X(); // 默认构造函数
      X(const X&); // 拷贝构造函数
      X(X &&); // 移动构造函数
      X& operator=(const X&); // 赋值拷贝运算符，清空目标对象并拷贝
      X& operator=(X &&); // 移动赋值运算符，清空目标对象并移动
      ~X();  // 析构函数，清空资源

编译器会根据需要生成上面的成员函数(普通构造函数除外)。 如果程序员希望显式使用函数的默认实现

下面5中情况下，对象会被移动或拷贝
- 被赋值给其他对象： 

    Vector a({1,2,3});
    Vector b({4,5,6);
    a = b; // 赋值拷贝运算符
    
- 作为对象初始值：

    Vector a({1,2,3});
    Vector b(a); // 拷贝构造函数

- 作为函数实参：

    void show(Vector v) {
      
    }


#### 资源管理
广义的资源：
> 任何具有  获取 -> 使用 -> 释放 模式的东西，都可以称为资源。
> 例如 内存，锁，套接字，文件句柄，线程句柄 都可以算为资源。同时任何无法同时使用的硬件，也能抽象为资源。

通过 move，移动构造运算符，移动赋值运算符，unique_ptr  

#### 抑制操作
处于层次结构的类，使用默认的拷贝或者移动操作常常意味着风险：因为基类指针是无法了解派生类有什么成员。
因此最好是删除了默认的拷贝和移动操作：
    
    class Shape {
    public:
      Shape(const Shape &) = delete;
      Shape &operator=(const Shape&) = delete;
      
      Shape(Shape &&) = delete;
      Shape& operator=(Shape &&) = delete;
      
      
## 模板
### 引言
> 模板：用一组类型或值，对其进行参数化的一个类或者一个函数。

一个模板及模板参数的集合，被称为实例化。
** 每个程序使用的实例化代码，是在编译后期，也就是实例化期才产生的 **

    template<typename T>
    class Vector {
    public:
      Vecotr(const std::initializer_list<T> &lst):
            sz_(lst.size()),
            ary_(new T[lst.size()])
      {
        int i = 0;
        for(auto &m:lst) {
          ary_[i] = m;
          ++ i;
        }
      }
      
      Vector(const Vector &v):
            sz_(v.sz_),
            ary_(new T[v.sz_]
      {
        int i = 0;
        for(const T &m:v) {
          ary_[i] = m;
        }
        ++ i;
      }
      ~Vector() {
        delete[] ary_;
        sz_ = 0;
      }
      
    private:
      T *ary_;
      size_t sz_;
    };
    
value_tupe 和 constexpr，除了类型参数，模板还可以接受普通的值参数，例如下面的N
    template<typename T, int N>
    struct Buffer {
      using value_type = T;
      constexpr int size() { return N; }
      T[N];
    }
 
 
### 概念和泛型编程
模板提供了以下功能：
- 把类型(以及数值 和 模板)作为实参传递而不损失任何信息的能力。这为内联提供了很多便利，现有的实现可利用这点。
- 延迟类型检查(在实例化时执行)。意味着程序可以把多个上下文的有用信息捏合在一起。
- 把常量值作为实参传递的能力，也就是提供编译时计算能力。
> 总而言之，模板为** 编译时计算 ** 和 ** 类型控制 **提供了强力的辅助支持，是我们可以编写更加简洁高效的代码。

### 函数对象
模板的一个特殊用途就是函数对象(functor)，我们可以像使用一个函数那样使用对象：
例如：
    
    template <typename T>
    class LessThan{
    public:
        LessThan(const T &v) :
                val(v){
        }
        bool operator()(const T &c) const {
            return (c < val);
        }
    
    private:
        const T val;
    };

用于指明通用算法的关键操作含义的函数对象 (如lessThan 之于 count())，被称为 策略对象(policy object)又叫
(higher order object)

如果觉得将定义和使用分离比较麻烦，可以用lamda表达式：

    [&](int a){ return a < x; }
    
[&]: 是一个捕获列表(capture list)，它指明所用的局部变量名字(如x)将通过引用访问，如果我们只希望捕获 x，
则可以写成： 

    [&x](int a){ return a<x; }

[=x]: 希望给生成的函数对象传递一个x的拷贝
    
    [=x](int a){ return a < x; }
    
[]: 什么都不捕获

### 可变参数模板 (variadic template)
template<typename T, typename... Tail>
void f(T head, Tail... tail) {
  g(head);
  f(tail...);
}

> 省略号... 表示列表的 "剩余部分" 。最终tail会变为空，我们需要另外一个函数处理空：
template<>
void f(){

}

### 别名
很多情况下，我们应该为类型或者模板引入一个同义词。例如 
using size_t = unsigned int;

实际上，每个标准库容器都提供了value_type作为其值类型的名字，这样我们编写的代码就能在任何一个服从这种规范的
容器上工作了：
    
    template<typename C>
    using ElementType = typename C::value_type;


通过绑定某些或者全部模板实参，我们就能使用别名机制定义新的模板了：

    template<typename Key, typename Value>
    class Map{
      // ...
    };
    
    template<typename Value>
    using StringMap = Map<string,Value>
    
    StringMap<int> m; // m是一个 Map<string, int>
    
## 标准库概览
### 标准库组件
- 运行时语言支持(例如，对资源分配和运行时类型信息的支持)
- C标准库
- 字符串(包括国际字符和本地化的支持)
- 正则表达式的支持
- I/O流，可扩展的输入输出框架，用户可向其中添加自己设计的类型，流，缓冲策略，区域设定和字符集。
- 容器(std::vector std::map)和算法(std::find, std::sort)
- 数值计算的支持(标准数学函数，复数，向量及随机数发生器)
- 并发程序设计支持(thread, mutex, condition)
- 支持模板元程序设计的工具，STL风格的泛型程序设计(pair)
- 资源管理的 "智能指针" unique_ptr, shared_ptr, weak_ptr
- 特殊用途容器 如 array,bitset,tuple
本质上，C++标准库提供了最常用的基本数据结构及其上的基础算法


    <array>         array
    <chrono>        duration, time_point
    <cmath>         sqrt(), pow()
    <complex>       complex, sqrt(), pow()
    <forward_list>  forward_list
    <fstream>       fstream, ifstream, ofstream
    <future>        future, promise
    <ios>           hex, dec, scientific, fixed, defaultfloat
    <iostream>      istream, ostream, cin, cout
    <map>           map, multimap
    <memory>        unique_ptr, shared_ptr, weak_ptr, allocator
    <random>        default_random_engine, normal_distribution
    <regex>         regex, smatch
    <string>        string, basic_string
    <set>           set, multiset
    <sstream>       istrstream, ostrstream
    <stdexcept>     length_error, out_of_range, runtime_error
    <thread>        thread
    <unordered_map> unordered_map, unordered_multimap
    <utility>       move(), swap(), pair
    <vector>        vector

来自C中的标准库头文件，例如 <stdlib.h> ，对应版本为 <cstdlib>。声明都在std::的空间中

## 字符串和正则表达式
string类型，比char*提供更完整的字符串处理能力

### 字符串
额外的字符串处理能力有：
- 连接(concat):  "+"
- 下表操作:  "[] 或 at()"
- 替换字符串内容: "replace()"
- C风格字符串(以0结尾的char数组): c_str()

### string的实现
为了处理多字符集，标准库定义了一个通用的字符串模板 basic_string， string 实际上是此模板的char实例化的一个别名。

    template<typename Char>
    class basic_string {
      // ... Char类型的字符串
    }
    using string = basic_string<char>

用户可以定义任意类型的字符串，例如日文字符类型：Jchar:

    using Jstring = basic_string<Jchar>;
    
    
### 正则表达式

    regex pat (R"(\w{2}\s*\d{5}(-\d{4})?)"); // 美国邮政编码模式: XXddddd-dddd 及其变形
    
原始字符串字面值常量(raw string literal)，其以 -R"" 包括，原始字符串字面量值常量好处式：  
可以直接包含反斜线 \ 和 "" 而无需转义

<regex>中，正则表达式提供了如下支持：
- regex_match()：将正则表达式与一个(已知长度的)字符串进行匹配。
- regex_search()：在一个(任意长的)的数据流中搜索与正则表达式匹配的字符串。
- regex_replace()：在一个(任意长的)数据流中搜索与正则表达式匹配的字符串并将其替换。
- regex_iterator：遍历匹配结果和子匹配。
- regex_token_iterator：遍历未匹配部分。

#### 搜索

    int lineno = 0;
    for(string line; getline(cin,line);) {
      ++lineno;
      smatch matches;
      if(regex_search(line, matches, pat))
        cout << lineno << ":" << matches[0] << '\n';
    }
    
regex_search(line, matches, pat) 在line中搜索任何与正则表达式pat 匹配的子串，如果匹配到，就保存在matches中  
如果没有任何匹配，就返回false。
smatch matches： smatch实际上是一个string的vector，每个string 保存的是一个子匹配。

    .   任意单个字符("通配符")
    [   字符集开始
    ]   字符集结束
    {   指定重复次数开始
    }   指定重复次数结束
    (   分组开始
    )   分组结束
    
    \   下一个字符有特殊含义
    *   零或多次重复(后缀操作)
    +   一或多次重复(后缀操作)
    ?   可选(零或一次)(后缀操作)
    |   二选一 (或)
    ^   行开始; 非
    $   行结束
    
    
    