---
title: 计算机网络 TCP 握手与挥手
date: "2024-03-07T08:45:00+00:00"
published: true
feature: ""
---

建立 TCP 连接的时候，需要 3 次握手，断开 TCP 连接的时候需要 4 次挥手。

<!-- more -->

## 三次握手：

握手必须由 client 发起

1\. client --> server TCP 头：`SYN=1, seq=c`

2\. server --> client TCP 头：`SYN=1, ACK=1, seq=s, ack = c+1`

3\. client --> server TCP 头 ：`ACK=1, seq=c+1, ack = s+1`

client 发起连接申请，server 同意 client 的申请同时也发起连接申请，最后 client 也同意 server 的申请，三次握手完成 TCP 连接的建立。

建立连接必须 client 与 server 双方都有请求与确认的过程，因为 server 的请求与确认合并，所有握手过程是三次，

假如两次握手，意味着 client 请求，server 确认就建立了连接，如果有 client 的包出现延迟被认为是丢失，client 补发一个包，等到数据交换结束，延迟的包又到达服务器，就又建立了一个连接，这个连接会占用不必要的网络与服务器资源。

假如四次握手，不会比三次更加可靠，但是会消耗更多的资源。

## 四次挥手：

挥手可以由 client 或者 server 发起

1\. A --> B TCP 头：`FIN=1`

2\. B --> A TCP 头：`ACK=1`

3\. B --> A TCP 头：`FIN=1`

4\. A --> B TCP 头：`ACK=1`

A 发起结束请求，B 确认收到，并在完成所有数据传送之后也发起结束请求，A 确认收到，四次挥手断开 TCP 连接。

断开连接也需要双方都请求与确认，但是存在一方请求结束，另一方此前的数据传输还没有完成的情况，所以另一方先确认收到，等到数据传输完成才能也发出请求。这是握手只要三次而挥手需要四次的原因。
