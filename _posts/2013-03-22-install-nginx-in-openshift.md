---
layout: post
title: "Install Nginx in OpenShift"
description: "teach you how to install nginx in openshift"
category: life
tags: [blog, config]
published: true
comments: true
date: 2013-03-22 22:39:12 +0800
---
{% include JB/setup %}

### 前言
这几天将博客搬移到了`OpenShift`上，使用它的`DIY app`，搭建了一个`Nginx`环境用来跑我的静态博客。`OpenShift`使用的是`Amazon`的主机，国内访问有点慢，通过`CloudFlare`来做`CDN`感觉效果不是很明显，但是考虑到博客主要是自己访问，通过代理速度还可以接受，毕竟`CloudFlare`能够使网站在全美国的`ping`在10ms左右。

### OpenShift简介
[**OpenShift**](https://openshift.redhat.com)是Red Hat公司推出的一个PaaS，支持Java、PHP、Perl、Ruby等很多的开发框架，开发者通过`git`来部署代码，也可以通过ssh登录到主机进行配置。OpenShift对于免费用户的一些限制参考这个[页面](https://www.openshift.com/faq/how-many-applications-can-i-deploy-and-what-resource-limitations-would-they-have)。

<!--more-->

### 创建OpenShift applications
创建一个新的app，名为blog。

    rhc app create blog diy-0.1

查看blog的相关信息。

    rhc app show -a blog

使用上个命令列出的信息ssh到服务器，形式大概是：

    ssh <random-strings>@blog-<your-namespace>.rhcloud.com

系统中默认会有一些定义好了的环境变量([点我查看](https://www.openshift.com/page/openshift-environment-variables)),再接下来的配置中会用到。

### 安装Nginx
现在就可以进行`nginx`的安装了。首先要下载`nginx`的源码

    cd $OPENSHIFT_TMP_DIR
    wget http://nginx.org/download/nginx-1.2.7.tar.gz
    tar zxf nginx-1.2.7.tar.gz

编译nginx需要PRCE。

    cd $OPENSHIFT_TMP_DIR
    wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.31.tar.bz2
    tar jxf prce-8.31.tar.bz2
    cf nginx-1.2.7
    ./configure --prefix=$OPENSHIFT_DATA_DIR --with-pcre=$OPENSHIFT_TMP_DIR/pcre-8.31
    make Install

编译完成后，`nginx`将会被装到`$OPENSHIFT_DATA_DIR`中。

### 配置Nignx
`OpenShift`只允许使用`$OPENSHIFT_INTERNAL_IP`和`OPENSHIFT_INTERNAL_PORT`，在`nginx`的`http`，`server`和`location`不能直接使用这些环境变量，所以我们用了点小技巧。

    vi $OPENSHIFT_DATA_DIR/conf/nginx.conf

编辑`listen`的值为

    http {
        ...
        server {
            listen $OPENSHIFT_IP:$OPENSHIFT_PORT;
            server_name localhost;
            ...
        }
        ...
    }

把这个作为模板，在app启动的时候生成配置文件

    mv $OPENSHIFT_DATA_DIR/conf/nginx.conf $OPENSHIFT_DATA_DIR/conf/nginx.conf.template

### 编写start和stop脚本
在程序代码中编辑`.openshift/action_hooks/start`，内容为：

    cd blog
    vim .openshift/action_hooks/start

```bash
#!/bin/bash
# The logic to start up your application should be put in this
# script. The application will work only if it binds to
# $OPENSHIFT_INTERNAL_IP:8080
# nohup $OPENSHIFT_REPO_DIR/diy/testrubyserver.rb $OPENSHIFT_INTERNAL_IP $OPENSHIFT_REPO_DIR/diy > $OPENSHIFT_DIY_LOG_DIR/server.log 2>&1 &
# replace the $OPENSHIFT_INTERNAL_IP and $OPENSHIFT_INTERNAL_PORT before starting up the server
sed -e "s/`echo '$OPENSHIFT_IP:$OPENSHIFT_PORT'`/`echo $OPENSHIFT_INTERNAL_IP:$OPENSHIFT_INTERNAL_PORT`/" $OPENSHIFT_DATA_DIR/conf/nginx.conf.template > $OPENSHIFT_DATA_DIR/conf/nginx.conf
nohup $OPENSHIFT_DATA_DIR/sbin/nginx > $OPENSHIFT_DIY_LOG_DIR/server.log 2>&1 &
```
    
    vi ./openshift/action_hooks/stop

```bash
#!/bin/bash
ps -ef | grep nginx | while read line
do
  kill -9 `echo $line | awk '{ print $2 }'`
done
exit 0
```

提交代码，然后就可以用`rhc app start|stop|restart blog`了。

>`PS`:当`nginx`使用3xx重定向请求时，指向的是`appname.rhcloud.com:8080`,8080是内部监听端口，从外部请求8080端口将会超时，解决这个问题需要在nginx配置的`http`部分中添加`port_in_redirect off;`。

### 参考
1. [http://shawhu.org/2012-10/Jekyll-OpenShift/](http://shawhu.org/2012-10/Jekyll-OpenShift/)
2. [https://www.openshift.com/blogs/lightweight-http-serving-using-nginx-on-openshift](https://www.openshift.com/blogs/lightweight-http-serving-using-nginx-on-openshift)
