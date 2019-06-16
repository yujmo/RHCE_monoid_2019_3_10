# RHEL7 修改启动/运行级别
multi-user.target # 进入到命令行模式
graphical.tar  # 进入到图形化界面

[root@test ~]# systemctl isolate multi-user.target  # 临时修改
[root@test ~]# systemctl isolate graphical.target 
[root@test ~]# systemctl isolate runlevel5.target 


[root@test ~]# systemctl get-default  # 查看当前的运行级别
graphical.target


## 修改默认的运行级别
/etc/systemd/system/default.target
[root@test ~]# systemctl set-default multi-user.target

[root@test ~]# rm -rf /etc/systemd/system/default.target
[root@test ~]# ln -sf /lib/systemd/system/runlevel3.target /etc/systemd/system/default.target
