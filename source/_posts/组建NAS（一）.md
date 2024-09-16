---
title: 组建NAS（一）
date: 2024-09-13
categories: 记录
tags:
  - DIY
  - NAS
---

## 硬件

- 机箱：见方L 8盘位机箱
- 主板：华硕 PRIME A520M-K
- CPU：AMD RYZEN 5 5600G
- 显卡：核显
- CPU散热：利民 Silver Soul 110 BLACK
- 机箱风扇：
  - 利民 C12C×3
  - 利民 C12015B×2
  - 利民 P9×1
- 内存：光威 天策 32GB(16GB×2) DDR4 3600MHz
- 系统硬盘：爱国者 SSD P3500 256GB
- 电源：安钛克 NE650W GOLD 全模组
- NAS硬盘：希捷 IronWolf ST2000VN004×2
- 配件：
  - 利民 FAN HUB X4 集线器×1
  - 利民 M.2 2280 固态散热马甲×1
  - 乐扩 PCIe Gen3 x1 转 4 SATA Gen3 扩展卡×1
  - SATA线×8
  - 安钛克 6pin 转 4 大4pin 全模组定制线×1

## 组装

1. 拆除机箱前置风扇面板和电源面板、移除顶盖、拆下硬盘笼
2. 将前置风扇利民 C12015B×2安装到前置风扇位置，风向前进后出
3. 将IO挡板安装到机箱后部
4. 将利民 C12C×1安装到机箱侧面风扇处，用于出风
5. 往主板上依次安装CPU、CPU散热（从内存方向进风从IO接口方向出风）、内存、系统硬盘、固态散热马甲，并将CPU散热电源线插到主板CPU_FAN
6. 将主板安装到机箱底部
7. 安装PCIe扩展卡
8. 将利民 P9×1安装到机箱后部风扇处，用于出风
9. 将全模组电源线按位置插到主板电源、CPU供电口，将集线器插到主板CHA_FAN，将机箱供电线插到电源供电口
10. 安装电源到机箱后部电源处
11. 将主板供电线、CPU供电线插入电源
12. 将SATA线×8、6pin 转 大4pin电源线插到硬盘笼背板
13. 安装硬盘笼到机箱前部
14. 将SATA线接入主板，硬盘笼供电线插入电源
15. 将利民 C12C×2安装到顶盖风扇槽
16. 将机箱风扇供电线插入集线器，并将集线器贴在机箱内侧
17. 将机箱前置电源面板的电源按钮和前置USB插针接到主板
18. 安装前置面板和顶盖，插入电源和硬盘

## 系统和软件

- ### 安装系统

  1. 将键盘、鼠标、显示器和引导U盘（存放系统镜像，这里是Ubuntu22.04）插入机箱
  2. 启动主机，进入BIOS（注意检查CPU温度），选择从引导U盘启动
  3. 按指引安装系统
  4. 拔出引导U盘、重启系统

- ### 移除snap

  - ```sh
    sudo apt autoremove --purge snapd
    ```

- ### 系统盘扩容

  - ```sh
    lsblk
    ```

    ```
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    nvme0n1                   259:0    0 238.5G  0 disk 
    ├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
    ├─nvme0n1p2               259:2    0     2G  0 part /boot
    └─nvme0n1p3               259:3    0 235.4G  0 part 
      └─ubuntu--vg-ubuntu--lv 253:0    0   100G  0 lvm  /
    ```

  - 可以看到系统根目录只分配了100G

  - ```sh
    sudo vgdisplay
    ```

    ```
      --- Volume group ---
      VG Name               ubuntu-vg
      System ID             
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  2
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                1
      Open LV               1
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               235.42 GiB
      PE Size               4.00 MiB
      Total PE              60268
      Alloc PE / Size       25600 / 100.00 GiB
      Free  PE / Size       34668 / 135.42 GiB
      VG UUID               aoRfz0-qgEh-A7rr-FJmB-s5MW-MJfW-rnfvfX
    ```

  - 卷组VG已经占满硬盘空间PV，说明逻辑卷LV空间没占满

  - ```sh
    sudo lvdisplay
    ```

    ```
      --- Logical volume ---
      LV Path                /dev/ubuntu-vg/ubuntu-lv
      LV Name                ubuntu-lv
      VG Name                ubuntu-vg
      LV UUID                JGyBIQ-2alO-XCXr-gCft-VBlS-xxlz-MwRmlf
      LV Write Access        read/write
      LV Creation host, time ubuntu-server, 2024-09-13 20:55:48 +0000
      LV Status              available
      # open                 1
      LV Size                100.00 GiB
      Current LE             25600
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           253:0
    ```

  - ```sh
    sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
    ```

  - 再次查看硬盘情况

    ```
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    nvme0n1                   259:0    0 238.5G  0 disk 
    ├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
    ├─nvme0n1p2               259:2    0     2G  0 part /boot
    └─nvme0n1p3               259:3    0 235.4G  0 part 
      └─ubuntu--vg-ubuntu--lv 253:0    0 235.4G  0 lvm  /
    ```

- ### 支持中文

  - ```sh
    sudo apt update
    sudo apt install language-pack-zh-hans
    localectl set-locale LANG=zh_CN.UTF-8
    reboot
    ```

- ### 安装代理

  - 解压clash并赋予可执行权限

  - 下载Country.mmdb文件移动至~/.config/clash文件夹

  - 用自己的配置修改~/.config/clash/config.yaml

  - 打开系统代理（添加代理环境变量）

  - 启动clash

  - 通过在配置中填写`external-controller`开启外部控制

    > ### 认证
    >
    > - 外部控制器接受`Bearer Token`作为认证方式
    >   - 使用`Authorization: Bearer <Your Secret>`添加到请求头以进行验证
    >
    > ### API
    >
    > - `/logs`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取实时日志
    >
    >     - ```sh
    >       curl -X GET localhost:9090/logs
    >       ```
    >
    > - `/traffic`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取实时流量
    >
    >     - ```sh
    >       curl -X GET localhost:9090/traffic
    >       ```
    >
    > - `/version`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取Clash版本
    >
    >     - ```sh
    >       curl -X GET localhost:9090/version
    >       ```
    >
    > - `/configs`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取基础配置
    >
    >     - ```sh
    >       curl -X GET localhost:9090/configs
    >       ```
    >
    >   - 方法：`PATCH`
    >
    >     - 描述：增量修改配置
    >
    >     - ```sh
    >       curl -X PATCH -d '{}' localhost:9090/configs
    >       ```
    >
    > - `/proxies`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取所有节点/选择器信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/proxies
    >       ```
    >
    > - `/proxies/:name`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取指定节点/选择器信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/proxies/nodeOrSelectorName
    >       ```
    >
    >   - 方法：`PUT`
    >
    >     - 描述：切换选择器中选中节点
    >
    >     - ```sh
    >       curl -X PUT -d '{"name": "nodeName"}' localhost:9090/proxies/SelectorName
    >       ```
    >
    > - `/proxies/:name/delay`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取指定节点延迟
    >
    >     - ```sh
    >       curl -X GET "localhost:9090/proxies/nodeOrSelectorName/delay?url=http://xxxx&timeout=xxxx"
    >       ```
    >
    > - `/rules`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取规则信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/rules
    >       ```
    >
    > - `/connections`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取连接信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/connections
    >       ```
    >
    >   - 方法：`DELETE`
    >
    >     - 描述：关闭所有连接
    >
    >     - ```sh
    >       curl -X DELETE localhost:9090/connections
    >       ```
    >
    > - `/connections/:id`
    >
    >   - 方法：`DELETE`
    >
    >     - 描述：关闭指定连接
    >
    >     - ```sh
    >       curl -X DELETE localhost:9090/connections/connectionID
    >       ```
    >
    > - `/providers/proxies`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取所有代理集信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/providers/proxies
    >       ```
    >
    > - `/providers/proxies/:name`
    >
    >   - 方法：`GET`
    >
    >     - 描述：获取指定代理集信息
    >
    >     - ```sh
    >       curl -X GET localhost:9090/providers/proxies/proxyName
    >       ```

