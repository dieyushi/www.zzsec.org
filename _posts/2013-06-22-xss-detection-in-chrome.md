---
layout: post
title: "插件实现XSS检测"
description: "XSS detection in chrome extension"
category: web
tags: [xss]
published: true
comments: true
date: 2013-06-22 11:04:11 +0800
---
{% include JB/setup %}

### 前言

有很多软件可以做XSS的检测，如[XSSer](http://xsser.sourceforge.net/)等等。本文实现的着重点不是XSS的检测，而是XSS触发之后的行为检测，呃，直接点说就是拦截发送Cookie这一行为。

### 实现

通过使用`chrome.* APIs`，我们可以实现请求拦截，cookie获取等行为。具体实现如下：

<!--more-->

```javascript
chrome.webRequest.onBeforeRequest.addListener(function( details ) {
    chrome.tabs.query({'active': true}, function (tabs) {
        var url = tabs[0].url;

        CookieString = "";
        chrome.cookies.getAll({url: url}, function(cookies) {
            if (cookies.length == 0)
              return;
            for (var i = 0; i < cookies.length - 1; i++) {
              if (cookies[i].httpOnly == false) {
                CookieString += cookies[i].name + "=" + cookies[i].value + "; ";
              }
            }
            CookieString += cookies[cookies.length-1].name + "=" + cookies[cookies.length-1].value;

            if (details.url.indexOf(escape(CookieString)) > 0 || details.url.indexOf(CookieString) > 0){
                chrome.tabs.query({'active': true}, function (tabs) {
                    chrome.tabs.remove(tabs[0].id, function(){});
                });
                alert("cookies stealen");
            }
        });
    });
    },
    {urls: ["<all_urls>"]},
    ["blocking"]
);
```

其实上面的代码只是关闭了标签页，不过由于是异步调用，cookie在关闭之前早就发出去了。怎么block这个webrequest呢。

### 参考

1. https://developer.chrome.com/extensions/getstarted.html
2. https://developer.chrome.com/extensions/cookies.html
3. https://developer.chrome.com/extensions/tabs.html
