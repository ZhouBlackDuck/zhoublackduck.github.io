---
date: 2024-10-08
title: 组建NAS（五）
categories: 记录
tags:
  - DIY
  - NAS
---

## 使用TailScale

虽然NAS已经可以正常使用，但是ZeroTier对手机端的支持并没有想象中好，于是选择更换组网框架——TailScale

### 安装TailScale

1. 停止并删除ZeroTier相关容器，删除相关数据卷和数据文件，删除相关镜像

2. 到官网[Tailscale · Best VPN Service for Secure Networks](https://tailscale.com/)创建一个网络，并生成auth-key

3. 拉取TailScale镜像

   ```bash
   docker pull tailscale/tailscale:latest
   ```

4. 运行容器

   ```bash
   sudo docker run -d --name tailscale --restart always --cap-add NET_ADMIN --cap-add SYS_MODULE -v /dev/net/tun:/dev/net/tun -v tailscale-state:/var/lib/tailscale -e TS_AUTHKEY=<auth-key> -e TS_STATE_DIR=/var/lib/tailscale -e TS_USERSPACE=false -e TS_ACCEPT_DNS=true --hostname <hostname> --network host tailscale/tailscale:latest
   ```

5. 访问网络控制面板，解除对设备的有期限授权

6. 在控制面板关闭MagicDNS，添加自定义DNS服务器

7. 在DNS服务器添加本地DNS记录

8. 其他客户端只需下载相应操作系统客户端并使用同一账户登录即可

