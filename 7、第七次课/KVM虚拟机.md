# KVM安装

## 第一步

    VMWARE开启CPU支持。三个选项全部打钩
    [root@test ~]# egrep 'vmx|svm' /proc/cpuinfo  #查看CPU是否支持虚拟化，cpuinfo文件用于查看cpu的信息

    [root@test yum.repos.d]# cd /etc/yum.repos.d/

    [root@test yum.repos.d]# rm -rf * # 删除当前目录下的所有文件

    [root@test yum.repos.d]# wget http://mirrors.163.com/.help/CentOS7-Base-163.repo

    [root@test yum.repos.d]# yum clean all
    [root@test yum.repos.d]# yum makecache

    [root@test ~]# yum grouplist 
    [root@test ~]# yum groupinstall "虚拟化主机" –y

    [root@test 桌面]# wget http://mirrors.163.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso # 下载Centos mini版镜像


    [root@test 桌面]# yum install virt-install -y # 安装kvm的命令行管理工具
    [root@test ~]# yum install virt-manager
    [root@test ~]# yum install virt-viewer -y

    kvm 虚拟CPU和内存，qemu虚拟其他的硬件，直接调用kvm或qemu的接口很麻烦，所有有了libvirt这个工具。

    [root@test 桌面]# systemctl status libvirtd
    [root@test 桌面]# ps -aux |grep libvirtd


    modules模块：
    Linux操作系统的核心具有模块化的特性，每一个模块都有特定的功能。可以按需求载入不同的模块。
    [root@test 桌面]# lsmod # 这个命令显示当内核已经装载的模块
    [root@test 桌面]# lsmod | grep kvm # 查看当前内核是否支持kvm
    [root@test 桌面]# cat /proc/modules | grep kvm

    1、图形化界面管理KVM虚拟机
    退出虚拟机：ctrl+alt
    2、命令行界面管理KVM虚拟机
    [root@test ~]# virt-install --name test --disk path=/var/lib/libvirt/images/test.img,size=5,format=raw --memory 1024 --cdrom 桌面/CentOS-7-x86_64-Minimal-1810.iso --autostart
    
    --cdrom 指定安装的系统镜像
    
    --memory 指定内存大小 1024M

    --disk 指定虚拟机的硬盘路径
        path 硬盘路径
        size 硬盘大小
        format 硬盘格式

    [root@test ~]# virsh # 是一个交互式的管理工具
    virsh # domstate test # 查看虚拟机的状态
    virsh # net-info default # 查看网络信息
    virsh # dominfo default # 查看虚拟机的信息

    virsh # reboot test # 重启虚拟机
    域 test 正在被重新启动

    virsh # shutdown test # 关闭虚拟机
    域 test 被关闭

    virsh # domstate test # 查看虚拟机的运行状态
    running

    virsh # start test # 开启虚拟机

    
    virsh # suspend test # 暂停虚拟机
    域 test 被挂起

    virsh # setmem test 512M # 动态设置内存的大小

    virsh # setmem test 888M

    virsh # setmem test 8880M
    错误：无效参数：无法将内存设置为高于最大内存
