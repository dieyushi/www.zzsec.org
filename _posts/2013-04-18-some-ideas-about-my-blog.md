---
layout: post
title: "关于博客的一些想法"
description: "some ideas about this blog"
category: life
tags: [jekyll, golang, blog]
published: true
disqus: y
date: 2013-04-18 17:17:34 +0800
---


### 现状

经过一段时间的折腾，博客在`Heroku`，`Openshift`，`qiniu`分别存在一段时间，现在慢慢的稳定下来了，博客托管在`Stdyun`免费`Octopress`托管上，国内的访问速度还可以，如果一直这样不要求备案的话，就准备一直放在这里了。

博客使用了`Jekyll`和自己修改过的`ilost`主题。经过一段时间的使用，感觉`Jekyll`总体来说很不错，定制性很强，但是也有一些是使用中渐渐暴露出来的。其中有两点是我最不能接受的。

- Jekyll会给你安装一堆ruby gems，ruby升级和jekyll升级都会很烦人
- Jekyll编译时很慢，硬盘狂闪。

<!--more-->

### 想法

针对这些问题，我生出了自己写一套静态博客生成引擎的想法，下面是一些基本的设计想法。

- 使用golang编写，解决依赖问题，最后生成单一可执行文件，同时也可以作为web服务器使用
- 使用json格式的配置文件，golang还没有特别完美的yaml解析库，采用支持更好的json，同时对可读性影响不大
- 使用html/template包来处理模板，不采用liquid或者mustache
- 可以通过rsync部署
- 什么MarkDown啊代码高亮啊自动检测改动更新网站啊什么的肯定是要完美支持的
- 暂不考虑插件支持，只提供常见的统计、留言、分享等基本功能

现阶段已经有几款类似的软件在github上出现了，分别是

- [Gor](https://github.com/wendal/gor)
- [Jkl](https://github.com/bradrydzewski/jkl)
- [Hastie](https://github.com/mkaz/hastie)

这些多少和我的需求有些出入，为了学习，我也再多造一个轮子吧。
