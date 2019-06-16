# CentOS6系统启动

## POST硬件自检
    计算机接通电源，硬件自检

## BIOS 
    # 选择引导设备（U盘、硬盘、光驱、网络等等），比如选择硬盘，BIOS将控制权交给MBR。
    MBR（主引导记录），是一个特殊的引导扇区。MBR存放了硬盘的分区信息、主引导程序（grub）
    MBR512字节大小=分区信息的大小为64字节、grub大小为446字节。

## GRUB
    从磁盘中读取grub，将控制权交给grub，硬盘中存在多个操作系统，在grub界面选择其中的一个操作系统或内核。

## 加载内核
    完成硬件的初始化等工作

## 执行/sbin/init程序
    init是系统的第一个进程，PID为1，以后所有的进程都是在它的基础上产生的
### init进程的工作流程
+ init进程读取的第一个脚本是/etc/inittab，文件定义了系统默认的运行级别，init进程根据inittab文件来启动第几个运行级别。

    运行级别：当前操作系统正在运行的功能级别。级别从0~6，每个运行级别都具有不同的功能。
    0：系统停机状态，系统的默认运行级别肯定不能设置为0。
    1：单用户工作状态，root权限，用于系统维护，禁止远程登录，跟windows下面的安全模式登录类似。
    2：多用户状态，没有nfs支持
    3：完整的多用户模式，有nfs支持，登录后进入控制台命令行模式
    4：系统未使用，保留一般不用。
    5：X11控制台，登录后进入GUI图形模式
    6：系统正常关闭并重启，默认运行级别肯定不能设置为6
    [root@localhost ~]# vim /etc/inittab #修改默认的运行级别，重启之后生效
    [root@localhost ~]# init 3 # 临时切换运行级别
    [root@localhost ~]# init 6 # 重启

+ 第一个根据运行级别启动的脚本是/etc/rc.d/rc.sysinit，决定了系统的一些基本配置。比如设置系统的时钟、设置主机名、激活selinux、定义网络配置参数、挂载分区、加载swap。

+ 根据运行级别启动对应的目录下的脚本文件，以运行级别5为例，执行/etc/rc5.d/目录下的脚本
    [root@localhost ~]# ls /etc/rc5.d/
    S开头的文件代表开机启动的服务，K开头的文件代表关机要执行的任务

+ 执行/etc/rc.d/rc.local文件
    自定义开机启动的命令

+ 执行/bin/login，等待用户登录

## init进程的优缺点：
    优点：
        1、概念简单
        2、确定的执行顺序，脚本文件一个执行完毕再执行下一个脚本，有益于错误排查
    缺点：
        完全确定的执行顺序是init最大的缺陷。如果Linux系统只用于服务器，那么漫长的启动过程没问题。但是比如说想ArchLinux这样的桌面系统，漫长的启动过程对桌面用户来说，是不能接受的。

## init进程的改进upstart
    由于init的弊端，Ubuntu的开发人员决定重新设计和开发一个全新的init系统，upstart。upstart基于事件机制，比如U盘插入到USB接口中，udev得到内核通知，发现该设备，这个就是全新的时间。upstart在感知到该事件之后触发相应的任务。upstart也加快了系统的启动时间。
    理由：init是同步阻塞的，一个脚本运行时，后续脚本必须等待。upstart将不想关的服务并行启动。

## 主角init进程的改进systemd
    systemd 是Linux 系统中最新的init系统

### 整个发展史
    init->upstart->systemd





# CentOS7系统启动
## POST硬件自检
    计算机接通电源，硬件自检

## BIOS 
    # 选择引导设备（U盘、硬盘、光驱、网络等等），比如选择硬盘，BIOS将控制权交给MBR。
    MBR（主引导记录），是一个特殊的引导扇区。MBR存放了硬盘的分区信息、主引导程序（grub）
    MBR512字节大小=分区信息的大小为64字节、grub大小为446字节。

## GRUB
    从磁盘中读取grub，将控制权交给grub，硬盘中存在多个操作系统，在grub界面选择其中的一个操作系统或内核。

## 加载内核
    完成硬件的初始化等工作

## systemd
    执行systemd进程。# PID为1，是所有进程的祖先
    [root@test ~]# ps -aux |grep systemd #查看进程ID号

## systemd概述
    systemd开启和监督整个系统是基于单元(unit)的概念。系统初始化需要做的事情很多，需要启动后台服务（SSHD），需要做配置工作（比如挂载文件系统）等等。在系统启动过程中的每一步都被systemd抽象为一个unit。
    常见的unit类型：
    1、Service：用于守护进程的启动、停止、重启
    2、Target：此类unit为其他unit进行逻辑分组，本身不做什么，只是引用其他的unit。
    systemd使用target来处理引导和服务管理过程，这些systemd里面的target被用于分组不同的引导单元以及启动同步进程。

    systemd执行的第一个目标是default.target。其实default.target是指向graphical.target的软链接。
    [root@test ~]# ls -l /etc/systemd/system/default.target  
    # graphical.target这个target就好像init中的run level5

    runlevel 0 poweroff.target
    runlevel 1 resuce.target
    runlevel 2,3,4  multi-user.target
    runlevel 5 graphical.target
    runlevel 6 reboot.target

    第一步：graphical.target
        [root@test ~]# cat /etc/systemd/system/default.target
        #  This file is part of systemd.
        #
        #  systemd is free software; you can redistribute it and/or modify it
        #  under the terms of the GNU Lesser General Public License as published by
        #  the Free Software Foundation; either version 2.1 of the License, or
        #  (at your option) any later version.

        [Unit]
        Description=Graphical Interface
        Documentation=man:systemd.special(7)
        Requires=multi-user.target
        Wants=display-manager.service
        Conflicts=rescue.service rescue.target
        After=multi-user.target rescue.service rescue.target display-manager.service
        AllowIsolate=yes

        [root@test ~]#  /etc/systemd/system/default.target.wants/ # 执行这个目录下的服务
        [root@test ~]# /usr/lib/systemd/system/multi-user.target #执行这个target
    
    第二步：multi-user.target 
        # 为多用户支持设置系统环境，非root用户在这个阶段的引导过程中启用，防火墙等相关的服务也会在这个阶段启动。multi-user.target将自己的子单元存在/etc/systemd/system/multi-user.target.wants/目录下
        [root@test ~]# cat /usr/lib/systemd/system/multi-user.target
        #  This file is part of systemd.
        #
        #  systemd is free software; you can redistribute it and/or modify it
        #  under the terms of the GNU Lesser General Public License as published by
        #  the Free Software Foundation; either version 2.1 of the License, or
        #  (at your option) any later version.

        [Unit]
        Description=Multi-User System
        Documentation=man:systemd.special(7)
        Requires=basic.target
        Conflicts=rescue.service rescue.target
        After=basic.target rescue.service rescue.target
        AllowIsolate=yes

    第三步：basic.target
        # 这个target用于启动普通的服务，主要是图形化管理相关的服务
        [root@test ~]# vim /usr/lib/systemd/system/basic.target
        #  This file is part of systemd.
        #
        #  systemd is free software; you can redistribute it and/or modify it
        #  under the terms of the GNU Lesser General Public License as published by
        #  the Free Software Foundation; either version 2.1 of the License, or
        #  (at your option) any later version.

        [Unit]
        Description=Basic System
        Documentation=man:systemd.special(7)

        Requires=sysinit.target
        After=sysinit.target
        Wants=sockets.target timers.target paths.target slices.target
        After=sockets.target paths.target slices.target

        [root@test ~]# cd /etc/systemd/system/basic.target.wants/

    第四步：sysinit.target 
        # 这个target启动重要的系统访问，比如系统挂载、内存空间交换和设备、以及内核的补充选项等。
        [root@test ~]# vim /usr/lib/systemd/system/sysinit.target
        #  This file is part of systemd.
        #
        #  systemd is free software; you can redistribute it and/or modify it
        #  under the terms of the GNU Lesser General Public License as published by
        #  the Free Software Foundation; either version 2.1 of the License, or
        #  (at your option) any later version.

        [Unit]
        Description=System Initialization
        Documentation=man:systemd.special(7)
        Conflicts=emergency.service emergency.target
        Wants=local-fs.target swap.target
        After=local-fs.target swap.target emergency.service emergency.target

        [root@test ~]# cd /etc/systemd/system/sysinit.target.wants/

    第五步：
        local-fs.target # 不启动与用户相关的服务，仅仅处理底层核心服务
            [root@test ~]# vim /usr/lib/systemd/system/local-fs.target

## systemd与init相比的优点
    1、启动速度快、各个服务并行运行。
    2、systemd目标：尽可能启动更少的进程，尽可能将更多的进程并行启动。
    3、systemd提供按需启动的能力。init在系统初始化的时候会将所有可能用到的后台服务进程全部运行。
    4、systemd自带了日志服务journald。
    5、实现事务性依赖关系管理。虽然systemd最大限度的并发执行很多有依赖关系的工作，但是类似于网络和sshd这个的工作，还是存在先后依赖关系的，无法并发执行。systemd维护一个“事务一致性”，保证所有相关的服务，都可以正常启动，而不出现相关依赖、导致死锁的情况。