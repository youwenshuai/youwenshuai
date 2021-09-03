---
title: vm网络连接设置
tags: 网络
---

1、自动获取IP地址

虚拟机使用桥接模式，相当于连接到物理机的网络里，物理机网络有DHCP服务器自动分配IP地址。

\#dhclient 自动获取ip地址命令

\#ifconfig 查询系统里网卡信息，ip地址、MAC地址

 

分配到ip地址后，用物理机进行ping ip地址，检测是否ping通。

2、手动设置ip地址

如果虚拟机不能自动获取IP，只能手动配置，配置方法如下：

输入命令

\#vi /etc/sysconfig/network-scripts/ifcfg-eth0 [编辑网卡的配置文件]

输入上述命令后回车，打开配置文件，使用方向键移动光标到最后一行，按字母键“O”，进入编辑模式，输入以下内容：

IPADDR=192.168.4.10

NETMASK=255.255.255.0

GATEWAY=192.168.4.1

另外光标移动到”ONBOOT=no”这一行，更改为ONBOOT=yes

“BOOTPROTO=dhcp”，更改为BOOTPROTO=none

完成后，按一下键盘左上角ESC键，输入:wq 在屏幕的左下方可以看到，输入回车保存配置文件。

 

之后需要重启一下网络服务，命令为

\#servicenetwork restart

网络重启后，eth0的ip就生效了，使用命令#ifconfigeth0 查看

接下来检测配置的IP是否可以ping通，在物理机使用快捷键WINDOWS+R 打开运行框，输入命令cmd，输入ping 192.168.4.10 进行检测，ping通说明IP配置正确。

 

备注：我所在的物理机网段为192.168.4.0 网段。大家做实验的时候根据自己的环境进行设

定，保持虚拟机和物理机在同一网段即可。

网卡起不来 

查看  /etc/udev/rules.d/70-persistent-net.rules， eth网卡的MAC 是否一致 重新启动服务