# LNMP动态网站架构
    由Linux+Nginx+MySQL(Mariadb)+PHP(世界上最好的语言)
    Linux不一定是Centos、RHEL,也可以是Debian、Ubuntu

## 软件安装的三种方式
    1、源码编译安装
    2、rpm包安装
    3、yum安装
    #、脚本安装 # shell脚本，就是将不同的指令放到同一个文件中

## LNMP安装
    1、源码安装
    2、LNMP一键安装[][ http://lnmp.org](https://lnmp.org)
        [root@test ~]# wget http://soft.vpser.net/lnmp/lnmp1.5.tar.gz # 下载安装脚本
        [root@test ~]# tar -zxvf lnmp1.5.tar.gz 
        [root@test ~]# cd lnmp1.5
        [root@test lnmp1.5]# bash install.sh # 执行安装脚本
    3、yum安装   
    [root@client ~]# yum install epel-release -y
    [root@client ~]# yum install mariadb mariadb-server php-mysql php-fpm php nginx -y
 
    [root@client ~]# systemctl start nginx
    [root@client ~]# systemctl start mariadb
    [root@client ~]# systemctl start php-fpm
    # 修改nginx服务的配置文件，使其支持php环境
    [root@client ~]# vim /etc/nginx/nginx.conf
    # 加入下面的参数
        #location / {
        #}
        location ~ \.php$ {
        root /usr/share/nginx/html;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        }

        #error_page 404 /404.html;
        #    location = /40x.html {
        # }
    [root@client ~]# systemctl restart nginx

    # 创建php的测试页面
    [root@client ~]# touch /usr/share/nginx/html/index.php #在网站的根目录下面创建php测试页面
    [root@client ~]# vim /usr/share/nginx/html/index.php
    <?php
            phpinfo()
    ?>

    # 使用curl、w3m访问搭建的网站


# Linux内核更新
[root@client ~]# uname -r #查看当前内核版本
## 载入公钥
    [root@test ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

## 安装ELRepo
    [root@test ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

## 查看以kernel开头的可用的rpm包
    [root@test ~]# yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*

## 安装最新版本的kernel
    [root@test ~]# yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-ml.x86_64