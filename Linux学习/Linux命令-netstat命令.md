---
title: Linux命令-netstat命令
copyright: true
date: 2018-10-23 12:53:47
tags:
     - Linux命令
categories: Linux
---

用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

## 语法

netstat [-acCeFghilMnNoprstuvVwx]

## 选项

```
-a   或–all                           显示所有连线中的Socket
-c   或–continuous               持续列出网络状态
-h  或–help                         在线帮助
-l  或–listening                     显示监控中的服务器的Socket
-n  或–numeric                    不解析主机名
-t  或–tcp                            显示TCP 传输协议的连线状况
-u或–udp                            显示UDP传输协议的连线状况
```

注意：LISTEN和LISTENING的状态只有用-a或者-l才能看到

## 网络连接状态详解

共有12中可能的状态，前面11种是按照TCP连接建立的三次握手和TCP连接断开的四次挥手过程来描述的。
1)、LISTEN:首先服务端需要打开一个socket进行监听，状态为LISTEN./ *The socket is listening for incoming connections. 侦听来自远方TCP端口的连接请求* /

2)、 SYN_SENT:客户端通过应用程序调用connect进行active open.于是客户端tcp发送一个SYN以请求建立一个连接.之后状态置为SYN_SENT./*The socket is actively attempting to establish a connection. 在发送连接请求后等待匹配的连接请求* /

3)、 SYN_RECV:服务端应发出ACK确认客户端的 SYN,同时自己向客户端发送一个SYN. 之后状态置为SYN_RECV/ *A connection request has been received from the network. 在收到和发送一个连接请求后等待对连接请求的确认* /

4)、ESTABLISHED: 代表一个打开的连接，双方可以进行或已经在数据交互了。/ *The socket has an established connection. 代表一个打开的连接，数据可以传送给用户* /

5)、 FIN_WAIT1:主动关闭(active close)端应用程序调用close，于是其TCP发出FIN请求主动关闭连接，之后进入FIN_WAIT1状态./ *The socket is closed, and the connection is shutting down. 等待远程TCP的连接中断请求，或先前的连接中断请求的确认* /

6)、CLOSE_WAIT:被动关闭(passive close)端TCP接到FIN后，就发出ACK以回应FIN请求(它的接收也作为文件结束符传递给上层应用程序),并进入CLOSE_WAIT./ *The remote end has shut down, waiting for the socket to close. 等待从本地用户发来的连接中断请求* /

7)、FIN_WAIT2:主动关闭端接到ACK后，就进入了 FIN-WAIT-2 ./ *Connection is closed, and the socket is waiting for a shutdown from the remote end. 从远程TCP等待连接中断请求* /

8)、LAST_ACK:被动关闭端一段时间后，接收到文件结束符的应用程 序将调用CLOSE关闭连接。这导致它的TCP也发送一个 FIN,等待对方的ACK.就进入了LAST-ACK ./ *The remote end has shut down, and the socket is closed. Waiting for acknowledgement. 等待原来发向远程TCP的连接中断请求的确认* /

9)、TIME_WAIT:在主动关闭端接收到FIN后，TCP 就发送ACK包，并进入TIME-WAIT状态。/ *The socket is waiting after close to handle packets still in the network.等待足够的时间以确保远程TCP接收到连接中断请求的确认* /

10)、CLOSING: 比较少见./ *Both sockets are shut down but we still don’t have all our data sent. 等待远程TCP对连接中断的确认* /

11)、CLOSED: 被动关闭端在接受到ACK包后，就进入了closed的状态。连接结束./ *The socket is not being used. 没有任何连接状态* /

12)、UNKNOWN: 未知的Socket状态。/ *The state of the socket is unknown.* /

SYN: (同步序列编号,Synchronize Sequence Numbers)该标志仅在三次握手建立TCP连接时有效。表示一个新的TCP连接请求。
ACK: (确认编号,Acknowledgement Number)是对TCP请求的确认标志,同时提示对端系统已经成功接收所有数据。
FIN: (结束标志,FINish)用来结束一个TCP回话.但对应端口仍处于开放状态,准备接收后续数据。

## 常用实例

1） 查看TCP的连接状态

```
netstat -natlp | awk '{print $6}' | sort | uniq -c |sort -rn
netstat -n | awk '/^tcp/ {++S[$NF]};END {for(a in S) print a,S[a]}'
netstat -n | awk '/^tcp/ {print $NF}' | sort |uniq -c | sort -rn
```

2） 查找请求数较多的前20个IP（常用于查找攻来源）

```
netstat -anpl | grep 80 | grep tcp | awk '{print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -nr | head -20
netstat -ant | awk '/:80/{split($5,ip,":")';++A[ip[1]]}END{for(i in A) print A[i],i}' | sort -rn |head -20 
tcpdump -i eth0 -tnn dst port 80 -c 1000 | awk -F"." '{print $1"."$2"."$3"."$4}' | sort | uniq -c | sort -nr | head -20
```

