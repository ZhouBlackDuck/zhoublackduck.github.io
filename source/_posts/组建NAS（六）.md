---
title: 组建NAS（六）
date: 2025-02-25
categories:
  - 记录
tags:
  - NAS
  - 日志
  - Docker
---

### Docker日志管理

修改Docker守护进程配置`/etc/docker/daemon.json`

```json
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "512m",
        "max-file": "3"
    }
}
```

对新建容器生效

### Docker IPv6

编辑`/etc/sysctl.conf`开启IPv6转发

```bash
echo "net.ipv6.conf.all.forwarding = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

编辑`/etc/docker/daemon.json`开启IPv6

```json
{
    "ipv6": true,
    "fixed-cidr-v6": "fd13:a4f3:a0b0::/64",
    "experimental": true,
    "ip6tables": true
}
```

对新建容器生效
