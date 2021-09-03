---
title: docker 日志大小限制
tags: docker
---

创建/etc/docker/daemon.json如果已经存在则不用创建

```sh
cat /etc/docker/daemon.json
{
"registry-mirrors": ["http://"],
"log-driver":"json-file",
"log-opts":{"max-size":"1G","max-file":"1"}
}
systemctl daemon-reload
systemctl restart docker
```

