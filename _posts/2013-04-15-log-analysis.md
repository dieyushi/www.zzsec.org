---
layout: post
title: "日志处理"
description: "web server access log analysis"
category: web
tags: [log, websecurity]
published:
---
{% include JB/setup %}

### 前言

最近要追踪入侵网站的ip，有网站访问日志，格式大概如下：

``` text
2012-01-15 00:11:18 W3SVC1 64.27.15.64 GET /IndexNews.ashx - 80 - 60.18.209.110 Mozilla/5.0+(Windows+NT+6.1)+AppleWebKit/535.2+(KHTML,+like+Gecko)+Chrome/15.0.874.120+Safari/535.2 200 0 0
2012-01-15 00:11:18 W3SVC1 64.27.15.64 GET /favicon.ico - 80 - 60.18.209.110 Mozilla/5.0+(Windows+NT+6.1)+AppleWebKit/535.2+(KHTML,+like+Gecko)+Chrome/15.0.874.120+Safari/535.2 404 0 2
2012-01-15 00:11:19 W3SVC1 64.27.15.64 GET /IndexNews.ashx - 80 - 60.18.209.110 Mozilla/5.0+(Windows+NT+6.1)+AppleWebKit/535.2+(KHTML,+like+Gecko)+Chrome/15.0.874.120+Safari/535.2 200 0 0
```
### 方法

首先要统计访问过某一页面的ip，并逆向排序，方法如下：

``` bash
$ ack "Admin_Main.aspx" *.log | awk 'BEGIN{FS=" "} {print $10}' | sort | uniq -c | sort -k1.1nr
```
