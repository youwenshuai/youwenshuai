---
title: redis部署
tags: 
- 数据库
---

redis 4.0.8 cluster 安装 需要ruby2.2.2+

1.wget http://download.redis.io/releases/redis-4.0.10.tar.gz

2.tar xf redis-4.0.8.tar.gz -C /opt/redis/ --strip-components=1

3.yum -y install gcc make tcl

4.cd /opt/redis/src && make MALLOC=libc （默认值为libc）&& make 

5.安装ruby 2.2.2+ && gem install redis

6.cd .. && mkdir -p cluster/{6379..6384}

7.cp redis.conf redis.conf.bak

8.vim redis.conf

​	bind 192.168.203.13

​	protected-mode yes

​	port 6379

​	tcp-backlog 511

​	timeout 0

​	tcp-keepalive 300

​	daemonize yes	//后台运行

​	supervised no

​	pidfile /opt/redis/cluster/6379/redis_6379.pid

​	loglevel notice

​	logfile "/opt/redis/cluster/6379/redis_6379.log"

​	databases 16

​	always-show-logo yes

​	save 900 1

​	save 300 10

​	save 60 10000

​	stop-writes-on-bgsave-error yes

​	rdbcompression yes

​	rdbchecksum yes

​	dbfilename dump.rdb

​	dir /opt/redis/cluster/6379/

​	# slaveof 192.168.203.13 6379 //cluster不允许

​	slave-serve-stale-data yes

​	slave-read-only yes

​	repl-diskless-sync no

​	repl-diskless-sync-delay 5

​	repl-disable-tcp-nodelay no

​	slave-priority 100

​	maxmemory 5368709120 //最大使用5G内存，给系统预留3G内存

​	maxmemory-policy volatile-lru	//只对设置了过期时间的key进行LRU（默认值）

​	maxmemory-samples 5	//default

​	lazyfree-lazy-eviction no

​	lazyfree-lazy-expire no

​	lazyfree-lazy-server-del no

​	slave-lazy-flush no

​	appendonly yes	//提供更好的数据持久性保障

​	appendfilename "appendonly.aof"

​	appendfsync everysec	//everysec：每秒钟同步一次，折中的方案。

​	no-appendfsync-on-rewrite no

​	auto-aof-rewrite-percentage 100

​	auto-aof-rewrite-min-size 64mb

​	aof-load-truncated yes

​	aof-use-rdb-preamble no

​	lua-time-limit 5000

​	cluster-enabled yes

​	cluster-config-file nodes-6379.conf	//cluster 自主管理的配置文件

​	cluster-node-timeout 15000

​	cluster-slave-validity-factor 10

​	slowlog-log-slower-than 10000

​	slowlog-max-len 128

​	latency-monitor-threshold 0

​	notify-keyspace-events ""

​	hash-max-ziplist-entries 512

​	hash-max-ziplist-value 64

​	list-max-ziplist-size -2

​	list-compress-depth 0

​	set-max-intset-entries 512

​	zset-max-ziplist-entries 128

​	zset-max-ziplist-value 64

​	hll-sparse-max-bytes 3000

​	activerehashing yes

​	client-output-buffer-limit normal 0 0 0

​	client-output-buffer-limit slave 256mb 64mb 60

​	client-output-buffer-limit pubsub 32mb 8mb 60

​	hz 10

​	aof-rewrite-incremental-fsync yes

9.for i in {6379..6384}; do cp redis.conf cluster/$i/$i.conf; done

10.for i in {6379..6384}; do sed -i "s/6379/$i/g" cluster/$i/$i.conf; done	//放置配置文件

11.vim /etc/security/limits.conf //追加如下行（设置ulimit）

​	*               soft    noproc  10240

​	*               hard    noproc  10240

​	*               soft    nofile  10240

​	*               hard    nofile  10240

12.for i in {6379..6384}; do sed -i /opt/redis/src/redis-server /opt/redis/cluster/$i/$i.conf; done //启动redis

13./opt/redis/src/redis-trib.rb create --replicas 1 192.168.203.13:6379 192.168.203.13:6380 192.168.203.13:6381 192.168.203.13:6382 192.168.203.13:6383 192.168.203.13:6384  //创建集群

14.echo for i in {6379..6384}; do /opt/redis/src/redis-server /opt/redis/cluster/$i/$i.conf; done >> /etc/rc.local //开机自启

参考文档：

​	https://redis.io/topics/cluster-tutorial

​	https://blog.csdn.net/ljl890705/article/details/51039015

/*	初始化redis cluter */

​	for i in {6379..6384}; do /opt/redis/src/redis-cli -h 192.168.205.55 -p $i shutdown; done

​	for i in {6379..6384}; do rm -rf /opt/redis/cluster/*; done

​	mkdir -p /opt/redis/cluster/{6379..6384}

​	for i in {6379..6384}; do cp /opt/redis/redis.conf /opt/redis/cluster/$i/$i.conf; done

​	for i in {6379..6384}; do sed -i "s/6379/$i/g" /opt/redis/cluster/$i/$i.conf; done

/ * 启动redis cluster */

for i in {6379..6384}; do /opt/redis/src/redis-server /opt/redis/cluster/$i/$i.conf; done && \

/opt/redis/src/redis-trib.rb create --replicas 1 192.168.205.55:6379 192.168.205.55:6380 192.168.205.55:6381 192.168.205.55:6382 192.168.205.55:6383 192.168.205.55:6384

// 输入yes 默认前三个为主，后三个为从