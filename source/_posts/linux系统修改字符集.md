---
title: linux系统修改字符集
tags: 系统
---

```bash
#echo $LANG
en_US.UTF-8
#env |grep LANG
LANG=en_US.UTF-8
#locale |grep CTYPE
LC_CTYPE="en_US.UTF-8"
```

更改系统字符集

执行export LANG=(字符集名称)

常用字符集：en_US.utf8、zh_CN.gb2312、zh_CN.gbk、zh_CN.utf8