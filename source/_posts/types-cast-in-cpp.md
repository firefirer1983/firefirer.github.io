---
title: Types Cast In C++
date: 2018-05-16 22:04:56
tags:
---
# Cast in C++
## static_cast<to-type>(expressions);
> ** static_cast ** 是最为常见的，用于转换基本类型和具有继承关系的类之间转换。
> 不太用于指针类型的之间的转换，他的效率没有reinterpret_cast的效率高。
> 该运算符把expressions转换为to_type类型，但没有运行时类型检查来保证转换的安全性。  
它主要有如下几种用法：  
1. 用于类层次结构中基类和子类之间指针或引用的转换。   
** up-cast **（把子类的指针或引用转换成基类表示）是安全的。  
** down-cast **（把基类指针或引用转换成子类表示）时，由于没有动态类型检查，所以是不安全的。
2. 用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。   
3. 把空指针转换成目标类型的空指针。
4. 把任何类型的表达式转换成void类型。
** 注意：static_cast不能转换掉expressions的const，volatile，或者__unaligned属性。 **

    
1. 类层次结构中基类和子类之间指针或引用的转换，既可以up-cast(SubClass -> BaseClass)，也可以down-cast  
(BaseClass -> SubClass)    


    class Shape {};
    class Circle: public Shape {
    }
    Shape *base = new Shape();
    Circle *circle = new Circle();
    base = static_cast<Shape*>(circle); // up-cast
    
    circle = static_cast<Circle*>(base); // down-cast


2. 基本类型之间转换：

    
    int i;
    char c;
    i = static_cast<int>(c);
    

## implicit_cast<to-type>(expressions);
> implicit_cast 只能进行up-cast，否则就编译报错：

    template<typename To, typename From>
    inline To implicit_cast(From const &f)
    {
      return f;
    }

例子：

    class Shape{};
    class Circle:public Shape{};
    class Square:public Shape{};
    class Cake: public Circle,public Square{};
    void Cut(Circle &c){
      std::cout << "cut circle" << std::endl;
    }
    void Cut(Square &s){
      std::cout << "cut square" << std::endl;
    }
    int main(){
      Shape shape;
      Cake cake;
      Cut(cake); //编译报错，因为不知道call那个Cut
      Cut(static_cast<Circle &>(cake)); // ok!
      Cut(static_cast<Square &>(cake)); // ok!
      Cut(static_cast<Shape &>(shape)); // 编译可以通过，但是由于是down-cast会有风险
      Cut(implicity_cast<Shape &>(shape)); // 编译无法通过，因为implicity-cast只能做up-cast
      return 0;
    }
    
## const_cast<to-type>(expressions)
> 用于去除指针变量的常属性，将它转换为一个对应指针类型的普通变量，反过来也可以将一个非常量指针转换为一个常量指针变量
  
  
  
    class B{    
      public:    
      int num;    
    }    
    void foo(){    
      const B b1;    
      b1.num = 100; //comile error    
      B b2 = const_cast<B>(b1);    
      b2.num = 200; //fine    
    }    
    
## reinterpret_cast<to-type>(expressions)
> to-type必须是一个指针、引用、算术类型、函数指针或者成员指针。   
  它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，   
  在把该整数转换成原类型的指针，还可以得到原先的指针值）。   
  
> 两种情况下比较常见：
  1 需要直接操作内存的地方， 比如网络封包解包， 数据压缩加解密算法
  2 需要和c代码接口的地方

  例如下面做hash的例子，任何一个数值的hash = 其地址值的高16位和原地址值的异或的结果。
  
    #include <iostream>
    using namespace std;
    
    // Returns a hash code based on an address
    unsigned short Hash( void *p ) {
       unsigned int val = reinterpret_cast<unsigned int>( p );
       return ( unsigned short )( val ^ (val >> 16));
    }
    
    using namespace std;
    int main() {
       int a[10];
       for ( int i = 0; i < 10; i++ )
          cout << Hash( a + i ) << endl;
    }
    
    Output: 
    64641
    64645
    64889
    64893
    64881
    64885
    64873
    64877
    64865
    64869


## dynamic_cast<to-type>(expressions)
> 该运算符把 expressions 转换成 to-type 类型的对象。to-type 必须是类的指针、类的引用或者void 
> 如果to-type是类指针，那expressions必须是一个指针，如果to-type是一个引用，那expressions也必须是一个引用。
> dynamic_cast主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换。   
  在类层次间进行上行转换时，   
  dynamic_cast和static_cast的效果是一样的；   
  在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast更安全。
  
1. 只能在继承类对象的指针之间或引用之间进行类型转换
2. 这种转换并非在编译时，而是在运行时，动态的。
3. 没有继承关系，但被转换的类具有虚函数对象的指针进行转换  


    class B{    
      public:    
      int num;
      virtual void foo();    
    };    
        
    class D:public B{    
    public:    
      char *name[100];    
      void foo(){
        std::cout << name << std::endl;
      }
    };    
    
    void func(B *pb){
      D *pd1 = static_cast<D *>(pb); // 不安全的
      
      D *pd2 = dynamic_cast<D *>((pb);    
      if(pd2){
        pd2->foo(); // 安全的，因为动态转换检查
      }
    }
    
    int main(int argc, char *argv[]){
      B b;
      D d;
      func(&d); // pd1 和 pd2都是一样的，而且是安全的;
      func(&b); // pd1是不安全的，而pd2为nullptr
    }
    
> 另外要注意：B要有虚函数，否则会编译出错；static_cast则没有这个限制。   
  这是由于运行时类型检查需要运行时类型信息，而这个信息存储在类的虚函数表中，只有定义了虚函数的类才有虚函数表，   
  没有定义虚函数的类是没有虚函数表的。