---
layout: post
title: "CGI WebServer"
description: "tell you how to write a cgi webserver"
category: code
tags: [c, server]
published: true
comments: true
date: 2013-10-25 19:52:41 +0800
---
{% include JB/setup %}

### 前言

好久没有写博客了，最近一段时间都在忙着确定工作和确定毕业论文题目等事情。好不容易忙完这些事情了，接下来工作方向定了，博客内容就尽量向工作内容靠拢，不会再像之前那样主题天马行空了。

<!--more-->

### CGI简介

在知道如何写一个 __CGI WebServer__ 之前，我们先看下什么是CGI。通用网关接口(Common Gateway Interface)，是一种重要的互联网技术，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序，请求数据。CGI描述了客户端和服务器程序之间传输数据的一种标准。

CGI的一个目的是要独立于任何语音额。WebServer无需在这个问题是对语音有任何了解。常见的编写CGI程序的语言有 _Perl_ 、 _Unix shell script_ 、 _Python_ 、 _Ruby_ 、 _PHP_ 、 _C/C++_  等。

CGI的工作原理，从WEB服务器的角度看，是在特定的位置，如 __http://www.example.com/test.cgi__ 定义了可以运行的CGI程序test.cgi。当收到一个匹配URL的请求，相应的程序就会被调用，并将客户端发送的数据作为输入。程序的输出会由Web服务器手机，并加上合适额HTTP Header，再发送回客户端。

CGI的缺点是每次请求都要生成一个程序的副本来运行，不能承受大的工作量。因此出现了 __FastCGI__ ，它会在第一次调用脚本时，缓存脚本编译后的版本。另一种办法是直接把解析器放在Web服务器中，这样就无需新建一个进程来执行脚本， __Apache__ 服务器就有很多这样的模块，比如 _mod_php_ 、 _mod_python_ 等等。

### 代码实现

在`GitHub`上搜到了一个实现， __https://github.com/klange/cgiserver__ ,代码很简洁，只有一千多行，但是该有的基本上都有了。它的主要流程是这样子的。

```c
    while (1) {
        /*
         * Accept an incoming connection and pass it on to a new thread.
         */
        unsigned int c_len;
        struct socket_request * incoming = calloc(sizeof(struct socket_request),1);
        c_len = sizeof(incoming->address);
        _last_unaccepted = (void *)incoming;
        incoming->fd = accept(serversock, (struct sockaddr *) &(incoming->address), &c_len);
        _last_unaccepted = NULL;
        pthread_create(&(incoming->thread), NULL, handleRequest, (void *)(incoming));
    }
```

对每一个连接都新建一个线程来处理。并且将相关的fd，threadid等信息传给线程。线程中的操作我们只重点看CGI部分，前面Header信息的读取和处理，包括Cookie，Referer，filename等的处理都有很详细的注释，就不详细说明了。

接下来就是经典的Linux管道编程了。fork，pipe，dup2，execl这一系列函数。

```c
    int cgi_pipe_r[2];
    int cgi_pipe_w[2];
    if (pipe(cgi_pipe_r) < 0) {
        fprintf(stderr, "Failed to create read pipe!\n");
    }
    if (pipe(cgi_pipe_w) < 0) {
        fprintf(stderr, "Failed to create write pipe!\n");
    }

    /*
     * Fork.
     */
    pid_t _pid = 0;
    _pid = fork();
```

先准备管道，然后fork出来子进程。将子进程的标准输出重定向到管道1，将子进程的标准输入重定向到管道0，这样子就能在父进程中处理子进程的输入输出了。

```c
    dup2(cgi_pipe_r[0],STDIN_FILENO);
    dup2(cgi_pipe_w[1],STDOUT_FILENO);
```

之后就是进入目录，使用`setenv`来设置环境变量。最重要的是设置 __SCRIPT_NAME__ , __SCRIPT_FILENAME__ 这些。之后就是执行脚本了。

```c
    char executable[1024];
    executable[0] = '\0';
    sprintf(executable, "./%s", _filename);
    execlp(executable, executable,(char *)0);
```

接下来就是从cgi_pipe中读取信息，首先了是读取头信息。

```c
	fprintf(socket_stream, "HTTP/1.1 200 OK\r\n");
    fprintf(socket_stream, "Server: " VERSION_STRING "\r\n");
    unsigned int j = 0;
    while (!feof(cgi_pipe)) {
        /*
         * Read until we are out of headers.
         */
        char * in = fgets(buf, CGI_BUFFER - 2, cgi_pipe);
        if (!in) {
            fprintf(stderr,"[warn] Read nothing [%d on %p %d %d]\n", ferror(cgi_pipe), cgi_pipe, cgi_pipe_w[1], _pid);
            perror("[warn] Specifically");
            buf[0] = '\0';
            break;
        }
        if (!strcmp(in, "\r\n") || !strcmp(in, "\n")) {
            /*
             * Done reading headers.
             */
            buf[0] = '\0';
            break;
        }
        if (!strstr(in, ": ") && !strstr(in, "\r\n")) {
            /*
             * Line was too long or is garbage?
             */
            fprintf(stderr, "[warn] Garbage trying to read header line from CGI [%zu]\n", strlen(buf));
            break;
        }
        fwrite(in, 1, strlen(in), socket_stream);
        //fprintf(socket_stream, "%s", buf);
        ++j;
    }
    if (j < 1) {
        fprintf(stderr,"[warn] CGI script did not give us headers.\n");
    }
    if (feof(cgi_pipe)) {
        fprintf(stderr,"[warn] Sadness: Pipe closed during headers.\n");
    }
```

然后是读取CGI的返回信息。

```c
    while (!feof(cgi_pipe)) {
        size_t read = -1;
        read = fread(buf, 1, CGI_BUFFER - 1, cgi_pipe);
        if (read < 1) {
            /*
             * Read nothing, we are done (or something broke)
             */
            fprintf(stderr, "[warn] Read nothing on content without eof.\n");
            perror("[warn] Error on read");
            break;
        }
        if (enc_mode == 0) {
            /*
             * Length of this chunk.
             */
            fprintf(socket_stream, "\r\n%zX\r\n", read);
        }
        fwrite(buf, 1, read, socket_stream);
    }
    if (enc_mode == 0) {
        /*
         * We end `chunked` encoding with a 0-length block
         */
        fprintf(socket_stream, "\r\n0\r\n\r\n");
    }
```

之后就是一些资源释放的操作了。
