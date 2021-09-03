---
title: DNS服务器搭建
tags: 
- Linux 
---

##### dns服务端

```sh
#yum -y install bind 
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { any; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any; };
        recursion yes;
        dnssec-enable yes;
        dnssec-validation no;
        dnssec-lookaside auto;
        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "wode.com" IN {
        type master;
        file "wode";

};

zone "0.168.192.in-addr.arpa"{
        type master;
        file "fan";

};

2cd /var/named
cp named.localhost wode 
cp named.localhost fan
chgrp named wode
chgrp named fan
vim wode 
$TTL 86400

@       IN SOA  des.wode.com. root.wode.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN      NS dns.wode.com.
@       IN      MX 5 mail.wode.com.
dns     IN      A  192.168.0.150
mail    IN      A  192.168.0.150
www     IN      A  192.168.0.150
ftp     IN      CNAME    WWW
~
vim fan

$TTL 86400
@       IN SOA dns.wode.com. root.wode.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN      NS dns.wode.com.
@       IN      MX 5 mail.wode.com.
150     IN      PTR     dns.wode.com.
150     IN      PTR     mail.wode.com.
150     IN      PTR     www.wode.com.
~
```

