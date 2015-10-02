---
layout: post
title: "UNIX网络编程读书笔记"
description: "unix network programming"
category: coding
tags: [c, linux]
published: true
comments: true
date: 2014-01-16 12:07:26 +0800
---
{% include JB/setup %}

### 前言

博客很久没有进行更新了，反思下还是自己太懒了。最近一段时间主要是以学习为主，这几天在读《UNIX网络编程》，加深自己对UNIX编程和网络编程的认识。

### Accept

前一段时间还在找工作的时候，看到过一道笔试题，说的是，accept发生在TCP三次握手的哪个阶段。书上有个图可以很好的解释这个问题。

<!--more-->

![TCP]({{ site.static_url }}/pic/2014-01-16-01.png)

### Bind & Listen

UNIX网络编程第4章习题中提出了两个问题。

1. __如果不调用listen，将会怎么样__
2. __如果不调用bind，直接调用listen又会怎么样__

我们来写程序来测试下会发生什么。

```c

/* A Simple Server */
#include <time.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(int argc, char **argv)
{
    int listenfd, connfd;
    struct sockaddr_in servaddr = {0};
    char buff[4096];
    time_t ticks;

    if ( (listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0 ) {
        return printf("Error to create socket\n");
    }
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(9999);
    if (0 != bind(listenfd, (struct sockaddr *)&servaddr, (socklen_t)sizeof(servaddr)))
        return printf("Error to bind addr\n");

    if (0 != listen(listenfd, 1024))
        return printf("Error to listen\n");
    while (1) {
        connfd = accept(listenfd, (struct sockaddr *)NULL, NULL);
        if (connfd == -1)
            printf("errno %d, %s\n", errno, strerror(errno));
        ticks = time(NULL);
        snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
        write(connfd, buff, strlen(buff));
        close(connfd);
    }
}
```

```c

/* A simple Client */
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int main(int argc, char **argv)
{
    int sockfd, n;
    char recvline[4097];
    struct sockaddr_in servaddr = {0};

    if (argc != 2)
        return printf("usage: %s <IPaddress>", argv[0]);
    if ( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
        return printf("socket error\n");
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(9999);
    if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
        return printf("inet_pton error for %s\n", argv[1]);
    if (connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0) {
        printf("errno %d, %s\n", errno, strerror(errno));
        return printf("connect error\n");
    }

    while ( (n = read(sockfd, recvline, 4096)) > 0 ) {
        recvline[n] = 0;
        fputs(recvline, stdout);
    }
    if (n < 0)
        return printf("read error\n");
    return 0;
}

```

分别注释掉Server中的bind和listen，运行后会有下面的结果。

* __不调用bind直接调用listen的情况__
    * 服务器端，listen调用会赋予监听套接字一个临时端口
    * 客户端连接的端口服务器未开放，不在监听状态，connect连接失败，errno为ECONNREFUSED，Connection refused

* __不调用listen的情况__
    * 服务器端调用accept失败，返回值-1，errno为EINVAL，Invalid argument
    * 客户端connect调用失败，errno为ETIMEDOUT，Connection timed out

