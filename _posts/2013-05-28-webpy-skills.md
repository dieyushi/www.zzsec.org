---
layout: post
title: "web.py的一些坑"
description: "some skills about web.py framework"
category: web
tags: [python, framework]
published: true
disqus: y
date: 2013-05-28 17:10:32 +0800
---


### 前言

好久没写博客，一是因为这一段时间没有再去做题了，出差，回来之后准备了一家公司的面试，还要上党校。搞得自己每天看起来很忙，但是真的没做多少事情。最近抽时间再写一个网站，采用了web.py框架，记录点一开始遇到的问题。顺便缅怀下[Aaron Swartz ](http://zh.wikipedia.org/wiki/%E4%BA%9A%E4%BC%A6%C2%B7%E6%96%AF%E6%B2%83%E8%8C%A8)并吐槽下那简洁到坑爹的文档。

### 一些问题

- 使用apache + mod_wsgi部署时，报错说是找不到模板，使用了绝对路径才ok，可以使用`__file__`来做。
- web.py的模板系统的定义变量语句`$def with(name1, name2)`要放到页面起始位置。
- 使用了jquery的话，$会引起冲突，需要转义为`$$`。
- 可以使用`$:render.header()`和`$:render.foot()`来实现类似PHP include引用模板。
- 如果要向模板中传入很多值，可以用`locals()`来传值。`locals()`返回值是字典型。
- 使用apache + mod_wsgi部署时，import本地模块会抱错，解决方法在[http://webpy.org/install#apachemodwsgi](http://webpy.org/install#apachemodwsgi)中有提到。
- 使用apache + mod_wsgi部署时，session也要使用绝对路径。
- 使用sessions时，默认设置的cookie_path是错误的，具体表现是使用`python code.py`运行时是正确的，使用mod_wsgi时每次都会在服务器端重新生成一个新的session，相关代码在`web/session.py`中，解决办法是在定义session之前，设置参数，`web.config.session_parameters['cookie_path'] = '/'`。
