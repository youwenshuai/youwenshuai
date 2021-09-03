---
title: redis 单机持久化 
tags: 
- 数据库
---

1.安装

yum -y install gcc make tcl //安装依赖

wget http://download.redis.io/releases/redis-4.0.X.tar.gz

tar xf redis-4.0.8.tar.gz -C /opt/redis/ --strip-components=1

cd /opt/redis/src && make MALLOC=libc

2.优化系统参数

vim /etc/sysctl.d/redis.conf

​	vm.overcommit_memory = 1

​	net.core.somaxconn = 1024

sysctl --system

echo never > /sys/kernel/mm/transparent_hugepage/enabled

vim /etc/rc.local

​	echo never > /sys/kernel/mm/transparent_hugepage/enabled

​	/opt/redis/src/redis-server /opt/redis/redis.conf  //开机自启

chmod +x /etc/rc.d/rc.local

3.修改配置文件

cd /opt/redis

mv redis.conf redis.conf.bak

vim redis.conf

​	bind 192.168.203.13	//修改为本机地址（ip add 查看）

​	protected-mode yes

​	port 6379

​	tcp-backlog 511

​	timeout 0

​	tcp-keepalive 300

​	daemonize yes	//后台运行

​	supervised no

​	pidfile /opt/redis/redis_6379.pid

​	loglevel notice

​	logfile "/opt/redis/cluster/redis_6379.log"

​	databases 16

​	always-show-logo yes

​	save 900 1

​	save 300 10

​	save 60 10000

​	stop-writes-on-bgsave-error yes

​	rdbcompression yes

​	rdbchecksum yes

​	dbfilename dump.rdb

​	dir /opt/redis/

​	# slaveof 192.168.203.13 6379 // slave 设置为 master的地址及端口

​	slave-serve-stale-data yes

​	slave-read-only yes

​	repl-diskless-sync no

​	repl-diskless-sync-delay 5

​	repl-disable-tcp-nodelay no

​	slave-priority 100

​	maxmemory 5368709120 //最大使用5G内存，给系统预留3G内存,根据系统内存设定，一般设置为系统内存的50%

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

​	cluster-enabled no

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

/opt/redis/src/redis-server /opt/redis/redis.conf	//启动redis