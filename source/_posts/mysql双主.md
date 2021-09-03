---
title: mysql双主搭建
tags: 
- 数据库
---

-----------------------------------------------mysql双主

**共同：**

cat /etc/my.cnf

[mysqld]

\#skip-grant-tables

\# DO NOT MODIFY, Universe will generate this part

port = 3306

server_id = 2

basedir = /opt/mysql

datadir = /opt/mysqldata

log_bin = /opt/mysqldata/mysql-bin

\# BINLOG

binlog_error_action = ABORT_SERVER

binlog_format = row

binlog_rows_query_log_events = 1

log_slave_updates = 1

master_info_repository = TABLE

max_binlog_size = 1024M

relay_log = mysql-relay-bin

relay_log_info_repository = TABLE

relay_log_recovery = 1

sync_binlog = 1

sync_relay_log = 1

\# GTID #

gtid_mode = ON

enforce_gtid_consistency = 1

\# ENGINE

default_storage_engine = InnoDB

innodb_buffer_pool_size = 2048M

innodb_data_file_path = ibdata1:1G:autoextend

innodb_file_per_table = 1

innodb_flush_log_at_trx_commit=1

innodb_flush_method = O_DIRECT

innodb_io_capacity = 200

innodb_log_buffer_size = 64M

innodb_log_file_size = 1024M

innodb_log_files_in_group = 2

innodb_max_dirty_pages_pct = 60

innodb_print_all_deadlocks=1

innodb_stats_on_metadata = 0

innodb_strict_mode = 1

\# CACHE

key_buffer_size = 32M

tmp_table_size = 32M

max_heap_table_size = 32M

table_open_cache = 1024

query_cache_type = 0

query_cache_size = 0

max_connections = 1000

thread_cache_size = 1024

open_files_limit = 65535

\# SLOW LOG

slow_query_log = 1

slow_query_log_file = mysql-slow.log

log_slow_admin_statements = 1

log_slow_slave_statements = 1

long_query_time  = 1

\#Rplication

binlog-do-db=portal_test

binlog-do-db=portal_cctv

binlog-ignore-db=mysql

binlog-ignore-db=test

binlog-ignore-db=information-schema

\# SEMISYNC #

plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"

rpl_semi_sync_master_enabled = 1

rpl_semi_sync_slave_enabled = 0

rpl_semi_sync_master_wait_for_slave_count = 1

rpl_semi_sync_master_wait_no_slave = 0

rpl_semi_sync_master_timeout = 300000 # 5 minutes

\# CLIENT_DEPRECATE_EOF

session_track_schema = 1

session_track_state_change = 1

session_track_system_variables = '*'

\# MISC

character_set_server = utf8

log_timestamps=SYSTEM

lower_case_table_names = 1

max_allowed_packet = 64M

read_only = 0

skip_external_locking=1

skip_name_resolve = 1

skip_slave_start = 1

\#socket = /opt/mysqldata/mysqld.sock

\#pid_file = /opt/mysqldata/mysqld.pid

cp support-files/mysql.server /etc/init.d/mysqld 

./bin/mysqld --defaults-file=/etc/my.cnf --user=mysql --datadir=/mydata/data --basedir=/opt/mysql --initialize --lc-messages_dir=/opt/mysql/share

**node5:**

grant replication slave on *.* to 'node5'@'192.168.1.%' identified by 'node5'; 

**node6:**

stop slave;

change master to master_host='192.168.1.5',master_user='node5',master_password='node5',master_auto_position=1;

start slave;

**node6:**

grant replication slave on *.* to 'node6'@'192.168.1.%' identified by 'node6'; 

**node5:**

stop slave;

change master to master_host='192.168.1.6',master_user='node6',master_password='node6',master_auto_position=1;

start slave;

-------------------------------------------------------------keepalived

\#tar xf keepalived-1.4.4.tar.gz

\#yum -y install openssl-devel  openssl

\#./configure --prefix=/opt/keepalived

\#make && make install

\#cp sbin/keepalived /etc/init.d/

\#cp etc/sysconfig/keepalived /etc/sysconfig/

\#mkdir /etc/keepalived

\#cp etc/keepalived/keepalived.conf /etc/keepalived/

\#cp sbin/keepalived /usr/sbin/

\#echo "/etc/init.d/keepalived " >>/etc/rc.local 

\#cat /etc/keepalived/keepalived.conf

! Configuration file for keepalived

vrrp_script chk_mysql {

   script "/etc/keepalived/check_mysql.sh"

   interval 5

   fall 3

   rise 3

   }

vrrp_instance V_mysql_1 {

​    state BACKUP

​    interface  eno16777736

​    virtual_router_id 4

​    priority 80

​    advert_int 1

​    nopreempt

​    authentication {

​       auth_type PASS

​       auth_pass 1111

​    }

​    track_script {

​    chk_mysql

​    }

​    virtual_ipaddress {

​      192.168.1.100

​    }

}

\#cat check_mysql.sh 

counter=$(ss -na|grep "LISTEN"|grep "3306"|wc -l)

if [ "${counter}" -eq 0 ]; then

​        pkill -9 keepalived

fi

**!!!注意  virtual_router_id 一样 否则可能会造成抢占VIP**