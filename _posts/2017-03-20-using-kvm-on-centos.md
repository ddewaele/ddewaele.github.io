---
layout: post
title: Using KVM on CentOS
tags: [centos, kvm, vm]
excerpt_separator: <!--more-->
---
## Introduction

Having the ability to run a Kernel Basd Virtual Machine (KVM) on a hosted solution is a luxury you won't easily find on most cloud providers (Amazon, Google, Azure). On my hosting provider, I can get a quad core i7 32gb RAM server for less than 70 euros / month, a fraction of the cost that you would pay with a cloud provider. 

I tought it might be interesting to see what kind of virtualization options I had on Centos and stumbled upon KVM, LibVRT and Qemu. 

<!--more-->

## Installing some software
    
    yum -y install qemu-kvm libvirt virt-install bridge-utils
    yum -y install kvm virt-manager libvirt virt-install qemu-kvm xauth dejavu-lgc-sans-fonts
    yum -y install /usr/bin/virt-sysprep

## Verify install

    [root@CentOS-72-64-minimal ~]# lsmod | grep kvm 
    kvm_intel             162153  12 
    kvm                   525409  1 kvm_intel


## Verify service

    [root@CentOS-72-64-minimal ~]# systemctl status libvirtd
    ● libvirtd.service - Virtualization daemon
       Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
       Active: active (running) since Sat 2017-03-18 13:32:02 CET; 8h ago
         Docs: man:libvirtd(8)
               http://libvirt.org
     Main PID: 11490 (libvirtd)
       Memory: 6.1G
       CGroup: /system.slice/libvirtd.service
               ├─11490 /usr/sbin/libvirtd
               ├─11563 /sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
               └─11564 /sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper

If the service isn't started, start it using 

    systemctl status libvirtd


## Creating a VM

You can create virtual machine with the following command

    virt-install --name centos7 --ram 4096 \
    --disk path=/var/kvm/images/centos7.img,size=30 \
    --vcpus 2 --os-type linux --os-variant rhel7 \
    --network bridge=virbr0 --graphics none --console pty,target_type=serial \
    --location 'http://ftp.iij.ad.jp/pub/linux/centos/7/os/x86_64/' 
    --extra-args 'console=ttyS0,115200n8 serial'

The command above will start the installation process. You can then continue the text based installation and let it boot.
Aftr the installation you can shutdown the machine.

you can always return to the host by pressing `CTRL-]`.

Executing `virsh console centos7` will get you back into the VM.

## Commands 

### Starting VMs

    virsh start centos7
    virsh start centos7 --console

### Stopping VMs

    virsh shutdown centos7 
    virsh destroy centos7

### Console VMs
    virsh console centos7


### Removing VMs

    virsh undefine centos7

## Cloning a VM

Now that you've done the entire installation of a VM, it might be interesting to make a clone of it.

Create a clone by issueing the following command

    virt-clone --original centos7 --name template --file /var/kvm/images/template.img

This will generate a template.xml file and a tmplate image in the provided location.

Now that we have a template, it's very easy to create some additional VMs basd on that template.

    virt-clone --original template --name centos-a --file /var/kvm/images/centos-a.img
    virt-clone --original template --name centos-b --file /var/kvm/images/centos-b.img
    virt-clone --original template --name centos-manager --file /var/kvm/images/centos-manager.img

## Importing an existing template

If you ever want to migrate VMs on another machine, its sufficient to copy the image file and the xml file, and execute the command below:

    virsh define template.xml 


## File storage

Where does all of this get stored ?

- XML files are located in  `/etc/libvirt/qemu/`
- Image files are located in `/var/kvm/images/`
