---
title: zabbix自定义键值
tags: 
- 监控
---

#### **1.redis 监控**

实施：

```sh
mkdir /opt/zabbix_agentd/scripts/
touch  /opt/zabbix_agentd/scripts/data_dir.sh
chmod +x /opt/zabbix_agentd/scripts/data_dir.sh
vi /opt/zabbix_agentd/etc/zabbix_agentd.conf
UserParameter=redis.data,/opt/zabbix_agentd/scripts/data_dir.sh
vi /opt/zabbix_agentd/scripts/data_dir.sh
#!/bin/bash
data_size=`du -s /opt/redis/data`
echo $(echo $data_size |awk '{print $1}')"000"
pkill -9 zabbix
/opt/zabbix_agentd/sbin/zabbix_agentd -c /opt/zabbix_agentd/etc/zabbix_agentd.conf
1.get_redis_clients.sh
#!/bin/bash
client_num=`echo 'info '|redis-cli -a cctv.com |grep 'connected_clients' |awk -F':' '{print $2}'`
echo "$client_num"
2.get_redis_copy_aof.sh
#!/bin/bash
replication_status=("`echo 'info '|redis-cli -a cctv.com |grep 'aof_enabled' |awk -F':' '{print $2}'`")
echo $replication_status
3.get_redis_data_mem.sh
#!/bin/bash
echo 'info '|redis-cli -a cctv.com|grep -w "used_memory_rss" |awk -F':' '{print $2}'
4.get_redis_hisrory_max_mem.sh
#!/bin/bash
echo 'info '|redis-cli -a cctv.com|grep -w 'used_memory_peak' |awk -F':' '{print $2}'
5.get_redis_mem_rate.sh
#!/bin/bash
echo 'info '|redis-cli -a cctv.com|grep mem_fragmentation_ratio |awk -F':' '{print $2}'
6.get_redis_mem_total.sh
#!/bin/bash
echo 'info '|redis-cli -a cctv.com|grep -w 'used_memory_rss'|awk -F':' '{print $2}'|awk -F' ' '{print $1}'
7.get_redis_status.sh
#!/bin/bash
redis_status=`/bin/echo 'ping'|redis-cli -a cctv.com`
if [ $redis_status == PONG ];then
        echo "1"
else 
        echo "0"
fi
8.get_redis_version.sh
#!/bin/bash
redis_status=`/bin/echo 'ping'|redis-cli -a cctv.com`
if [ $redis_status == PONG ];then
        echo "1"
else 
        echo "0"
fi
9.get_set_redis_mem_total.sh
#!/bin/bash
total_mem=`cat /opt/redis/etc/redis.conf |grep 'maxmemory' |grep -v '^#' |awk -F' ' '{print $2}'`
echo "$total_mem"
```



#### **2.mysql 监控**

实施：

```sh
yum -y install php php-mysql
rpm -ivh percona-zabbix-templates-1.1.8-1.noarch.rpm
cp /var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf /opt/zabbix_agentd/etc/zabbix_agentd.conf.d
cat zabbix_agentd.conf
Include=/opt/zabbix_agentd/etc/zabbix_agentd.conf.d
cd /var/lib/zabbix/percona/scripts/
cat ss_get_mysql_stats.php  #数据库连接
$mysql_user = 'root';
$mysql_pass = 'password';
cat get_mysql_stats_wrapper.sh 
HOST=101.129.224.207
RES=`/opt/mysql/bin/mysql -e 'SHOW SLAVE STATUS\G' | egrep '(Slave_IO_Running|Slave_SQL_Running):' | awk -F: '{print $2}' | tr '\n' ','`
pkill -9 zabbix
/opt/zabbix_agentd/sbin/zabbix_agentd -c /opt/zabbix_agentd/etc/zabbix_agentd.conf
```



#### **3.keepalived 状态监控**

```sh
mkdir /opt/zabbix_agentd/script
cat chk_keepalived_status.sh 
#!/bin/bash
keep_status=$(echo "`/etc/init.d/keepalived status`" | awk  '{print $5}')
result=0
if [[ `echo $LANG` == 'en_US.UTF-8' ]];then
        if [[ $keep_status == 'running...' ]];then
                result=1
                echo $result
        else
                result=0
                echo $result
        fi
else
        if [[ `/etc/init.d/keepalived status|awk '{print $4}'` == '正在运行...' ]];then
                result=1
                echo $result
        else
                result=0
                echo $result
        fi
pkill -9 zabbix
/opt/zabbix_agentd/sbin/zabbix_agentd -c /opt/zabbix_agentd/etc/zabbix_agentd.conf
```

#### **4.LVS状态监控**

```sh
1.get_keepalived_status.sh
#!/bin/bash
num=`/bin/ps -ef |grep keepalived |grep -v "grep" |wc -l`
if [ "$num" == 5 ];then
        echo "1"
else 
        echo "0"
fi
2.get_lvs_bit_sec.sh
#!/bin/bash
tail -1 /proc/net/ip_vs_stats | /usr/bin/awk '{print strtonum("0x"$1),strtonum("0x"$2), strtonum("0x"$3),strtonum("0x"$4), strtonum("0x"$5)}'|awk '{print $4}'
3.get_lvs_conns_sec.sh
#!/bin/bash
cat /proc/net/ip_vs_conn |wc -l
4.get_lvs_keepalived_vip_status.sh
#!/bin/bash
KEEPALIVE_CONF_DIR=/etc/keepalived/keepalived.conf
IP_LIST=("`cat $KEEPALIVE_CONF_DIR |grep virtual_server |awk -F' ' '{print $2}'|sort |uniq`")
n=0
for ip in ${IP_LIST[@]};do
        if [ '`/sbin/ip addr |grep '$ip'|wc -l`' == 1 ];then
                let n=n+1
        else
                n=0 
        fi
done
if [ `cat $KEEPALIVE_CONF_DIR |grep virtual_server |awk -F' ' '{print $2}'|sort |uniq |wc -l` == "$n" ];then
        echo "0"
else
        echo "1"
fi
5.get_lvs_packets_sec.sh
#!/bin/bash
tail -1 /proc/net/ip_vs_stats | /usr/bin/awk '{print strtonum("0x"$1),strtonum("0x"$2), strtonum("0x"$3), strtonum("0x"$4),strtonum("0x"$5)}'|awk '{print $2}'
```

