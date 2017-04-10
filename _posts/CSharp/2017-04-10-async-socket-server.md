---
layout: pageshow
title: 网络编程实践-异步通信服务器中踩的一些坑
category: CSharp
description: 编写服务器socket软件时遇到了一些问题
---

## Socket服务器

因为目前对网络通信知识了解不深，不敢乱讲话，还是多看博客，如[协议森林](http://www.cnblogs.com/vamei/archive/2012/12/05/2802811.html)。  
Scoket服务器通过TCP/IP协议，与客户端连接，进行网络通信。连接方式又存在长连接和短连接两种。短连接即每次客户端请求数据通信完成后断开与服务器的连接，但是TCP/IP建立握手连接很消耗计算机资源；长连接即一旦客户端与服务器建立连接后，通过保活，来建立长期稳定连接。

## 方案选择

物联网系统，类似摩拜单车的后台服务器系统，我个人猜测：按照摩拜单车的实时性高的特点，应该是采用的长连接，每个地区上万台的子设备与服务器通信，通过心跳和数据帧与服务器保活，定期给服务器发送数据（包括GPS信息）。  
针对上万台子设备的连接，对每个连接单独开一个线程进行处理显然不现实，所以该采取异步通信方式，只有子设备有数据活动时才会调用线程来接收。  
针对物联网系统，主要有收发、处理和存储三块功能。  
**收发**：与远程客户端进行数据通信。  
**处理**：对数据进行解码、计算等操作。  
**存储**：将处理后的数据存储。  
这三块功能可以分别使用三个线程进行单独操作，不会相互堵塞。

## 异步通信遇到的坑

异步通信参考了[MSDN官方示例](https://msdn.microsoft.com/en-us/library/fx6588te.aspx)。  
这里踩过了一个坑，值得拿出来说说。  
关键点在与`ReadCallback(IAsyncResult ar)`中，当远程客户端主动端开连接时，并不会引发`SocketException`，而是会不停出发`ReadCallback`，每次读取到是字节为0，这样就会进入`while(true)`循环，长期占用CPU资源。如果是双核系统，一旦有两个远程客户端意外端开连接，CPU占用会达到100%。

```csharp
 private void ReadCallback(IAsyncResult ar)
        {
            SocketObject state = (SocketObject)ar.AsyncState;
            Socket clientSocket = state.workSocket;
            
            try
            {
               

                int bytesRead = clientSocket.EndReceive(ar);
                byte[] buffer = new byte[bytesRead];
                if (bytesRead == 0)
                {
                	 // when a client disconnect the socket initiatively and properly, it will trigger ReadCallback and bytesRead always be 0, it will be while(true) to occupy cpu
                if (clientSocket.Poll(1000, SelectMode.SelectRead)) {
                    throw new SocketException();
                }
                }
                else
                {
                    Array.Copy(state.buffer, 0, buffer, 0, bytesRead); // avoid reference value
                    Tuple<EndPoint, byte[]> rcvBytesTuple = new Tuple<EndPoint, byte[]>(clientSocket.RemoteEndPoint,
                        buffer);

                    RcvBytesQueue.Enqueue(rcvBytesTuple); // ConcurrentQueue<Tuple<EndPoint, byte[]>> RcvBytesQueue

                    if (!Dispatcher.RcvResetEvent.WaitOne(0)) // if WaitOne(0) return false, the ManualResetEvent has not set
                    {
                        Dispatcher.RcvResetEvent.Set(); // when data received, inform other thread to process  
                    }
                }

                // ready for receive next bytes
                clientSocket.BeginReceive(state.buffer, 0, SocketObject.BufferSize, 0, new AsyncCallback(ReadCallback),state);
            }
            catch (Exception ex)
            {
                // do something 
            }
        }

```

因此，要对这种情况主动抛出异常，通过[Socket.Poll](https://msdn.microsoft.com/en-us/library/ee435495.aspx)来监测连接是否正常，来避免进入死循环无效占用CPU资源。
