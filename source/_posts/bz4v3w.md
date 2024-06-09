---
title: 计算机网络 TCP 握手与挥手
date: '2024-03-07T08:45:00+00:00'
published: true
feature: ''
---
建立TCP连接的时候，需要3次握手，断开TCP连接的时候需要4次挥手。

## 三次握手：

握手必须由client发起

1\. client --> server TCP头：`SYN=1, seq=c`

2\. server --> client TCP头：`SYN=1, ACK=1, seq=s, ack = c+1`

3\. client --> server TCP头 ：`ACK=1, seq=c+1, ack = s+1`

client发起连接申请，server同意client的申请同时也发起连接申请，最后client也同意server的申请，三次握手完成TCP连接的建立。

建立连接必须client与server双方都有请求与确认的过程，因为server的请求与确认合并，所有握手过程是三次，

假如两次握手，意味着client请求，server确认就建立了连接，如果有client的包出现延迟被认为是丢失，client补发一个包，等到数据交换结束，延迟的包又到达服务器，就又建立了一个连接，这个连接会占用不必要的网络与服务器资源。

假如四次握手，不会比三次更加可靠，但是会消耗更多的资源。

## 四次挥手：

挥手可以由client或者server发起

1\. A --> B TCP头：`FIN=1`

2\. B --> A TCP头：`ACK=1`

3\. B --> A TCP头：`FIN=1`

4\. A --> B TCP头：`ACK=1`

A发起结束请求，B确认收到，并在完成所有数据传送之后也发起结束请求，A确认收到，四次挥手断开TCP连接。

断开连接也需要双方都请求与确认，但是存在一方请求结束，另一方此前的数据传输还没有完成的情况，所以另一方先确认收到，等到数据传输完成才能也发出请求。这是握手只要三次而挥手需要四次的原因。
