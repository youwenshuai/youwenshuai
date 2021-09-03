---
title: docker报错案例
tags: docker
---

> docker报错Get https://registry-1.docker.io/v2/: x509: certificate has expired or is not yet valid
>
> 这个错误一般都是本地系统时间错误导致报错证书过期
>
> 只需要输入
>
> ntpdate cn.pool.ntp.org 
>
> 同步一下时间
>
> 然后date查看下就ok了

