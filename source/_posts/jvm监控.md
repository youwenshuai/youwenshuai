---
title: jvm监控项
tags:
- java
- 监控
---

> 获取tomcat的最大线程数。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345  'Catalina:name="http-bio-8080",type=ThreadPool' maxThreads
> 06/09/2017 17:34:47 +0800 org.archive.jmx.Client maxThreads: 200
>
> 获取tomcat繁忙线程数。
>  java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345 'Catalina:name="http-bio-8080",type=ThreadPool' currentThreadsBusy
> 06/09/2017 17:36:48 +0800 org.archive.jmx.Client currentThreadsBusy: 5
>
> 获取tomcat当前已经分配线程数。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345  'Catalina:name="http-bio-8080",type=ThreadPool' currentThreadCount
> 06/09/2017 17:38:15 +0800 org.archive.jmx.Client currentThreadCount: 11
>
> 获取活动线程的当前数目，包括守护线程和非守护线程。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345 java.lang:type=Threading ThreadCount
> 06/09/2017 17:55:34 +0800 org.archive.jmx.Client ThreadCount: 225
>
> 获取自从 Java 虚拟机启动以来创建和启动的线程总数目。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345 java.lang:type=Threading TotalStartedThreadCount
> 06/09/2017 17:55:52 +0800 org.archive.jmx.Client TotalStartedThreadCount: 112225
>
> 获取Java 虚拟机启动或峰值重置以来峰值活动线程计数。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345 java.lang:type=Threading PeakThreadCount
> 06/09/2017 17:56:06 +0800 org.archive.jmx.Client PeakThreadCount: 244
>
> 获取守护线程总数。
> java -jar cmdline-jmxclient-0.10.3.jar - 192.168.1.100:12345 java.lang:type=Threading DaemonThreadCount
> 06/09/2017 17:52:20 +0800 org.archive.jmx.Client DaemonThreadCount: 195

-------

> 名称：tomcat已分配线程数
> 键值：jmx["Catalina:name=\"http-bio-8080\",type=ThreadPool",currentThreadCount]
>
> 名称：tomcat最大线程数
> 键值：jmx["Catalina:name=\"http-bio-8080\",type=ThreadPool",maxThreads]
>
> 名称：tomcat繁忙线程数
> 键值：jmx["Catalina:name=\"http-bio-8080\",type=ThreadPool",currentThreadsBusy]
>
> 名称：java虚拟机启动以来创建和启动的线程总数目
> 键值：jmx["java.lang:type=Threading","TotalStartedThreadCount"]
>
> 名称：tomcat活动线程的当前数目，包括守护线程和非守护线程。
> 键值：jmx["java.lang:type=Threading","ThreadCount"]
>
> 名称：java虚拟机启动或峰值重置以来峰值活动线程数
> 键值：jmx["java.lang:type=Threading","PeakThreadCount"]
>







