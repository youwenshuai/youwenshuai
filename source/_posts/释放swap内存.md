---
title: linux释放swap内存
tags: 系统
---

#### 一：如何释放swap内存

1. 重启占用swap的进程
2. 关闭swap分区(有的时候无法查到占用swap的进程)

#### 二：关闭swap分区需要注意的事项和步骤

1. 确保系统空闲内存大于Swap已用内存
2. 可以先清理内存cache，空出足够内存（echo "1" > /proc/sys/vm/drop_caches）
3. 关闭Swap分区(swapoff -a)，这个过程需要时间等待
4. swap分区释放后，恢复Swap分区(swapon -a)
5. 恢复内存cache的设置(ehco "0" > /proc/sys/vm/drop_caches)

#### 三：释放swap

1. 查看当前swap分区挂载位置

   swapon -s 

2. 关停这个分区

   swapoff /dev/sda5

3. 再次查看状态

   swapon -s

4. 查看内存情况

   free -m

5. 将swap挂载在原分区

   swapon /dev/sda5