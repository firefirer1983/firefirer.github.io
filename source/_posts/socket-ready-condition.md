---
title: Socket Ready Condition
date: 2018-05-06 22:25:06
tags:
---
# select返回的条件:
## read ready:
1. socket的read buffer收到的byte数目大于或者等于socke receive buffer的low-water mark  
我们可以通过SO_RCVLOWAT的选项设定low-water mark的值，此值默认设置为 1(TCP&UDP都是1)
2. 连接的读下半段关闭了。(例如收到FIN flag)。sock 上的read不会阻塞，而是直接返回0 ( 例如 EOF )
3. 负责监听(accept)的sockets上有任意连接过来。
4. 有一个socket的错误有待处理，此时 socket read不阻塞而是直接返回-1，同时设置errno，这些待处理错误  
可以通过制定SO_ERROR socket选项调用getsockopt获取且清除。

## write ready:
1. socket的write buffer中空闲的bytes数目大于或者等于 socket send buffer的low-water mark的时候。  
可以通过SO_SNDLOWAT选项调整low-water mark的值，此值默认设置为 2048( TCP & UDP 统一)
2. 连接的写下半段关闭了。socket上的写操作会产生SIGPIPE的信号。
3. 使用非阻塞式connect的socket已建立连接，或者connect以失败告终。
4. socket上有待处理错误(pending error)，这时 write将不会被阻塞而直接返回-1，同时设置errno