---
title: mysql常用语法
tags: 
- 数据库
---

##### mysql表复制

mysql >create table t2 like t1;

mysql >insert into t2 select * from t1;

##### mysql表索引

create index index_t1 on t1(name);

show index from t1\G

drop index index_t1 on t1;

创建索引

alter table t1 add index in_name (name);

alter table t1 drop index in_name;

删除主键索引

alter table t1 modify id int unsigned not null;

alter table t1 drop primary key;

创建唯一索引

alter table t1 add unique(name);

alter table t1 add unique un_name(name);

删除唯一索引

alter table t1 drop index un_name;

alter table t1 drop index un_name;

创建主键索引

alter table t1 add primary key(id)；

alter table t1 modify id int unsigned not null auto_increment;

##### mysql视图

创建视图

create view v_t1 as select * from t1 where id>2 and id<5;

删除视图

drop vied v_t1;

##### mysql内置函数

字符串

concat(string,[]) //连接自串

lcase(string) //转换成小写

ucase(string) //转换成大写

length(string)//长度

ltrim(string)//左边去空

rtrim(string)//右去空

repeat(string,count) //重复count次

replace(str,search_str,replace_str) //在str中的replace_str替换成search_str

space(count)//生成空格

substring(str,pos,[length])//从pos开始取length 长度

数学函数

bin(decimal_number)//十进制转二进制

ceiling(num) 向上取整

floor(num) 向下取整

max(col) //聚合取大

min(col)//聚合取小

sqrt(num)//开平方

日期函数

curdata()//目前日期

crutime()//目前时间

now()//目前时间日期

unix_timestamp(date)//返回当前date的unix时间戳

from_unixtime()//返回unix时间戳的日期

week(date)//返回第几周

year("1992-02-02")//返回年

datediff(expr,expr2)//返回起始时间差天数

rand()//返回0-1的随机数

##### mysql预处理语句

? prepare

prepare stmt1 from 'select from t1 where id>?';

set @i=5;

select @i;

execute stmt1 using @i //使用

drop prepare stmt1 //删除

##### mysql事务处理

myISAM引擎不支持  innodb支持

数据回滚

set autocommit=0 关闭自动提交功能

delete from t1 where id=2;

savepoint p1;

delete from t1 where id=3;

savepoint p2;

rollback to p1;还原到p1还原点

rollback;最初还原点

##### mysql存储

? procedure;//类似函数

\d // 修改定界符

create procedure p2()

begin

set @i=3;

where @i<=100 do

insert into t2(name) values(concat("user",@i));

set @i=@i+1；

end while;

end //

\d ;  改回定界符

show procedure status\G;

call p2; 调用存储

select * from t2; 查看 

##### mysql触发器

? trigger;

\#制造一个同时插入 t1 t2

\d //

create trigger t1 before insert on t1 for each row

begin

insert into t2(id) values(new.id);

end//

\d ; 改回定界符

show triggers; 查看

insert into t1(id) values(1);

\#删除

\d //

create trigger t2 before delete on t1 for each row

begin

delete from t2 where id=old.id;

end//

\d ;

\#修改

\d //

create trigger t3 before uptate on t1 for each row

begin update t2 set id=new.id where id=old.id;

end//

\d ;

##### 重排auto_increment值

alter table tablename auto_increment=1;

##### 分组聚合

group by  与with rollup一起使用

select cname,pname,count(cname) from t1 group by cname,pname with rollup;

##### sql语句优化

show status like "com%";//增删改查相关记录

show status like "innodb_rows%"//影响行数

show status like "connections"//连接mysql次数（无论成功与否）

show status like "uptime"//启动时间

show status like "slow_qureies";//慢查询次数

show variables like "%slow_query%";//查询相关配置开启与否 查询慢查询是否开启

看慢查询日志

索引优化

check table sales;//检查表错误

optimize table sales;//优化表空间

数据导入导出 优化

mysql>select name from t1 into outfile  "/tmp/test.txt";

mysql>load data infile "/tmp/test.txt" into table t1(name);

myisam引擎存储的表

mysql>alter table t1 disable keys;

mysql>load data infile "/tmp/test.txt" into table t1(name);

mysql>alter table t1 enable keys;

确保数据没有重复可以关闭唯一索引 可以提高导入效率//慎用

set unique_checks=0  //关闭

set unique_checks=1   //开启

innodb导入优化

按照逐渐顺序保存的 可以将导入的数据主键的顺序排列，可以有效提高导入速率

set autocommit=0;关闭自动提交可以提高速率

insert 优化

数据库的连接关闭等

group by优化

避免嵌套查询

数据库优化

优化表的类型

通过拆分提高表的访问率 分区 分表 主从数据库

使用中间表提高统计查询速度  --视图

mysql服务器优化

myisam读锁定（都不能写，可以读）

lock table t1 read;

myisam写锁定(谁也写不了读不了，除了本人)

lock table t1 write;

解锁 unlock tables;

四种字符集

Server characterset utf8

db characterset utf8

client  characterset utf8

conn  characterset utf8

如何设置

/etc/my.cnf

default-character-set=utf8

[mysqld]

character-set-server = utf8

collation-server = utf8_general_ci //校验字符集

binary log 日志

cat /etc/my.cnf

log-bin=mysql-bin

slow log 日志

cat /etc/my.cnf

log_slow_queries=slow.log

long_query_time=5

socket问题

没有socket 可尝试用tcp连接

mysql -uroot -proot --protocol tcp -hlocalhost 

root密码丢失无法登陆

mysqld_safe --skip-grant-tables --user=mysql &