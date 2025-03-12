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

### Bind9权威DNS

拉取镜像

```bash
docker pull internetsystemsconsortium/bind9:9.20
```

启动容器

```bash
docker run \
        --name=bind9 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        # 远程管理端口
        # --publish 127.0.0.1:953:953/tcp \
        # name.conf
        --volume /etc/bind \
        # 工作目录
        --volume /var/cache/bind \
        # 次级域
        --volume /var/lib/bind \
        # 日志
        --volume /var/log \
        internetsystemsconsortium/bind9:9.20
```

编辑配置文件`/etc/bind/named.conf`

```
options {
	directory "/var/cache/bind";
	listen-on { 127.0.0.1; };
	listen-on-v6 { ::1; };
	allow-recursion {
		none;
	};
	allow-update {
		none;
	};
	allow-transfer {
		none;
	};
}
include "/path/to/zone/conf/definition";
```

编辑`include`指向的配置文件

```
zone "xxx.xxx." {
	type primary;
	file "/path/to/zone/file";
}
```

编写反向域文件以便提供PTR记录
