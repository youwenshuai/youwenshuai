---
title: htpasswd创建密码文件
tags: 系统
---

##### htpasswd

htpasswd指令用来创建和更新用于基本认证的用户认证密码文件。htpasswd指令必须对密码文件有读写权限，否则会返回错误码。

   此命令的适用范围：RedHat、RHEL、Ubuntu、CentOS、Fedora。

##### 语法

**htpasswd [ -c ] [ -m ] [ -D ] passwdfile username**

**htpasswd -b [ -c ] [ -m | -d | -p | -s ] [ -D ] passwdfile username password**

**htpasswd -n [ -m | -d | -s | -p ] username**

**htpasswd -nb [ -m | -d | -s | -p ] username password**

##### 参数列表

| 选项 | 说明                                                 |
| ---- | ---------------------------------------------------- |
| -b   | 使用批处理方式，直接从命令行获取密码，不提示用户输入 |
| -c   | 创建密码文件，如果文件存在，那么内容被清空重写       |
| -n   | 将结果送到标准输出                                   |
| -m   | 使用MD5加密                                          |
| -s   | 使用crypt()加密                                      |
| -p   | 使用文本密码                                         |
| -D   | 从认证文件中删除用户记录                             |

##### 实例

```
htpasswd -cm htpfile1 weijie        //创建认证文件，使用md5加密
New password: 
Re-type new password: 
Adding password for user weijie 
You have new mail in /var/spool/mail/root
cat htpfile1                           //显示认证文件
weijie:$apr1$/RxQ5LT9$L1WJPkxknMizG5DwGVGv4.
```

