# 系统启动

## POST硬件自检
    计算机接通电源，硬件自检

## BIOS 
    # 选择引导设备（U盘、硬盘、光驱、网络等等），比如选择硬盘，BIOS将控制权交给MBR。
    MBR（主引导记录），是一个特殊的引导扇区。MBR存放了硬盘的分区信息、主引导程序（grub）
    MBR512字节大小=分区信息的大小为64字节、grub大小为446字节。

## GRUB
    #从磁盘中读取grub，将控制权交给grub（REHL7 grub2），硬盘中存在多个操作系统，在grub界面选择其中的一个操作系统或内核。

## systemd
    GRUB加载内核（Linux），完成内核的初始化。# PID为1，是所有进程的祖先
    [root@test ~]# ps -aux |grep systemd #查看进程ID号

# systemd概述
    systemd执行的第一个目标是default.target。其实default.target是指向graphical.target的软链接。


    [root@test ~]# find / -name graphical.target # 在/目录下查找一个名为graphical.\ target的文件
    /usr/lib/systemd/system/graphical.target
    [root@test ~]# find / -name default.target
    /etc/systemd/system/default.target
    [root@test ~]# ls -l /etc/systemd/system/default.target 
    lrwxrwxrwx. 1 root root 36 5月  11 20:31 /etc/systemd/system/default.target -> /lib/systemd/system/graphical.target

    [root@test system]# cd /usr/lib/systemd/system #进入到该目录中去
    在系统启动过程中需要执行很多个不同的进程，用target做一个统一的管理
    #target用于控制多个service、socket

    default.target是graphical.target的软链接
        [root@test ~]# ls -l /etc/systemd/system/default.target 
        lrwxrwxrwx. 1 root root 36 5月  11 20:31 /etc/systemd/system/default.target -> /lib/systemd/system/graphical.target
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

    multi-user.target # 为多用户支持设置系统环境，非root用户在这个阶段的引导过程中启用，防火墙等相关的服务也会在这个阶段启动。multi-user.target将自己的子单元存在/etc/systemd/system/multi-user.target.wants/目录下
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

basic.target# 这个target用于启动普通的服务，主要是图形化管理相关的服务
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

sysinit.target # 这个target启动重要的系统访问，比如系统挂载、内存空间交换和设备、以及内核的补充选项等。
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

local-fs.target # 不启动与用户相关的服务，仅仅处理底层核心服务
    [root@test ~]# vim /usr/lib/systemd/system/local-fs.target


# RHEL6（init）--->upstart--->systemd
    init源自于UNIX,最早的Linux发行版都是使用init的。在init中术语：runlevel（运行级别）
    0 #关机
    1 #救援模式
    2
    3 #命令行模式
    4
    5 #图形化界面
    6 #重启
    RHEL6按部就班的执行系统的初始化
