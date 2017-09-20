---
layout: pageshow
title: linux网络编程listen接口中backlog的含义
category: linux
description: 翻译自国外博客
---

**本篇博客参考并翻译自[国外博客](http://veithen.github.io/2014/01/01/how-tcp-backlog-works-in-linux.html),仅作学习分享**  
当应用程序通过`listen`系统调用将socket设置为监听状态时，需要设置其'backlog'的大小，这个值通常描述的是连接队列的大小。但这个队列具体如何工作的呢，需要好好推敲一下。　　
TCP通过三次握手建立连接的过程中，服务器端的状态有**SYN_RCVD**和**ESTABLISHED**两种状态，但是`accept`系统调用返回的是处于**ESTABLISHED**状态的连接，这样意味着TCP/IP协议栈有两种选择来实现backlog队列:  
1. 第一种方式是使用同一个队列，队列大小取决于`listen`系统调用时`backlog`的大小。当收到一个SYN请求，服务器返回一个SYN/ACK数据包然后将这个连接放入队列。之后收到对应的ACK后，这个连接状态变为ESTABLISHED可以交付给应用程序了。所以这个队列中包含的连接有两种状态：SYN_RCVD和ESTABLISHED。只有ESTABLISHED状态的连接才能被`accept`系统调用接受并返回，然后将这个连接从队列中移除。  
2. 第二种方式的使用两个队列，一个是SYN_RCVD状态队列，另一个是ESTABLISHED状态队列。连接经过三次握手的过程，首先进入SYN_RCVD队列，然后移出进入到ESTABLISHED队列。所以`accept`调用只从ESTABLISHED队列消费已握手成功的连接。这种情况下，backlog参数大小决定了ESTABLISHED队列大小。　　
历史上，BSD UNIX使用第一种方案。这样意味着只要队列满了，系统直接丢弃其他SYN请求(而不是返回**RST**)，这样客户端会进行超时重发。这个引自*section 14.5, listen Backlog Queue in W. Richard Stevens’ classic textbook TCP/IP Illustrated, Volume 3*  
但是在Linux上，`listen`就有些不同了，通过`man listen`可以看到一下说明　　
>  The  behavior of the backlog argument on TCP sockets changed with Linux
       2.2.  Now it specifies the  queue  length  for  completely  established
       sockets  waiting  to  be  accepted, instead of the number of incomplete
       connection requests.  The maximum length of the  queue  for  incomplete
       sockets  can be set using /proc/sys/net/ipv4/tcp_max_syn_backlog.  When
       syncookies are enabled there is no logical maximum length and this set‐
       ting is ignored.  See tcp(7) for more information.
       If    the   backlog   argument   is   greater   than   the   value   in
       /proc/sys/net/core/somaxconn, then it is  silently  truncated  to  that
       value;  the  default  value  in  this  file  is 128.  In kernels before
       2.4.25, this limit was a hard coded value, SOMAXCONN,  with  the  value
       128.

这说明Linux上使用的是第二种方案，SYN_RCVD队列大小通过系统参数设置，ESTABLISHED队列大小通过backlog设置。  
有趣问题是，当ESTABLISHED队列满后，SYN_RCVD队列中某个连接收到ACK想要完成三次握手进入ESTABLISHED队列时，系统内核会怎么处理？通过查看linux内核源码(可见原[博客](http://veithen.github.io/2014/01/01/how-tcp-backlog-works-in-linux.html)),Linux默认情况下是什么都不做，直接忽视ACK数据包，因为服务器端会有重发定时器，当超过一定时间后触发服务器端的重发机制，重新发送SYN/ACK数据包给客户端。
