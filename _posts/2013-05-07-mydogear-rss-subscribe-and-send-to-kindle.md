---
layout: post
title: "狗耳朵与Kindle推送"
description: "mydogear rss subscribe and send to kindle via chrome or pc"
category: life
tags: [kindle]
published: true
disqus: y
date: 2013-05-07 19:11:26 +0800
---


### 前言

关于kindle的评价和购买已经在之前的一篇文章里提过了([链接](http://www.zzsec.org/2013/04/kindle/)),这次推荐Kindle的一些服务来提高阅读体验。主要是通过[狗耳朵](http://www.mydogear.com/)订阅RSS和使用软件或插件向Kindle推送文章。

### 狗耳朵

一直以来都是使用`Google Reader`来订阅感兴趣的网站或博客内容，随着订阅的条目越来阅读，只要一天没有看，就可以看到GR的未读数目300多条，然后就感到压力很大，有时候都感觉自己有点轻微的强迫症了，强迫自己随时看新出现的订阅内容。但是订阅的博客按质量和深度可以分为几个层次，有的只需要随便看看有点印象就可以，而有的却需要好好的静下心来阅读。这个需求在电脑上经常会被各种各样突然出现的事情打断，自从有了kindle，就可以把这个事情放到晚上睡觉前来做，每天拿出半个到一个多小时的时间来消化今天的内容。这就需要将rss内容推送到kindle上。

狗耳朵实现的正是这个需求。 __狗耳朵是全文RSS订阅推送服务，支持Kindle，多看，Nook等移动电子阅读设备。__

<!--more-->

最初使用狗耳朵的时候，是不支持图片推送的，显示的只是图片的链接，这样在看一些有图示的文章的时候比较痛苦，要经常开网络。但是最近狗耳朵更新后，添加了VIP用户，VIP用户支持10M以内的图片推送了。就为这个功能，我买了一个月的VIP服务体验了下，只有一次没把所有的图显示出来，那次是有个文章贴了大量的图片，超过了10M。

狗耳朵可以支持google reader订阅，可以自己添加RSS订阅(VIP用户)，排版良好，定时推送，个人使用非常满意，同时推荐大家购买VIP服务享受最好的订阅阅读体验和支持下这个站点，每月的费用几乎可以忽略不计。

### Send to Kindle

有很多软件都叫做`Send to Kindle`，本文提到的是指amazon的`Send to Kindle`[客户端](http://www.amazon.com/gp/sendtokindle/pc)和Chrome `Send to Kindle (by Klip.me)`[插件](https://chrome.google.com/webstore/detail/send-to-kindle-by-klipme/ipkfnchcgalnafehpglfbommidgmalan)。

PC端主要实现的是将书籍推送到`Kindle Personal Document`，Amazon的服务器可以将常见格式转成Kindle支持的格式，然后推送到Kindle阅读。
Chrome插件主要实现的是把当前网页推送到kindle，只提取正文，排版一般良好。

### 总结

这些优秀的软件和硬件的出现，可以使我们充分的利用大量的碎片时间来阅读，剩下的就是个人自己寻找优秀的博客和网站来订阅了。
