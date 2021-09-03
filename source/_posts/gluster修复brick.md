---
title: gluster修复brick方法
tags: 存储
---

```sh
挂载fatab

###CC01

重启下glus
df -h
创建新的brick目录
mkdir /gfs/kvmiso1
mkdir /gfs/data1
mkdir /gfs/tv1
mkdir /gfs/mysqlbackup1

###CC02

getfattr -d -m. -e hex /gfs/kvmiso
getfattr -d -m. -e hex /gfs/data
getfattr -d -m. -e hex /gfs/tv
getfattr -d -m. -e hex /gfs/mysqlbackup

###CC01

mount -t glusterfs  ghstorage01:/s1 /test1
mount -t glusterfs  ghstorage01:/s2 /test2
mount -t glusterfs  ghstorage01:/s3 /test3
mount -t glusterfs  ghstorage01:/s4 /test4
mkdir /test1/testDir001
mkdir /test2/testDir001
mkdir /test3/testDir001
mkdir /test4/testDir001
rmdir /test1/testDir001
rmdir /test2/testDir001
rmdir /test3/testDir001
rmdir /test4/testDir001
setfattr -n trusted.non-existent-key -v abc /test1
setfattr -n trusted.non-existent-key -v abc /test2
setfattr -n trusted.non-existent-key -v abc /test3
setfattr -n trusted.non-existent-key -v abc /test4
setfattr -x trusted.non-existent-key /test1
setfattr -x trusted.non-existent-key /test2
setfattr -x trusted.non-existent-key /test3
setfattr -x trusted.non-existent-key /test4

###CC02

getfattr -d -m. -e hex /gfs/kvmiso
getfattr -d -m. -e hex /gfs/data
getfattr -d -m. -e hex /gfs/tv
getfattr -d -m. -e hex /gfs/mysqlbackup

###CC01

gluster volume heal s1 info
gluster volume heal s2 info
gluster volume heal s3 info
gluster volume heal s4 info
gluster volume replace-brick s1 ghstorage01:/gfs/kvmiso ghstorage01:/gfs/kvmiso1 commit force
gluster volume replace-brick s2 ghstorage01:/gfs/data ghstorage01:/gfs/data1 commit force
gluster volume replace-brick s3 ghstorage01:/gfs/tv ghstorage01:/gfs/tv1 commit force
gluster volume replace-brick s4 ghstorage01:/gfs/mysqlbackup ghstorage01:/gfs/mysqlbackup1 commit force
```

