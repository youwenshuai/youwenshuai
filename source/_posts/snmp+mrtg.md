---
title: snmp+mrtg实现服务器监控图形展示
tags: 监控
---

1.snmp调试

2.修改mrtg配置文件

3.根据配置文件生成图片

4.生成首页文件

5.cron 轮休生成监控图片

```sh
#yum -y install snmp*

#cat /etc/snmp/snmpd.conf

rocommunity public

service snmpd start

chkconfig snmpd on

#yum -y install mrtg*

#cat /etc/mrtg/mrtg.cfg

HtmlDir: /var/www/mrtg

ImageDir: /var/www/mrtg

LogDir: /var/lib/mrtg

ThreshDir: /var/lib/mrtg

Target[eth3_lan]: /192.168.0.182:public@localhost

Options[eth3_lan]:growright

Directory[eth3_lan]:eth3

MaxBytes[eth3_lan]:100000000

Kmg[eth3_lan]:K,M,G

YLegend[eth3_lan]:Bytes per Second

ShortLegend[eth3_lan]:B/s

Legend1[eth3_lan]:meimiaoliuruliang

Legend2[eth3_lan]:meimiaoliuchuliang

LegendI[eth3_lan]:liuru

LegendO[eth3_lan]:liuchu

Title[eth3_lan]:eth3wangluoliul

PageTop[eth3_lan]:<h1>eth3wangluoliuliang</h1>

Target[cpuload]:.1.3.6.1.4.1.2021.11.50.0&.1.3.6.1.4.1.2021.11.53.0:public@localhost

Options[cpuload]:nopercent,growright

Directory[cpuload]:cpu

MaxBytes[cpuload]:100

Unscaled[cpuload]:dwym

YLegend[cpuload]:cpu Utilization

ShortLegend[cpuload]:%

Legend1[cpuload]:CPU yonghufuzai

Legend2[cpuload]:cpu xianzhi

LegendI[cpuload]:yonghu

LegendO[cpuload]:xianzhi

Title[cpuload]:fuzai+xianzhi

PageTop[cpuload]:<h1>fuzai+xianzhi</h1>

#cat /etc/cron.d/mrtg

LANG=C LC_ALL=C /usr/bin/mrtg /etc/mrtg/mrtg.cfg --lock-file /var/lock/mrtg/mrtg_l --confcache-file /var/lib/mrtg/mrtg.ok  执行三次直到没有报错
```

生成首页

```sh
indexmaker --output /var/www/mrtg/index.html --title="mrtg" /etc/mrtg/mrtg.cfg
```

apache启动测试