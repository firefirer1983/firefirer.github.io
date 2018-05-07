---
title: Network Data Structure
date: 2018-05-04 12:18:18
tags:
categories: 
- Unix Network Programming
---
# Ethernet Frame
{% asset_img IEEE-802.3-Ethernet-Frame-Format.png Ethernet Frame Structure%}

1. PREAMBLE (7 Bytes)
    > Ethernet frame以7个字节的Preamble作为开始，Preamble有一连串的 0 和 1组成，作用是给发送和接收两边建立一个 bit 同步。  
     Preamble 在开始设计的时候，Ethernet frame在旧式的网络设备由于delay而在传输过程常常会丢失开始的一些bits，Preamble就作为防护带引入。  
     Preamble 告诉接收端frame is comming，接收端在真正的frame开始前有时间去锁住住data stream。  
2. SFD (1 Byte)
    > SFD (Start of frame delimiter: frame开始分隔符，定界符), 这一字节总是设置为10101011 (0xAB)
3. Destination Address (6 Bytes)
    > 接收端的 Mac 地址
4. Source Address (6 Bytes)
    > 发送端的 Mac 地址
5. Length (2 Bytes)
    > 记录整个Ethernet frame的长度，最小值为46，最大值为1500(由于 Ethernet的设计限制)
6. Data (? Bytes)
    > 这是 IP header和 IP data都是在这里，为了避免 data length小于46 bytes，这里会自动填充 0s
7. CRC (4 Bytes)
    > 是用 Destination Address + Source Address + Length + Data field 计算出来的，用于校验，如果接收端算出的值与此不符  
    那么说明Ethernet frame被损坏，数据不能使用

# IP Packet
## IP Packets Encapsulation : &nbsp; packet header + packet playload
> IP Packets由 Header 和 Payload 组成，Payload其实就是下一层协议( TCP/UDP )的包裹  

{% asset_img ip_encapsulation.jpg%}

## Packet Header
{% asset_img ip_header.jpg%}

1. Version: 0~3 (4 bits)
    > IP协议的版本 IPv4/IPv6
2. IHL: 4~7 (4 bits)
    > IP Header 长度值
3. DSCP: 8~13 (6 bits)
    > Differentiated Services Code Point
4. ECN: 14~15 (2 bits)
    > Explicit Congestion Notifaction; 路由器用此字段表示拥挤的程度。
5. Total Length: 16~32 （16 bits)
    > 整个 IP packet 的长度 (IP header + IP payload)
6. Identification: 0~15 (16 bits)
    > 如果在传输过程中 IP packet被分块传输，同一个packet的所有分块的 identification数值都是一样的。  
    > 用此来重新组装复原packet
7. Flags: 16~18 (3 bits)
    > 当IP packet太大而无法处理的时候，这些 Flags来表示此packet是否能分块传输
8. Time to Live(TTL): 0~7 (8 bits)
    > 为了避免 IP packet 在无数个路由器网络中死循环, TTL的值表示 : 此packet可以在多少个路由器间跳转。  
    每跳转一次，此值减 1，如果为 0，则路由器直接丢弃此 packet
9. Protocol: 8~15 (8 bits)
    > 下一层协议的类型，也就是 IP packet payload 所带的数据是属于哪种协议的( TCP/UDP )
    - ICMP: 1
    - TCP: 6
    - UDP: 7
10. Header Checksum: 16~31 (16 bits)
    > IP Header的校验码。
11. Source Address: (32 bits)
    > 发包者的 IP 地址
12. Destination Address: (32 bits)
    > 接收者的 IP 地址
13. Options：(32 bits)
    > 可选
    
# TCP Segment Structure
> TCP : Transmission Control Protocol 传输控制协议  
TCP 从一个数据流中接收数据， 将数据分成很多个chunk，给每个chunk 加上 TCP Header后，就生成了一个TCP segment  
然后TCP segment再被封装成 IP packet，在各个客户端进行数据传输与交换。  
- TCP：Segment
- IP：Packet/Datagram
- Ethernet：Frame

## TCP Sement 包括 : &nbsp;  tcp segment header + tcp segment payload
{% asset_img wiki-tcp-header-structure.png%}

## TCP Header
> TCP Header由10个必需的字段组成 和 一个可选的字段组成:


1. Source port: (16 bits)
    > 发送者的port number
2. Destination port: (16 bits)
    > 接收者的port number
3. Sequence number (32 bits)
    > Sequence number 有两个用途:  
    - 当SYN flag为 1， 那么 Sequence number就是 initial sequence number. 也就是此对话连接第一个字节所在segment的Sequence num 
    - 当SYNC flag 为0， 那么这个Sequence number就是相对 initial sequence number 的计数
4. Acknowledgment number (32 bits)
    > 当 ACK flag 被设置为 1 ，那么Acknowledgment number 就是发送端
5. Data offset (4 bits)
    > 记录TCP header的大小，TCP header最小是20 Bytes，最大是60 Bytes，因此 TCP header中可选字段可达40 Bytes大小。
6. Reserved (3 bits)
    > 保留未用
7. Flags (9 bits) aka Control bits
    > - NS:  实验性质，实际使用少  
    > - CWR: Congestion Window Reduced flag
    > - URG: 用于告知当前的数据是“紧急segment“，“紧急的segment”会被发送端立刻发出，  
    接收端收到后也会直接处理掉。无需等待之前的segment是否处理完。紧急segment常用的场景：　　
    当host在发送数据到远端的设备．如果突然一个问题出现，host需要远端设备立刻停止处理之前的数据　　
    这时候host就可以通过发送一个URG的segment来达到此目的了。
    > - ACK: 用于告知对方成功接收packet
    > - PUSH: 就像URG，PUSH存在的目的就是为了让segment在发送和接收端能够被优先处理掉  
    > PUSH flag常用于数据传输的开始和结束阶段，同时影响到发送端和接收端的行为。
    > 当某个host发送数据的时候，他会暂时将数据queue在一个TCP buffer(一块特殊的memory)。  
    > TCP会一直等到buffer中的数据足够多，形成的segment的大小足够大才去发送出去。这样做的目的
    > 是为了保证每个segment的有效性，不至于发送过多的小空segment导致浪费带宽。  
    > 但是在某种特殊情况下data queue的行为反而不合适，例如在视频直播的时候，数据不需要buf，  
    > 以免出现视频被cut off的现象。
    > - RST: 当收到一个不该本链接收到的segment的时候，就会反馈一个RST flag
    > 常见的场景，是client要连接到host的时候，如果申请一个host并没有service的port，那么host就会
    > 直接回一个RST，告知client无此服务，而且host也会单方面 reset 这个connection
    > - SYN: 用于建立 3-way handshake between client and host
    > - FIN: 用于结束connection。当双方数据交换完成后，就会出现FIN。
    > 需要注意的是：当我方host需要关闭连接而发送了FIN给对方，但我方host仍然可继续接收数据，
    > 直到对方也发送一个FIN，我方才会关闭连接，释放buffer。