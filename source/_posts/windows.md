---
title: 添加zabbix监控windows机器
tags: 
- 监控
- 系统
---

修改配置文件 zabbix_agentd.win.conf，配置文件可放在任何目录下

LogFile=c:\zabbix_agentd.log

Server=10.110.122.9

ListenPort=10050

ServerActive=10.110.122.9

Hostname=WQ

\# Include=c:\zabbix\zabbix_agentd.userparams.conf

\# Include=c:\zabbix\zabbix_agentd.conf.d\

\# Include=c:\zabbix\zabbix_agentd.conf.d\*.conf

安装服务

zabbix_agentd.exe -c c:\Users\WQ\Desktop\conf\zabbix_agentd.win.conf -i

启动服务

zabbix_agentd.exe -c c:\Users\WQ\Desktop\conf\zabbix_agentd.win.conf -s

zabbix_agentd.exe --start

