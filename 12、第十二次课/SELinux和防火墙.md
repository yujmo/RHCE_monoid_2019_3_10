## SELinux
### 一、简述
#### 1、SELinux
美国NSA（贡献）&Fedora（kernel2.6）社区共同发布
security-enhanced linux。Selinux默认在红帽的系统中都是开启的，在debain类的系统中，也有类似的MAC方案，叫apparmor

注：能开启selinux尽量开启

#### 2、DAC
自主访问控制，通过软件或者守护进程（daemons）用什么uid执行，对文件是否拥有rwx权限等来进行判断
#### 3、MAC
强制访问控制，通过kernel直接来针对不同的进程和文件以及接口等之间实现访问控制，和DAC是需要并行使用的

### 二、概念
#### 1、主体（subject）
主体大部分就是表示进程
#### 2、对象（object）
也就是客体，是主体尝试或者希望访问的一个目标，这个目标可以是一个文件，或者一个端口，甚至一个设备等可以被操作的
#### 3、策略
selinux安全服务器（一个内置于kerenel中的功能），拥有策略数据，设置不同的规则，是用户可以设置的工作模式
* targeted
是红帽的标准策略，只针对网络进行限制
* minimum  
是targeted的精简版本，只针对特定的进程，一般不使用这种策略
* mls
多级安全防护，是最严格的策略控制，是nsa默认使用的环境，每一个文件和端口设备都被强制要求进行策略过滤，要求非常高，通常不使用。

#### 4、工作模式
/etc/selinux/config  这个文件里面记录了当前的selinux工作模式和安全策略等级
selinux有三个工作模式：
* enforcing   强制模式，是rhel的默认模式，会强制执行selinux策略
* permissive  
警告模式，selinux策略不强制执行，会给出用户警告信息，但不阻止访问
* disabled  
禁用selinux，一旦设置为这个模式就是关闭了selinux，而且必须重启系统才能生效
```[root@openstack ~]# getenforce     //查看当前模式
 [root@openstack ~]# setenforce 0     //修改当前模式为警告模式，临时有效
```

#### 5、安全上下文
主体是否可以访问对象，取决于上下文，上下文又分为文件上下文和进程上下文。selinux会为进程和文件添加安全信息标签。
``` ls -Z  查看文件上下文 
unconfined_u:object_r:admin_home_t:s0
```
安全上下文通常用:作为分隔符，由4个字段组成
身份字段：角色：类型：使用的级别
* 身份字段
root        表示root身份
system_u    表示系统用户身份
user_u      表示一般用户身份
unconfined_u   未知用户身份
```
[root@openstack ~]# yum install setools-console-3.3.8-4.el7.x86_64    //安装setools软件包，里面有seinfo等工具
[root@openstack ~]# seinfo -u   //查看系统默认支持的用户
```
* 角色
主要是用来表示这个数据是进程还是文件还是目录，这个字段通常不需要修改，只是查看
```
[root@openstack ~]# seinfo -r    //查看系统默认的角色
```
object_r     这个角色代表了查看的上下文类型是文件或者目录，r就是role的意思
system_r 这个角色代表了上下文的类型是进程
* 类型
是上下文中最重要的字段，进程是否可以访问一个文件、端口、设备都由这个字段决定，必须匹配这个字段才可以访问
```
[root@openstack ~]# seinfo  -t | more   //查看类型
```
* 使用的级别灵敏度
是s0/s1/s2等字段，数值越大，灵敏度越高，通常不需要修改
#### 6、布尔值开关
布尔值是selinux中策略的开关

### 三、SELinux操作
#### 1、基本操作
```
setenforce   //设置
getenforce   //查看
ls -Z        //查看文件
ps -auxZ     //查看进程 
sestatus     //查看selinux当前的状态统计信息
getsebool -a //查看所有的布尔值
sestatus -b  //等同于getsebool
```
#### 2、实例&排故
```
[root@openstack ~]# vim /etc/ssh/sshd_config    //修改sshd 的端口号，然后尝试重启服务，selinux生效的情况下会阻止这种操作
[root@openstack ~]# cat /var/log/audit/audit.log  | grep ssh |grep AVC    //通过查看audit报告，我们可以获知sshd启动被selinux阻止的原因
[root@openstack ~]# semanage port -l | grep 22     //查看22端口对应的安全上下文信息
[root@openstack ~]# semanage port -a -t ssh_port_t -p tcp 2222    //添加新的端口到ssh的端口域中，只要上下文一致，就可以访问
[root@openstack ~]#semanage port -l | grep ssh

[root@openstack ~]# chcon -R --reference=/var/www/html /tmp/xx   使用chcon命令引用上下文，http服务中的文件也需要保证上下文的一致   

[root@openstack ~]# setsebool -P ftpd_anon_write off     //永久设置布尔值
[root@openstack ~]# sestatus -b |grep http  //查看布尔值
```
yum install policycoreutils\*    管理工具
yum install setroubleshoot   排故工具

## samba服务器
跨平台的文件共享服务器，还支持加入windows AD域和自身实现DC功能
#### 1、安装
[root@openstack ~]# yum install samba samba-client -y
#### 2、配置防火墙
[root@openstack ~]# firewall-cmd --add-service=samba
success
[root@openstack ~]# firewall-cmd --add-service=samba-client 
success
[root@openstack ~]# firewall-cmd --add-service=samba-client --permanent 
success
[root@openstack ~]# firewall-cmd --add-service=samba --permanent 
#### 3、selinux
[root@openstack ~]# mkdir /share    创建共享目录，默认情况下，selinux会阻止用户的这种操作

[root@openstack ~]# semanage fcontext -a -t user_home_dir_t '/share(/.*)?'   //用semanage fcontext比chcon更好，它不会再系统重打标签时失效.(/.*)?是用正则表示递归，要求列出的目录中的所有标签都和前面定义的一致
[root@openstack ~]# restorecon -vR /share/    //需要通过restorrecon手动刷新让标签上下文生效
[root@openstack ~]# setsebool -P samba_enable_home_dirs on   //启用samba支持家目录定义，默认是off，需要开启

#### 4、配置文件解读
[root@openstack samba]# mv smb.conf smb.conf.bak  
[root@openstack samba]# cp smb.conf.example smb.conf
```
workgroup = WORKGROUP     //工作组一致才能访问
; server string = Samba Server Version %v    //关闭版本号
max log size = 50   日志大小，0表示不限制
  
config file = /etc/samba/smb.conf.%U    //这个表示额外定义针对用户的配置文件，%U表示用户，
  
security = user       //设置用户访问samba的验证方式，有4种方式，默认是user，要求提供用户名和密码访问samba，如果设置为share，就是匿名访问，不要这样操作。第三种方式是server，通过服务器来验证账号和密码，第四种是domain验证，使用主控域服务器（PDC）来完成认证，必须涉及域的使用
passdb backend = tdbsam   //如果是user的认证方式，那么密码保存在哪里，默认是tdbsam，这个通常是保存在本地的一个密码数据文件，用指令smbpasswd或者pdbedit来实现
%v  表示版本
%m  表示主机名
%U  表示用户

[movie]     //用户看到的共享名
        comment = movie share     //注释信息
        browseable = yes          //可以看到共享目录
        writeable = yes           //可以写入
        path = /share             //共享路径
		write list = tom          //允许写入的用户,多个用户用逗号隔开
```
[root@openstack samba]# useradd -s /sbin/nologin tom  //创建一个用于登录samba的系统用户
[root@openstack samba]# smbpasswd -a tom   //将系统用户tom添加到samba用户中

[root@openstack samba]# useradd -s /sbin/nologin jarry
[root@openstack samba]# smbpasswd -a jarry
[root@openstack samba]# systemctl restart smb nmb
[root@openstack samba]# systemctl enable smb nmb

PS C:\Users\Administrator> net use * /del   删除win中已经建立的网络链接

samba故障的排除，首先是文件默认的权限rwx是否具备，然后是防火墙，再接着是selinux

[root@openstack samba]# testparm -v   //对samba的配置进行测试