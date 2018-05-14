---
title: MTU MSS
date: 2018-05-12 08:56:28
tags:
---
{% asset_img MTU-image-1.png MTU and MSS %}

# MTU( bytes ):
> Maximum packet size allowed on an interface，MTU的值是单方设置的，每个interface设置一个MTU

## Ethernet default MTU = 1500 Bytes ##
MTU的值只是Ethernet Frame的数据段的值，Ethernet Frame加上 Header 和 FCS后就超过了1500 Bytes
通常 Ethernet的数据段就是一个 IP packet 了。
通常 MTU 的值为1500，但是可以设置MTU 为 9000，那么此时的 Ethernet Frame就称为Jumbo Frame
- Jumbo Frame的好处：  
1. 更少的over head，因为传输同样的数据量只需要更少的 Ethernet Frame Header
2. 接收端只需要处理较少的Frames
- 缺点：  
1. Fragmentation出现，当下游接收端的MTU设置为1524的时候，就无法接收Jumbo frames的时候，接收就需要分割Frames及加Headers
2. 需要更长的传输时间，可能会出现时延
3. 当出现错误的时候，重传的数据量更大

# MSS( bytes )
> Maximum Segment Size of TCP，MSS的值是TCP协议中各传输端协商的值，是动态变化的。
通常MTU 为1500的话，MSS为1460，原因是：
ip headers: 20 bytes
tcp headers: 20 bytes
Ethernet frame MTU: 1500 = MSS + ip headers + tcp headers = 1460 + 20 + 20

