---
titel: 组建NAS（三）
date: 2024-09-23
categories: 记录
tags:
  - DIY
  - NAS
---

## 使用dae做网卡级代理

1. 删除docker关于代理的配置

2. 停止所有容器

3. 删除所有容器

4. 删除nextcloud-aio网络

5. 删除所有卷

6. 拉取dae镜像

   ```bash
   sudo docker pull daeuniverse/dae
   ```

7. 启动容器

   ```bash
   sudo docker run -d \
       --restart always \
       --network host \
       --pid host \
       --privileged \
       -v /sys:/sys \
       -v /etc/dae:/etc/dae \
       --name dae \
       daeuniverse/dae:latest
   ```

8. 在/etc/dae目录下创建config.dae文件，修改权限为640

   ```json
   global {
       log_level: info
       lan_interface: docker0
       wan_interface: auto
       auto_config_kernel_parameter: true
   }
   node {
       sock: 'socks5://localhost:7890'
       http: 'http://localhost:7890'
   }
   dns {
       upstream {
           alidns: 'udp://dns.alidns.com:53'
           googledns: 'tcp+udp://dns.google.com:53'
       }
       routing {
           request {
               qname(geosite:cn) -> alidns
               fallback: googledns
           }
       }
       routing {
           request {
               fallback: alidns
           }
           response {
               upstream(googledns) -> accept
               ip(geoip:private) && !qname(geosite:cn) -> googledns
               fallback: accept
           }
       }
   }
   group {
       my_group {
       policy: min_moving_avg
   }
   }
   routing {
       pname(clash) -> must_direct
       dip(geoip:private) -> direct
       dip(geoip:cn) -> direct
       domain(geosite:cn) -> direct
       domain(home.arpa) -> direct
       fallback: my_group
   }
   ```

9. 运行zerotier容器、zeronsd容器

10. 运行nginx proxy manager容器

11. 运行nextcloud-aio容器，将nextcloud-aio网卡加入dae

## 安装ftp服务器

1. 拉取镜像

   ```bash
   sudo docker pull kibatic/proftpd
   ```

2. 运行镜像

   ```bash
   sudo docker run -d --net host \
   	-e FTP_LIST="admin:<password>" \
   	-e MASQUERADE_ADDRESS=<serverIP> \
   	-v /path_to_ftp_dir:/home/admin \
   	--name proftpd \
   	kibatic/proftpd:latest
   ```
   

## 切换zerotier-aio

1. 拉取镜像

   ```bash
   sudo docker pull imashen/zerotier-aio
   ```

2. 运行容器

   ```bash
   sudo docker run -d -p 9993:9993/udp -p 3443:3443 -p 3180:3180 \
       -v zerotier-one:/var/lib/zerotier-one \
       -v zerotier-webui:/www/zerotier-webui/etc \
       -v zerotier-logs:/logs \
       -e NODE_ENV=production \
       -e ZEROTIER-WEBUI_PASSWD=<password> \
       -e HTTPS_PORT=3443 \
       --name zerotier-aio \
       imashen/zerotier-aio
   ```

3. 额，由于该镜像不支持zeronsd做DNS，所有最后只是取了容器中mkplanet工具生成planet文件，分发给各个客户端做加速