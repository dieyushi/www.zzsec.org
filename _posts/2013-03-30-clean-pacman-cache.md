---
layout: post
title: "Pacman缓存清理"
description: "clean pacman cache"
category: coding
tags: [archlinux,python]
---
{% include JB/setup %}

### 前言
有一段时间没有清理pacman的缓存了，今天`du -hs`了一下子发现缓存居然有5G之多，所以决定写一个小程序来清理多余的包，每个包只保留当前的最新版本。其实很早之前就想写这么一个小程序了，今天才动手花了半个小时把它实现了。

### 程序
代码是用python写的，把程序放在缓存目录，运行即可。

<!--more-->

```python
#! /usr/bin/env python

import os

dir =  os.path.dirname(os.path.realpath(__file__))
pkgs = {}
pkgs_ext = {}
for f in os.listdir(dir):
    s = f.split('-')
    if len(s) >= 4:
        pkg_name = s[:-3]
        _pkg_name = ""
        for _p_name in pkg_name:
            _pkg_name = _pkg_name + _p_name + '-'
        _pkg_name = _pkg_name[:-1]
        if _pkg_name in pkgs.keys():
            pkgs[_pkg_name].append(s[-3]+'-'+s[-2])
        else:
            pkgs[_pkg_name] = []
            pkgs[_pkg_name].append(s[-3]+'-'+s[-2])
        pkgs_ext[_pkg_name+'-'+s[-3]+'-'+s[-2] ] = s[-1]
    else:
        print s

del_list = []
pkgs_maxver = {}
for pkg in pkgs.keys():
    max_ver = ""
    for ver in pkgs[pkg]:
        if ver > max_ver:
            max_ver = ver
    pkgs_maxver[pkg] = max_ver 
    print pkgs[pkg]
    print max_ver

for pkg in pkgs.keys():
    for ver in pkgs[pkg]:
        if ver != pkgs_maxver[pkg]:
            fname = pkg + '-' + ver
            fname = fname + "-"+pkgs_ext[fname]
            del_list.append(fname)


for file4del in del_list:
    fullpath = os.path.join(dir,file4del)
    try:
        os.remove(fullpath)
    except:
        pass
```
