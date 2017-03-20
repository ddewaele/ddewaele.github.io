---
layout: post
title: Setting up CentOS networking in VirtualBox
categories: networking 
tags: [networking]
excerpt_separator: <!--more-->
---
## Introduction

This is a small tip on getting your networking service up and running in Centos.

If you've ever wondered why you're not getting an IP address on your CentOS VM in VirtualBox, even with the Bridged network adapter, then look no further....

![]({{ site.url }}/assets/images/virtualbox-bridged-adapter.png)

Notice how you're not getting an IP on the main `enp0s3` interface.

```
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:db:6f:94 brd ff:ff:ff:ff:ff:ff
```    

<!--more-->

To resolve this you need to enable the ONBOOT property of the network interface.

This can be done by executing

```
nmcli con mod enp0s3 connection.autoconnect yes
```

or by editing the ```/etc/sysconfig/network-scripts/ifcfg-enp0s3``` file and ensure that the `ONBOOT` parameter is set to yes.

```
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s3
UUID=22f5321b-0761-4790-9a21-94a4b97eecd9
DEVICE=enp0s3
ONBOOT=yes
```

After rebooting, you'll find that your VM has attached itself to your network

```
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:db:6f:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.40/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 6821sec preferred_lft 6821sec
    inet6 fe80::a00:27ff:fedb:6f94/64 scope link 
       valid_lft forever preferred_lft forever
```