---
title: IO Models
date: 2018-05-06 17:29:57
tags:
categories: 
- Unix Network Programming
---
# IO Models
## 阻塞式IO
    int fd = socket(AF_INET, SOCK_STREAM, 0);  
    read(fd, buf, sizeof(buf));
## 非阻塞式IO
## IO 复用
> IO with buffer
> 当在 IO 复用的环境下使用 read/write buffer (fgets只返回其中一行，其余输入仍在stdin缓冲区)，因此一定会将问题复杂化，  
> 根本原因是select/poll 无法得知buffer中的数据是否可读，而一直阻塞。
## 信号驱动式IO
## 异步IO

