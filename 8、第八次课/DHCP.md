# DHCP 
    动态获取IP地址

    # 初次连接网络
    1、寻找DHCP服务器
    当DHCP客户端第一次登录网络的时候，计算机发现本地没有任何IP，将以广播方式发送ＤＨＣＰdiscover发送信息来寻找DHCP服务，即想255.255.255.255发送特定的信息。网络上每一台电脑都能接受该广播信息，但是DHCP服务器回复。
    2、分配IP地址
    在网络中接收到DHCP discover信息，做出回复，将未分配使用的ip随机选择一个给客户端，并向客户端发包含IP地址等相关信息
    3、接收IP地址
    客户端接收到了DHCP的OFFER提供信息之后，选择第一个接收到的信息，再以广播的方式告诉DHCP服务器，告诉服务器客户端选定的ip地址
    4、IP地址确认
    当DHCP服务器收到DHCP客户端回答的REQUEST请求信息后，向客户端发送一个提供IP地址等相关信息的DHCP ACK确认信息，告诉客户端可以使用这个IP地址。客户端将IP地址与网卡绑定。

# Server Centos7
    1、关闭NAT模式的DHCP功能
    2、设置Server为静态IP地址

    [root@test ~]# yum install dhcp -y 
    [root@test ~]# vim /etc/dhcp/dhcpd.conf  # 该文件为空，需要拷贝一个配置文件魔板过来
    [root@test ~]# cat /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example > /etc/dhcp/dhcpd.conf #根据模板生成一个新的配置文件
    
    [root@test ~]# vim /etc/dhcp/dhcpd.conf 
    配置文件说明：
    # 服务器的域名
    option domain-name "test.com";
    #服务器分配给客户机的dns地址
    option domain-name-servers 192.168.24.254;
    #服务器提供给客户机的IP地址的使用租期，单位默认是秒
    default-lease-time 600;
    #最大的租期是7200秒
    max-lease-time 7200;
    #日志级别，这里设置是local7
    log-facility local7;
    #下面的内容是定义的作用域，如果作用域中的配置信息和上面的全局配置不一样，那么默认是作
        用域的优先生效
    subnet 192.168.24.0 netmask 255.255.255.0 {       
    range 192.168.24.222 192.168.24.230;           可以分发的地址范围
    option domain-name-servers 192.168.24.254;     dns服务器地址
    option domain-name "test.com";                 域名
    option routers 192.168.24.1;                    网关
    option broadcast-address 192.168.24.255;        广播地址
    default-lease-time 600;        
    max-lease-time 7200;
    }
    [root@test ~]# systemctl restart dhcpd
    [root@test ~]# systemctl status dhcpd
    [root@test ~]# netstat -ntulp |grep dhcp
    [root@test ~]# systemctl enable dhcpd # 开机自启

    [root@test dhcpd]# vim /var/lib/dhcpd/dhcpd.leases
    # 客户端查看租赁地址的详细内容


    [root@localhost ~]# vim /etc/dhcp/dhcpd.conf     # 静态IP地址设置 
    host fantasia { 
    hardware ethernet 00:0C:29:6C:98:2B; #mac 地址
    fixed-address 192.168.24.224; #ip地址
    }