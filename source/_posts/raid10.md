---
title: raid10制作
tags: 
- 系统
- 存储
---

```sh
1[root@linuxprobe ~]# mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sdb /dev/sdc /dev/sdd /dev/sde

2[root@linuxprobe ~]# mkfs.ext4 /dev/md0

3[root@linuxprobe ~]# mkdir /RAID
 [root@linuxprobe ~]# mount /dev/md0 /RAID
 [root@linuxprobe ~]# df -h

4[root@linuxprobe ~]# mdadm -D /dev/md0 查看

5损坏修复
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdb
[root@linuxprobe ~]# mdadm -D /dev/md0
[root@linuxprobe ~]# umount /RAID
[root@linuxprobe ~]# mdadm /dev/md0 -a /dev/sdb
[root@linuxprobe ~]# mdadm -D /dev/md0
```

