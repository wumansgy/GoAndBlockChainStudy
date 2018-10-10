---
title: Linux命令-ping命令
copyright: true
date: 2018-10-10 13:28:09
tags:
     - Linux命令
categories: Linux
---

Linux系统的ping命令是常用的网络命令，它通常用来测试与目标主机的连通性，我们经常会说“ping一下某机器，看是不是开着”、不能打开网页时会说“你先ping网关地址192.168.1.1试试”。它通过发送ICMP ECHO_REQUEST数据包到网络主机（send ICMP ECHO_REQUEST to network hosts），并显示响应情况，这样我们就可以根据它输出的信息来确定目标主机是否可访问（但这不是绝对的）。有些服务器为了防止通过ping探测到，通过防火墙设置了禁止ping或者在内核参数中禁止ping，这样就不能通过ping确定该主机是否还处于开启状态。

linux下的ping和windows下的ping稍有区别,linux下ping不会自动终止,需要按ctrl+c终止或者用参数-c指定要求完成的回应次数。

## 语法

`ping(选项)(参数)`

## 选项

```
-d：使用Socket的SO_DEBUG功能；
-c<完成次数>：设置完成要求回应的次数；
-f：极限检测；
-i<间隔秒数>：指定收发信息的间隔时间；
-I<网络界面>：使用指定的网络界面送出数据包；
-l<前置载入>：设置在送出要求信息之前，先行发出的数据包；
-n：只输出数值；
-p<范本样式>：设置填满数据包的范本样式；
-q：不显示指令执行过程，开头和结尾的相关信息除外；
-r：忽略普通的Routing Table，直接将数据包送到远端主机上；
-R：记录路由过程；
-s<数据包大小>：设置数据包的大小；
-t<存活数值>：设置存活数值TTL的大小；
-v：详细显示指令的执行过程。
```

## 参数

目的主机：指定发送ICMP报文的目的主机。

## 功能

ping命令用于：确定网络和各外部主机的状态；跟踪和隔离硬件和软件问题；测试、评估和管理网络。如果主机正在运行并连在网上，它就对回送信号进行响应。每个回送信号请求包含一个网际协议（IP）和 ICMP 头，后面紧跟一个 tim 结构，以及来填写这个信息包的足够的字节。缺省情况是连续发送回送信号请求直到接收到中断信号（Ctrl-C）。

ping 命令每秒发送一个数据报并且为每个接收到的响应打印一行输出。ping 命令计算信号往返时间和(信息)包丢失情况的统计信息，并且在完成之后显示一个简要总结。ping 命令在程序超时或当接收到 SIGINT 信号时结束。Host 参数或者是一个有效的主机名或者是因特网地址。

## 常用实例

1）ping的通的情况

```
# ping 192.168.120.205
PING 192.168.120.205 (192.168.120.205) 56(84) bytes of data.
64 bytes from 192.168.120.205: icmp_seq=1 ttl=64 time=0.720 ms
64 bytes from 192.168.120.205: icmp_seq=2 ttl=64 time=0.181 ms
--- 192.168.120.205 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.181/0.293/0.720/0.214 ms
```

2）ping不同的情况

```
# ping 192.168.120.202
PING 192.168.120.202 (192.168.120.202) 56(84) bytes of data.
From 192.168.120.204 icmp_seq=1 Destination Host Unreachable
From 192.168.120.204 icmp_seq=2 Destination Host Unreachable
-- 192.168.120.202 ping statistics ---
8 packets transmitted, 0 received, +6 errors, 100% packet loss, time 7005ms
, pipe 4
```

3）ping指定次数

```
# ping -c 10 192.168.120.206
PING 192.168.120.206 (192.168.120.206) 56(84) bytes of data.
64 bytes from 192.168.120.206: icmp_seq=1 ttl=64 time=1.25 ms
64 bytes from 192.168.120.206: icmp_seq=2 ttl=64 time=0.260 ms
64 bytes from 192.168.120.206: icmp_seq=3 ttl=64 time=0.242 ms
64 bytes from 192.168.120.206: icmp_seq=4 ttl=64 time=0.271 ms
64 bytes from 192.168.120.206: icmp_seq=5 ttl=64 time=0.274 ms
64 bytes from 192.168.120.206: icmp_seq=6 ttl=64 time=0.295 ms
64 bytes from 192.168.120.206: icmp_seq=7 ttl=64 time=0.269 ms
64 bytes from 192.168.120.206: icmp_seq=8 ttl=64 time=0.270 ms
64 bytes from 192.168.120.206: icmp_seq=9 ttl=64 time=0.253 ms
64 bytes from 192.168.120.206: icmp_seq=10 ttl=64 time=0.289 ms

--- 192.168.120.206 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9000ms
rtt min/avg/max/mdev = 0.242/0.367/1.251/0.295 ms
```

4）时间间隔和次数限制的ping

```
# ping -c 10 -i 0.5 192.168.120.206
PING 192.168.120.206 (192.168.120.206) 56(84) bytes of data.
64 bytes from 192.168.120.206: icmp_seq=1 ttl=64 time=1.24 ms
64 bytes from 192.168.120.206: icmp_seq=2 ttl=64 time=0.235 ms
64 bytes from 192.168.120.206: icmp_seq=3 ttl=64 time=0.244 ms
64 bytes from 192.168.120.206: icmp_seq=4 ttl=64 time=0.300 ms
64 bytes from 192.168.120.206: icmp_seq=5 ttl=64 time=0.255 ms
64 bytes from 192.168.120.206: icmp_seq=6 ttl=64 time=0.264 ms
64 bytes from 192.168.120.206: icmp_seq=7 ttl=64 time=0.263 ms
64 bytes from 192.168.120.206: icmp_seq=8 ttl=64 time=0.331 ms
64 bytes from 192.168.120.206: icmp_seq=9 ttl=64 time=0.247 ms
64 bytes from 192.168.120.206: icmp_seq=10 ttl=64 time=0.244 ms

--- 192.168.120.206 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 4499ms
rtt min/avg/max/mdev = 0.235/0.362/1.241/0.294 ms
```

5）多参数使用

```
# ping -i 3 -s 1024 -t 255 192.168.120.206
PING 192.168.120.206 (192.168.120.206) 1024(1052) bytes of data.
1032 bytes from 192.168.120.206: icmp_seq=1 ttl=64 time=1.99 ms
1032 bytes from 192.168.120.206: icmp_seq=2 ttl=64 time=0.694 ms
1032 bytes from 192.168.120.206: icmp_seq=3 ttl=64 time=0.300 ms
1032 bytes from 192.168.120.206: icmp_seq=4 ttl=64 time=0.481 ms
1032 bytes from 192.168.120.206: icmp_seq=5 ttl=64 time=0.415 ms
1032 bytes from 192.168.120.206: icmp_seq=6 ttl=64 time=0.600 ms
1032 bytes from 192.168.120.206: icmp_seq=7 ttl=64 time=0.411 ms
1032 bytes from 192.168.120.206: icmp_seq=8 ttl=64 time=0.281 ms
1032 bytes from 192.168.120.206: icmp_seq=9 ttl=64 time=0.318 ms
1032 bytes from 192.168.120.206: icmp_seq=10 ttl=64 time=0.362 ms
1032 bytes from 192.168.120.206: icmp_seq=11 ttl=64 time=0.408 ms
1032 bytes from 192.168.120.206: icmp_seq=12 ttl=64 time=0.445 ms
1032 bytes from 192.168.120.206: icmp_seq=13 ttl=64 time=0.397 ms
1032 bytes from 192.168.120.206: icmp_seq=14 ttl=64 time=0.406 ms
1032 bytes from 192.168.120.206: icmp_seq=15 ttl=64 time=0.458 ms

--- 192.168.120.206 ping statistics ---
15 packets transmitted, 15 received, 0% packet loss, time 41999ms
rtt min/avg/max/mdev = 0.281/0.531/1.993/0.404 ms
```

说明：

-i 3 发送周期为 3秒 -s 设置发送包的大小为1024 -t 设置TTL值为 255

参考链接：

<http://man.linuxde.net/ping>

<http://www.cnblogs.com/peida/archive/2013/03/06/2945407.html>