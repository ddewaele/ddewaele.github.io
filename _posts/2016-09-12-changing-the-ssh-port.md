---
layout: post
title: Changing the SSH port
categories: networking ssh
tags: [networking, ssh]
excerpt_separator: <!--more-->
---
# Introduction

I'm sure most of you have experienced this scenario : A server is put online, and although you've secured it properly, you still see people attempting to brute force attack your server by attempting to login via SSH.

```
sshd[25808]: input_userauth_request: invalid user ubnt [preauth]
sshd[25808]: Received disconnect from 91.224.161.103: 11:  [preauth]
sshd[25810]: Invalid user test from 91.224.161.103
sshd[25810]: input_userauth_request: invalid user test [preauth]
sshd[25810]: Received disconnect from 91.224.161.103: 11:  [preauth]
sshd[25812]: Invalid user tech from 91.224.161.103
sshd[25812]: input_userauth_request: invalid user tech [preauth]
sshd[25812]: Received disconnect from 91.224.161.103: 11:  [preauth]
sshd[25814]: Received disconnect from 91.224.161.103: 11:  [preauth]
```

Although you've setup your server to only allow SSH key based authentication (and as such nobody can login with a password), people are still trying to find their way in. You can dramatically recude these number of attacks by switching your SSH daeon to a non standard port.

In this post, I'll show you how to change that port, 

<!--more-->

Centos comes with SELinux, and when enabled, SELinux by default wlll only allow the ssh daemon to run on port 22.

Any attempt to restart the ssh daemon and using a different port will result in the following error

```
sshd[778]: Received signal 15; terminating.
sshd[27278]: error: Bind to port 2022 on 0.0.0.0 failed: Permission denied.
sshd[27278]: error: Bind to port 2022 on :: failed: Permission denied.
sshd[27278]: fatal: Cannot bind any address.
sshd[27286]: error: Bind to port 2022 on 0.0.0.0 failed: Permission denied.
sshd[27286]: error: Bind to port 2022 on :: failed: Permission denied.
sshd[27286]: fatal: Cannot bind any address.
sshd[27291]: Server listening on 0.0.0.0 port 22.
sshd[27291]: Server listening on :: port 22.
```

To check if SELinux is enabled, execute the `sestatus` command:

```
[root@localhost ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   enforcing
Mode from config file:          enforcing
Policy version:                 21
Policy from config file:        targeted
```

When disabled you'll see this

```
[root@localhost ~]# sestatus 
SELinux status:                 disabled
```

When SELinux is disabled you should be able to put the SSHD daemon on any port you like. However, when SELinux is enabled, you'll need to do some extra work.

To view the allowed ports for ssh you can execute the following command

```
[root@ip-172-30-0-30 ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      22
```

If you don't have the semanage command in your CentOS distro (ex: the minimal distro), you can install it using ```sudo yum install policycoreutils-python```

To allow the SSH daemon to also run on port 2022, you need to execute this (this can take a while to return)

```
[root@ip-172-30-0-30 ~]# semanage port -a -t ssh_port_t -p tcp 2022
```

After that, you'll be able to run the ssh daemon on port 2022

```
[root@ip-172-30-0-30 ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      2022, 22
```

As you can see the daemon starts up fine:

```
sshd[27291]: Received signal 15; terminating.
sshd[27367]: Server listening on 0.0.0.0 port 2022.
sshd[27367]: Server listening on :: port 2022.
```
