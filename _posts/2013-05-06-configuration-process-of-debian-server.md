---
layout: post
title: "Debian服务器配置"
description: "configuration process of Debian server"
category: life
tags: [debian, linux]
published: true
comments: true
date: 2013-05-06 01:29:20 +0800
---
{% include JB/setup %}

### 前言

由于工作的原因，手上有几台debian的服务器，全新安装的6.0。根据需要，开通了pptp服务，并将不需要的服务关闭。现在记录下整个流程。

### 添加公钥

默认/root目录下是没有.ssh目录的，需要手动先建立，然后执行下面的命令即可。

    scp ~/.ssh/id_rsa.pub root@your-server-ip:/root/.ssh/authorized_keys

### 卸载服务

系统默认开通了exim4, portmap和nfs-common服务。由于不需要这些服务，卸载

<!--more-->

    /etc/init.d/exim4 stop
    /etc/init.d/portmap stop
    /etc/init.d/nfs-common stop
    apt-get --purge remove nfs-common portmap
    apt-get --purge remove exim4 exim4-base exim4-config exim4-daemon-light

其中有一台不知道怎么还给安装了apache2和postgresql，卸载

    /etc/init.d/apache2 stop
    /etc/init.d/postgresql stop
    apt-get remove apache2-utils
    apg-get remove postgresql-8.4

### pptpd安装配置

直接从源里面安装就可以

    apt-get -y install pptpd

编辑pptpd.conf,配置ip，localip是服务器ip，remoteip是分配给用户的ip。这两个最好不要同服务器内网或者用户连接的内网相同。

```bash
cat >> /etc/pptpd.conf<<EOF
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
EOF
```

编辑pptpd-options,添加

    ms-dns 8.8.8.8
    ms-dns 8.8.4.4

为windows客户端提供DNS服务器

编辑`/etc/sysctl.conf`,开启ipv4转发，否则连接vpn后只能访问服务器，不能访问服务器以外的资源。

```bash
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
sysctl -p
```

设置iptables NAT转发

    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

保存iptables规则，在`/etc/network/interfaces`中添加pre-up，以便重启后继续有效。

    iptables-save > /etc/iptables.up.rules
    cat >> /etc/network/interface <<EOF
    pre-up iptables-restore < /etc/iptables.up.rules
    EOF

重启服务

    /etc/init.d/pptpd restart

搞定，最后查看开放的端口以及对于的服务

    netstat -anop

