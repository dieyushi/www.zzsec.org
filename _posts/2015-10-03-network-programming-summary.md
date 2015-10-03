---
layout: post
title: "网络编程的一些个人总结"
description: "personal network programming summary"
category: coding
tags: [c, linux]
published: true
comments: true
date: 2015-10-03 18:01:02 +0800
---
{% include JB/setup %}

下面是一些工作学习中总结的网络编程的方法。顺便提一句，又是一年没更新博客了，时间过的真快。

- 一些手册一定要好好读一下，比如 proc(5), capabilities(7), icmp(7), ipv6(7), netlink(7), raw(7), socket(7), tcp(7), udp(7)
- epoll 或者 select 处理事件时，可读事件时，read返回值-1，如果errno不为EAGAIN，可以认为失败，并关闭fd。read返回0，说明对方断开连接，此时也需要关闭fd。如果链路断了，如拔掉网线，需要是用keepalive来触发可写事件

<!--more-->

- 本地UDP发送过快也是会丢包的。非阻塞情况下的unix domain socket哪怕是STREAM的也是会丢包的
- 使用unix socket的一个linux专属技巧是abstract pathname，系统会自动管理sock path，详见man 7 unix，经测试使用的缺点为如果程序segment fault等原因崩溃，地址不会自动释放。下次绑定就会失败
- 使用unix socket通信相比于本地udp通信减少了校验和的计算。使用阻塞函数时，unix domain socket可以保证不丢包不乱序，但是当发送缓冲区满了的话则会阻塞。使用非阻塞操作时经测试会丢包
- 使用setsockopt设置发送缓冲时，SO_RCVBUF和SO_SNDBUF的最大值受系统设置限制(man 7 socket)。可以使用SO_RCVBUFFORCE和SO_SNDBUFFORCE来无视系统设置
- SIGPIPE信号，网络编程时一定要处理该信号。同样一般要设置的还有SO_REUSEADDR。当客户端close连接时，若server继续发送数据，会收到RST，继续写就会SIGPIPE
- Consider two peers, A and B, communicating via TCP. If B closes a socket and there is any data in B’s receive queue, B sends a TCP RST to A instead of following the standard TCP closing protocol, resulting in an ECONNRESET error return value from recv(  )
- 网络编程对事件进行封装，提供注册回调函数，在可读、可写时进行函数调用。一般用法，针对非阻塞情况，初始化时将可读事件注册，需要写的时候先写，写不下去的时候(errno=EAGAIN)再挂上可写事件，只要发送缓冲区还有空间，就是可写的
- 基于事件的编程框架，需要记录最后一次成功read或write的时间，如果idletime大于阈值，直接close
- 服务器编程可以设置最大的fd个数，然后一次性申请FileEvent数组，之后由fd到事件查询代价O(1)
- 针对非阻塞socket，connect返回EINPROGRESS时需要将fd加到可写事件监视集合中，当select()或者poll()返回可写事件时，需要用getsockopt去读SOL_SOCKET层面的SO_ERROR选项，SO_ERROR为0表示连接成功，否则为连接失败
- epoll ET模式的处理方式。读：只要可读就一直读，一直读到返回0，或者error = EAGAIN。写：只要可写就一直写，知道数据发送完，或者errno = EAGAIN
- socket read缓冲区最大值TCP可查看”/proc/sys/net/ipv4/tcp_rmem”， udp 65536
- 实现定时器时通常办法是select/poll/epoll接口，精度毫秒级；还有就是新增的系统调用timerfd_create 把时间变成了一个文件描述符，该“文件”在定时器超时的那一刻变得可读，高于poll的精度
- 在主动关闭连接时，可以先shutdown(fd, SHUT_WR)关闭写端，等对方close时再关闭读端。这样子的好处是如果对方已经发送了一些数据，这些数据不会漏收。这就要求对端在read返回0之后关闭连接或者shutdown写端
- 网络编程一种比较好的模型是“one loop per thread”，如果事件库不是线程安全的，则需要使用pipe或者
- socketpair通知，子线程接受到通知(fd可读)后处理，kernel 2.6.22加入了eventfd，是更好的通知方法
- TCP Nagle算法和TCP Delayed Ack机制可能会导致网络延时(Linux 40ms, Windows 200ms)，最容易产生问题的就是"Write-Write-Read”这种模型，发送端的Nagle算法和接收端的Delayed Ack会导致一直等到接收端delayed ack超时后数据才发送出去
- accept返回EMFILE，进程描述符用完了，无法创建新的socket，也无法close连接，会导致不断通知该可读事件，程序busy loop，cpu 100%，解决方法是事先准备一个nullfd=open(“/dev/null”)，close该fd，accept，close socket，然后再nullfd=open(“/dev/null”)，缺点是该方法线程不安全，多线程accept可能导致nullfd用于新socket创建，然后又处于busy loop中
- 关注下signalfd(2.6.22+)，eventfd(2.6.22+)，timerfd(2.6.25+)，分别可以做信号通知、事件通知和定时器，都可以select/poll/epoll，有事件发生时fd可读
- 清除不活跃的socket，使用定时器在处理大量连接时会很低效，可以采用timing wheel算法进行处理
- 在高并发的情况下，系统的fd数目可能是瓶颈，系统统一限制为/proc/sys/fs/file-max，该值的默认大小为内存总数/10k(fs/file_table.c)
