---
layout: post
title: "sqlmap的一些小技巧"
description: "some skills about sqlmap"
category: web
tags: [sqli, websecurity]
published: true
disqus: y
date: 2013-06-13 22:28:58 +0800
---


### 前言

很多人都使用sqlmap来进行SQL注入测试，但是很多人只是简简单单的`current-user`，`current-db`，`-D`，`-T`，`--dump`这样子来做，其实sqlmap还有很多很强大的功能，这里简单的总结下。


#### POST注入

有两种方法来进行post注入，一种是使用`--data`参数，将post的key和value用类似GET方式来提交。二是使用`-r`参数，sqlmap读取用户抓到的POST请求包，来进行POST注入检测。

#### 查看payload

之前一直是加本地代理，然后用burpsuit来看sqlmap的payload，到现在才发现用`-v`参数就可以实现。一直认为`-v`实现的只是控制警告，debug信息级别。实际上使用`-v 3`就可以显示注入的payload，4,5,6还可以显示HTTP请求，HTTP响应头和页面。

<!--more-->

#### 使用google搜索

sqlmap可以测试google搜索结果中的sql注入，很强大的功能吧。使用方法是参数`-g`。不过感觉实际使用中这个用的还是很少的。

#### 请求延时

在注入过程中请求太频繁的话可能会被防火墙拦截，这时候`--delay`参数就起作用了。可以设定两次HTTP请求间的延时。有的web程序会在多次错误访问后屏蔽所有请求，这样就导致之后所有的测试无法进行，绕过这个策略可以使用`--safe-url`，每隔一段时间去访问一个正常的页面。

#### 伪静态页面

有些web服务器进行了url rewrite或者网站是伪静态的，无法直接提供测试参数，这样子可以使用`*`来代替要测试的参数。

#### 执行系统命令

当数据库支持，并且当前用户有权限的时候，可以执行系统命令，使用`--os-cmd`或者`--os-shell`，具体的讲，当可以执行多语句的时候，会尝试用UDF（MySQL，PostgrepSQL）或者xp_cmdshell（MSSQL）来执行系统命令。不能执行多语句时，仍然会尝试创建一个webshell来执行语句，这时候就需要web的绝对路径了。总体来说，成功率偏低，不过个人也有成功的经验～

#### 测试等级

sqlmap使用`--level`参数来进行不同全面性的测试，默认为1，不同的参数影响了使用哪些payload，2时会进行cookie注入检测，3时会进行`useragent`检测。

### 参考

- [https://github.com/sqlmapproject/sqlmap/wiki/Usage](https://github.com/sqlmapproject/sqlmap/wiki/Usage)
