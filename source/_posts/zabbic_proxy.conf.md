---
title: zabbix_agentd配置文件
tags: 
- 监控
---

```sh
#cat  zabbix_agentd.conf
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
ListenPort=10050
Timeout=30
ServerActive=101.129.224.225
Server=101.129.224.225
Hostname=YSZQ-GS-JS01

#cat  zabbix_proxy.conf
Server=10.110.122.9
ProxyMode=0
Hostname=GS_proxy
LogFile=/tmp/zabbix_proxy.log
PidFile=/usr/local/zabbix_proxy/zabbix_proxy.pid
DBHost=localhost
DBName=zabbix_proxy 
DBUser=zabbix
DBPassword=zabbix
```

