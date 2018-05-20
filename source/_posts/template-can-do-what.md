---
title: What Can Template Do ?
date: 2018-05-19 22:16:02
tags:
---

# What can template do?
## 将可变长度的数组作为参数传入:  
** 通常当需要将一个数组作为参数传入的时候，有两种传递方法：一般是将数组名作为指针，一种是传数组引用 ** 


### 1. 将数组名字以指针的方式传入，但是数组的长度就需要另外一个int变量，由程序员显式地传入：

        void ary_print(const char *ary, int len) {
          for(int i=0; i<len; i++) {
            printf("%d ", ary[i]);
          }
          printf("\n");
        }
        char ary[20];
        ary_print(ary, sizeo(ary));
        
        
### 2. 通过宏，来优化方法1，可以达到少传数组长度的目的：

        #define mAryPrint(ary) \
        do{\
          ary_print(ary, sizeof(ary));\
        }while(0)
        mAryPrint(ary);

### 3. 通过特值化的template，将数组长度作为模板参数传入，长度的值由编译器来推导：
> 优点：可以只传数组，函数以数组引用的方式接收。
> 缺点：不同的长度的数组会自动产生不同的函数。

        template <int N>
        void ary_print(const char (&ary)[N]) {
          for(int i=0; i<N; i++) {
            printf("%d ",ary[i]);
          }
          printf("\n");
        }
        
        char ary[20];
        char (&rary)[20] = ary;
        for(int i=0; i<20; i++) {
        　　rary[i] = i;
        }
        ary_print(ary);
    









