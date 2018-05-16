---
title: C++ Features
date: 2018-05-16 10:23:00
tags:
---
    main

# Non-Copyable Class (不可复制赋值的类)
>  
> 在实际开发中，某些特定的类，不希望客户拷贝复制此类对象，因这种类是“不可复制赋值类”

1. boost里面有现成的，可直接继承就可： boost::noncopyable  


    class Channel : boost::noncopyable
    {
    public:
      typedef boost::function<void()> EventCallback;
      typedef boost::function<void(Timestamp)> ReadEventCallback;

2. 在std c11，需要用delete操作符明确的将此类的将赋值运算符删除，或者定义为private，用delete 比private更好，  
因为意义更明确，而且不需要定义函数体：


    delete:
    class Channel
    {
    public:
      Channel operator=(const Channel &ch) = delete; // 用delete函数将默认的赋值运算符删除
      
      typedef boost::function<void()> EventCallback;
      typedef boost::function<void(Timestamp)> ReadEventCallback;

    private:
     class Channel
     {
     private:
       Channel operator=(const Channel &ch){}; // 用private限制函数访问域达到删除的效果
     public:  
       typedef boost::function<void()> EventCallback;
       typedef boost::function<void(Timestamp)> ReadEventCallback;   

# const 和 mutable
> 对象的状态：  
> 一个对象的状态由其所有非静态数据成员的值构成 (静态成员不属于对象，属于类，所有对象共有)  
> 因此，成员函数可能改变此对象的状态，如果声明一个成员函数为 const，则可以用编译检查的方式保证此  
> 函数不会修改对象的状态

    class Box {
    public:
      unsigned Size() const {
        return (width*height);
      };
    private:
      unsigned width;
      unsigned height;
    }
    
但是有些对象的数据成员，并不是表示对象状态的，例如mutex lock (block)，只是为了操作数据成员而存在(meta data  
member)，因此在 const的函数里，如果要写此类数据成员，就需要将此类数据成员声明为 mutable 了，否则会编译失败:

    class Box {
    public:
      unsigned Size() const {
        std::lock_gard<std::mutex> scopelock(block);
        return (width*height);
      };
      unsigned SetWidth(unsigned w) {
        std::lock_gard<std::mutex> scopelock(block);
        width = w;
      }
      unsigned SetWidth(unsigned w) {
        std::lock_gard<std::mutex> scopelock(block);
        width = w;
      }
    private:
      mutable std::mutex block;
      unsigned width;
      unsigned height;
    }
    
