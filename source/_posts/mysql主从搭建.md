---
title: mysql主从搭建
tags: 
- 数据库
---

环境搭好

182  183

182用户授权

mysql>grant all on *.* to user@192.168.0.183 identified by 'user'

查看bin-log

/mnt/sdb/mysql/bin/mysqlbinlog mysql-bin.000001 |more;

flush logs;  刷新日志

show master  status; 查看最后一条

reset master; 清空

修改my.cnf

log-bin=mysql-bin

server-id=1

读锁定

flush tables with read lock

解锁

unlock tables;

备份数据

mysqldump -uroot -proot test -l -F > test.sql -F 即 flush logs;

恢复

mysql -uroot -proot test -v -f <test.sql -v 详细信息 -f skip错误

定点恢复

mysqlbinlog --no-defaults --start-postion="222" --stop-postion="333" mysql-bin.000004|mysql-uroot-proot test

在主服务器上建立账户并授权 slave;

mysql]# mysql -uroot -p*******

grant replication slave on *.* to 'slave'@'10.23.122.19%' identified by 'q123456'; 

6.登录主服务器的mysql,查询master的状态

mysql>show master status\G PS:这里的\G后面不加；要不会报错 

7.配置从服务器Slave；

mysql> stop slave; (PS:如果不执行这句话，下面会报错的）

mysql> Change master to master_host='10.23.124.68',master_user='slave',master_password='q123456',

-> master_log_file='mysql-bin.000001',

-> master_log_pos=120;

(PS:这里master_log_file的值是6 查询出来的值，这里master_log_pos的值是6 查询出来的值，)

mysql> start slave;// 启动二进制复制

\8. 查看从服务器的状态 Slave ；

mysql> show slave status\G 

mysql 分区技术

range 分区

create table employees (id int not null,fname varchar(30),lname varchar(30),hired date not null default '1970-01-01',separated date not null default '9999-12-31',job_code int not null,store_id int not null ) partition by range (store_id)( partition p0 values less than(6), partition p1 values less than(11), partition p2 values less than(16), partition p3 values less than(21) );

list分区

partition by list(store_id)(

partition pN values in (3,5,7,9),

....

);

hast分区 

主要用于测试 确保数据在预先确定的数目的分区的平均分布

create table t2(id int) engine=myisam partition by hash(id) partitions 5;

key分区

其中innodb 

独立表空间和共享表空间

设置成独立的表空间才能创建innodb表引擎的表分区