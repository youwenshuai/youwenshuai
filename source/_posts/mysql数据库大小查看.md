---
title: mysql数据库大小查看
tags: 
- 数据库
---

[用SQL命令查看Mysql数据库大小](https://www.cnblogs.com/yuwensong/p/7729267.html)

要想知道每个数据库的大小的话，步骤如下：

1、进入information_schema 数据库（存放了其他的数据库的信息）

use information_schema;

 

2、查询所有数据的大小：

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;

 

3、查看指定数据库的大小：

比如查看数据库home的大小

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';

 

4、查看指定数据库的某个表的大小

比如查看数据库home中 members 表的大小

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';