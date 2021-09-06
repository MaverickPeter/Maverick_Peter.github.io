---
layout:     post
title:      "服务器双网卡配置"
subtitle:   " \"工作记录\""
date:       2021-08-25 13:00:00
author:     "xxc"
header-img: "img/post-bg-networking.jpg"
catalog: true
tags:
    - 工作记录

---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


### 前情提要

网卡1走的路由器，在路由器的局域网下，可以上网。网卡2走的内网，可以通过IP直接内网访问。之前一直都是利用路由器不同端口的方法来访问路由器下的各台服务器，比较麻烦。现在想给每台服务器分配一个内网IP，直接访问，但同时也想通过路由器上网。

### 静态IP设置

#auto lo
#iface lo inet loopback

#路由器，有网

auto 网卡1
iface 网卡1 inet static
address 192.168.1.xxx
netmask 255.255.255.0
gateway 192.168.1.1

#内网访问

auto 网卡2
iface 网卡2 inet static
address 10.xx.xxx.xx
netmask 255.255.255.0



#### 临时添加路由

sudo route add -net 10.xx.xxx.0/24 gw 10.xx.xxx.1 dev 网卡2

-net 表示这是添加到网络的路由

gw 表示走后面的网关

dev 表示走这个网卡设备



#### 永久添加路由

sudo nano /etc/sysconfig/static-routes

any net 192.168.1.0/24 gw 192.168.1.1 dev 网卡1

any net 10.xx.xxx.0/24 gw 10.xx.xxx.1 dev 网卡2