# 无人值守安装

# 安装操作系统的方法
    1、插入光盘
    2、插入U盘
    3、网络安装
    4、自动化安装

## pxe介绍
    PXE(Pre-boot Execution Environment,预启动执行环境)由Intel和Systemsoft公司于1999年9月20日公布的技术；通过有线网卡启动计算机，不依赖本地存储设备（如硬盘）或本地已安装的操作系统；

    客户端/Server的工作模式：PXE客户端(PXE客户端可以是服务器、笔记本电脑或者其他装有PXE启动代码的机器）通过网络从服务器下载镜像，并由此支持通过网络启动操作系统。

    在启动过程中，客户端要求服务器分配IP地址，再用TFTP（trivial file transfer protocol）协议下载一个启动软件包到本机内存中执行，由这个启动软件包完成终端（客户端）基本软件设置，从而引导预先安装在服务器中的终端操作系统。PXE可以引导多种操作系统

    PXE客户端会调用网际协议(IP)、用户数据报协议(UDP)、动态主机设定协议(DHCP)、小型文件传输协议(TFTP)等网络协议；


①  PXE 客户端发送UDP广播请求 
PXE 客户端从自己的PXE网卡启动，通过PXE BootROM(自启动芯片)会以UDP(简单用户数据报协议)发送一个广播请求，向本网络中的DHCP服务器索取IP。

②  DHCP服务器提供信息 
DHCP服务器收到客户端的请求，验证是否来至合法的PXE 客户端的请求，验证通过它将给客户端一个“提供”响应，这个“提供”响应中包含了为客户端分配的IP地址、pxelinux启动程序(TFTP)位置，以及配置文件所在位置。

③  PXE客户端请求下载启动文件 
客户端收到服务器的“回应”后，会回应一个帧，以请求传送启动所需文件。这些启动文件包括：pxelinux.0（PXE启动文件）、pxelinux.cfg/default（PXE引导配置文件）、vmlinuz（Linux内核文件）、initrd.img（系统启动文件）等文件。

④  TETP服务器响应客户端请求并传送文件 
当服务器收到客户端的请求后，他们之间之后将有更多的信息在客户端与服务器之间作应答, 用以决定启动参数。BootROM由TFTP通讯协议从tftp服务器下载启动安装程序所必须的文件(pxelinux.0、pxelinux.cfg/default)。default文件下载完成后，会根据该文件中定义的引导顺序，启动Linux安装程序的引导内核。

⑤  请求下载自动应答文件 
客户端通过pxelinux.cfg/default文件成功的引导Linux安装内核后，安装程序首先必须确定你通过什么安装介质来安装linux，如果是通过网络安装(NFS, FTP, HTTP)，则会在这个时候初始化网络，并定位安装源位置。接着会读取default文件中指定的自动应答文件ks.cfg所在位置，根据该位置请求下载该文件。

⑥  客户端安装操作系统 
将ks.cfg文件下载回来后，通过该文件找到http镜像，并按照该文件的配置请求下载安装过程需要的软件包。

# 环境说明：
简介：SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，是 Linux历史上最杰出的新安全子系统。NSA是在Linux社区的帮助下开发了一种访问控制体系，在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。SELinux 默认安装在  Red Hat Enterprise Linux 上，其他发行版也可使用。

[root@test html]# getenforce # 查看当前selinux状态
[root@test html]# setenforce 0 # 临时关闭
[root@test html]# vim /etc/selinux/config
SELINUX=disabled # 永久关闭
 
[root@test html]# systemctl stop firewalld   # 关闭防火墙
[root@test html]# systemctl diable firewalld # 禁止开机启动防火墙
[root@test html]# systemctl mask firewalld   # 冻结防火墙

    # 设置DHCP
    [root@Mm dhcp]# vim dhcpd.conf 
        option domain-name "test.com";
        option domain-name-servers 114.114.114.114;
        log-facility local7;
        subnet 192.168.22.0 netmask 255.255.255.0 {
        range 192.168.22.100 192.168.22.240;
        option routers 192.168.22.1;
        option broadcast-address 192.168.22.255;
        default-lease-time 600;
        max-lease-time 7200;
        next-server 192.168.22.24; # 指定TFTP服务器的IP地址
        filename "/pxelinux.0"; # 告知客户端从TFTP根目录下载
        }

    # 安装TFTP服务器 
    [root@test ~]# yum install tftp-server -y
    [root@test ~]# systemctl restart tftp
    [root@test ~]# systemctl enable tftp
    [root@test ~]# cd /var/lib/tftpboot/ # tftp服务器的根目录

# 挂载、卸载
    windows下挂载，就是给磁盘提供一个盘符，比如插入U盘后，给个盘符I
    Linux下，不像Windows可以有C,D,E,多个目录，Linux只有一个根目录/。在装系统时，我们分配给linux的所有区都在/下的某个位置，比如/home等等。
    插入了新硬盘，分了新磁盘区sdb1。它现在还不属于/。Linux下，mount挂载的作用，就是将一个设备（通常是存储设备）挂接到一个已存在的目录上。访问这个目录就是访问该存储设备。



第四步：创建ks文件
	1. ks文件的作用：在安装操作系统的过程中，需要大量的和服务器交互操作，为了减少这个交互过程，kickstart就诞生了。使用kickstart，只需事先定义好一个Kickstart自动应答配置文件ks.cfg，并让安装程序知道该配置文件的位置，在安装过程中安装程序就可以自己从该文件中读取安装配置，这样就避免了在安装过程中多次的人机交互，从而实现无人值守的自动化安装。
	2. ks文件生成的两种方式：
		a. Centos提供了一个图形化的kickstart配置工具。在任何一个安装好的Linux系统上运行该工具，就可以很容易地创建你自己的kickstart配置文件。kickstart配置工具命令为system-config-kickstart。
阅读kickstart配置文件的手册。用任何一个文本编辑器都可以创建你自己的kickstart配置文件。