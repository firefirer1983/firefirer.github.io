---
title: Socket Options
date: 2018-05-11 09:51:08
tags:
---
# **常用的Socket Options**
- SO_REUSEADDR

1. 当创建一个socket的时候，需要绑定 ip 和 port ， 可以通过通配绑定的方式，让某一个socket绑定多个ip
或者多个port ，例如 0.0.0.0:21，就是绑定所有 ip上的 port 21。
如果这样就会出现 指定绑定和通配绑定冲突的情况，例如：  
socketA 先通过 0.0.0.0.21 通配绑定后， socketB 需要绑定 192.168.1.1:21就会失败，
除非设置 SO_REUSEADDR 

2. SO_REUSEADDR还会影响到 socket 的 TIME_WAIT 行为：  

> socket 通常都会有发送缓存区，当 send 返回的时候，不代表数据已经发送出去，而是代表数据
> 已经放入了发送缓冲区。对于tcp socket，在加入缓冲区和真正被发送之间时延会相当长，  
> 导致在close一个tcp socket的时候，缓冲区还保存待发送的数据，那么socket就会进入TIME_WAIT  
> 状态。这种状态会持续到数据被全部发送或者超时(此超时通常被称为Linger Time，大多数系统默认为 2 分钟

&nbsp; &nbsp; 一个没有设置SO_REUSEADDR的socket进入了TIME_WAIT状态的时候，内核仍然视其为绑定了原 ip 和 port  
&nbsp; &nbsp; 的socket，导致其他socket无法绑定此 ip和port，如果设为SO_REUSEADDR后，其他socket的bind会返回  
&nbsp; &nbsp; 成功，但是真正生效需要在socketA被真正释放后。  
&nbsp; &nbsp; 总结一下：  
&nbsp; &nbsp; ** 内核在处理一个设置了SO_REUSEADDR的socket绑定时，如果其绑定的ip和port和一个处于TIME_WAI
&nbsp; &nbsp; T状态的socket冲突时，内核将忽略这种冲突，即改变了系统对处于TIME_WAIT状态的socket的看待方式 **。
  

      if (setsockopt(tcp->io_watcher.fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)))
        return -errno;

- TCP_NODELAY

- TCP_KEEPALIVE (TCP_KEEPIDLE / TCP_KEEPINTVL / TCP_KEEPCNT )
> 当一个连接在2个小时(7200s)内没有任何数据交换，本端就会发送 keepalive 保活报文，此时会出现如下三种情况：  
> 1. 对端回复ACK。则本端TCP认为该连接依然存活。继续等7200s后再发送keepalive报文。  
> 2. 对端回复RESET。说明对端进程已经重启，本端的应用程序应该关闭该连接。
> 3. 没有对端的任何回复。则本端做重试，如果重试9次（前后重试间隔为75秒）仍然不可达，
> 则向应用程序返回错误信息，ETIMEOUT（无任何应答）或EHOST
<br>
    TCP_KEEPALIVE: on/off (default: on)  
    TCP_KEEPIDLE: seconds (default 7200s)  
    TCP_KEEINTVL: seconds (default 75s)
    TCP_KEEPCNT: retry count (default 9次)
  
  
- SIOCGIFFLAGS, SIOCSIFFLAGS
> 读取 或 设置 设备的 活动标志字

    IFF_UP	        接口正在运行.
    IFF_BROADCAST	有效的广播地址集.
    IFF_DEBUG	内部调试标志.
    IFF_LOOPBACK	这是自环接口.
    IFF_POINTOPOINT	这是点到点的链路接口.
    IFF_RUNNING	资源已分配.
    IFF_NOARP	无arp协议, 没有设置第二层目的地址.
    IFF_PROMISC	接口为杂凑(promiscuous)模式.
    IFF_NOTRAILERS	避免使用trailer .
    IFF_ALLMULTI	接收所有组播(multicast)报文.
    IFF_MASTER	主负载平衡群(bundle).
    IFF_SLAVE	从负载平衡群(bundle).
    IFF_MULTICAST	支持组播(multicast).
    IFF_PORTSEL	可以通过ifmap选择介质(media)类型.
    IFF_AUTOMEDIA	自动选择介质.
    
- FIOASYNC
> 设置或清除异步 IO 的标志(决定能否收到 SIGIO 信号)

- FIONBIO
> 设置文件为阻塞还是非阻塞，当设为非阻塞，但是read/write 失败的时候，  
> errno会为EAGAIN或者EWOULDBLOCK(windows)
> - EINTR: 指操作被中断唤醒，需要重新读/写  
> - EAGAIN: 非阻塞socket编程处理EAGAIN错误，不需要重新读/写
> - EWOULDBLOCK: windows版本的 EAGAIN

- FIOCLEX / FIONCLEX
> FIOCLEX (File io close on exec) : 当本进程执行exec的函数的时候，自动关闭所有打开的文件
> 原因是一般执行 exec 的目的是要用完全不同的新的进程替换老的进程，所以就需要将老的文件都关闭。
> 以防有泄漏