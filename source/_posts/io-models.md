---
title: IO Models
date: 2018-05-06 17:29:57
tags:
---
# IO Models
## 阻塞式IO
    int fd = socket(AF_INET, SOCK_STREAM, 0);  
    read(fd, buf, sizeof(buf));
## 非阻塞式IO
## IO 复用
## 信号驱动式IO
## 异步IO