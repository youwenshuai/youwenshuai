---
title: DHCP服务器搭建
tags:
- 网络
- 系统
---

##### 修改dhcp的主配置文件

```sh
#vim /etc/dhcp/dhcpd.conf
ddns-update-style none;
subnet 192.168.2.0 netmask 255.255.255.0 {
	range 192.168.2.100 192.168.2.200;
	option domain-name-servers 192.168.2.1;
	option domain-name "server.com";
	option routers 192.168.2.250;
	option broadcast-address 192.168.2.255;
	default-lease-time 600;
	max-lease-time 7200;
	}
```

##### 	修改网卡文件

```shell
#vim /etc/sysconfig/dhcpd
DHCPDARGS=eth3
```

