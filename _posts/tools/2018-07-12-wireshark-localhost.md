---
layout: pageshow
category: tools
title: wireshark捕获localhost数据及tcp.listen等问题
description: wireshark捕获本地回环数据
---

## wireshark捕获localhost方法
通过[RawCap](http://www.netresec.com/?page=RawCap)捕获localhost数据保存为**.pcap**文件，使用wireshark打开分析。  
RawCap可以添加环境变量进行全局访问，使用方法
>RawCap.exe 127.0.0.1 localhost.pcap

## tcp.listen监听队列
[来源](https://blog.csdn.net/tong_huijiao/article/details/53541197)
> listen函数仅由TCP服务器调用，它做两件事情： 
1、当socket函数创建一个套接口时，它被假设为一个主动套装口，也就是说，它是一个将调用connet发起连接的客户套接口。listen函数把一个未连接的套接口转换成一个被动套接口，指示内核应接受指向该套接口的连接请求。根据TCP状态转换图，调用listen导致套接口从CLOSED状态转换到LISTEN状态。 
2、本函数的第二个参数规定了内核应该为相应套接口排队的最大连接个数。 
为了更好的理解backlog参数，我们必须认识到内核为任何一个给定的监听套接口维护两个队列： 
1、未完成连接队列（incomplete connection queue），每个这样的SYN分节对应其中一项：已由某个客户发出并到达服务器，而服务器正在等待完成相应的TCP三路握手过程。这些套接口处于SYN_RCVD状态。 
2、已完成连接队列（completed connection queue），每个已完成TCP三路握手过程的客户对应其中一项。这些套接口处于ESTABLISHED状态。 
当来自客户的SYN到达时，TCP在未完成连接队列中创建一个新项，然后响应以三路握手的第二个分节：服务器的SYN响应，其中稍带对客户SYN的ACK（即SYN+ACK）。这一项一直保留在未完成连接队列中，直到三路握手的第三个分节（客户对服务器SYN的ACK）到达或者该项超时为止（曾经源自Berkeley的实现为这些未完成连接的项设置的超时值为75秒）。如果三路握手正常完成，该项就从未完成连接队列移到已完成连接队列的队尾。当进程调用accept时，已完成连接队列中的队头项将返回给进程，或者如果该队列为空，那么进程将被投入睡眠，直到TCP在该队列中放入一项才唤醒它。    

因而对端口监听后，系统会响应客户端的连接请求，将完成握手的连接放入ESTABLISHED队列，等待Accept函数进行获取。
