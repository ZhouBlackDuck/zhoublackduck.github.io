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

## 使用LDAP

Nextcloud本身在图片和影音上并不出色，只是集成方便，于是我打算使用专门的软件对图片、影音进行管理，因此就需要用到统一验证

### 安装OpenLDAP、Authelia

- 拉取镜像

  ```bash
  sudo docker pull osixia/openldap
  ```

- 运行容器

  ```bash
  sudo docker run -d \
      -e LDAP_DOMAIN=<域名> \
      -e LDAP_ORGANISATION=nas \
      -e LDAP_ADMIN_PASSWORD=<password> \
      --name openldap \
      --restart always \
      osixia/openldap
  ```

- 拉取UI镜像

  ```bash
  sudo docker pull osixia/phpldapadmin
  ```

- 运行UI，登录用户`cn=admin,dc=二级域名,dc=顶级域名`

  ```bash
  sudo docker run -d \
      -e PHPLDAPADMIN_LDAP_HOSTS=<ldap-server-ip> \
      -e PHPLDAPADMIN_HTTPS=false \
      -e PHPLDAPADMIN_TRUST_PROXY_SSL=true \
      -v pla-data:/var/www/phpldapadmin \
      --name pla \
      --restart always \
      osixia/phpldapadmin
  ```

- 拉取镜像

  ```bash
  sudo docker pull authelia/authelia
  ```

- 拉取postgres

  ```bash
  sudo docker pull postgres
  ```

- 编写配置configuration.yml

  ```yaml
  default_2fa_method: 'totp'
  server:
    address: 'tcp://:9091/'
  totp:
    issuer: '域名'
  identity_validation:
    reset_password:
      jwt_secret: '密钥'
  authentication_backend:
    ldap:
      address: 'ldap://openldap:389'
      base_dn: 'dc=二级域名,dc=顶级域名'
      user: 'cn=admin,dc=二级域名,dc=顶级域名'
      password: 'admin-password'
      users_filter: '(&({username_attribute}={input})(objectClass=inetOrgPerson))'
      groups_filter: '(&(member={dn})(objectClass=groupOfNames))'
  access_control:
    rules:
    - domain: 'auth.域名'
      policy: 'bypass'
  session:
    secret: '密钥'
    cookies:
      - domain: '域名'
        authelia_url: 'https://auth.域名'
  storage:
    encryption_key: '密钥>32'
    postgres:
      address: 'tcp://authelia_postgres:5432'
      database: 'authelia'
      username: 'authelia'
      password: 'database-password'
  notifier:
    filesystem:
      filename: '/config/notification.txt'
  identity_providers:
    oidc:
      hmac_secret: '密钥64'
      jwks:
        - key: '密钥'
      clients:
        - client_id: 'immich'
          client_name: 'immich'
          client_secret: '密钥'
          public: false
          authorization_policy: 'one_factor'
          redirect_uris:
            - 'https://immich.域名/auth/login'
            - 'https://immich.域名/user-settings'
            - 'app.immich:///oauth-callback'
          scopes:
            - 'openid'
            - 'profile'
            - 'email'
          userinfo_signed_response_alg: 'none'
        - client_id: 'jellyfin'
          client_name: 'jellyfin'
          client_secret: ''
          public: false
          authorization_policy: 'one_factor'
          require_pkce: true
          pkce_challenge_method: 'S256'
          redirect_uris:
            - 'https://jellyfin.域名/sso/OID/redirect/Authelia'
            - 'https://jellyfin.域名/sso/OID/r/Authelia'
          scopes:
            - 'openid'
            - 'profile'
            - 'groups'
          userinfo_signed_response_alg: 'none'
          token_endpoint_auth_method: 'client_secret_post'
        - client_id: 'nextcloud'
          client_name: 'nextcloud'
          client_secret: ''
          public: false
          authorization_policy: 'one_factor'
          require_pkce: true
          pkce_challenge_method: 'S256'
          redirect_uris:
            - 'https://nextcloud.域名/apps/user_oidc/code'
          scopes:
            - 'openid'
            - 'profile'
            - 'email'
            - 'groups'
          userinfo_signed_response_alg: 'none'
          token_endpoint_auth_method: 'client_secret_post'
  ```

- 编写docker-compose.yml

  ```yaml
  services:
    authelia:
      container_name: authelia
      image: authelia/authelia:latest
      restart: always
      environment:
        TZ: Asia/Shanghai
      volumes:
        - /path/to/authelia/config:/config
      depends_on:
        - database
        - ldap
        
    database:
      container_name: authelia_postgres
      image: postgres:latest
      restart: always
      environment:
        POSTGRES_PASSWORD: <password>
        POSTGRES_USER: authelia
        POSTGRES_DB: authelia
      volumes:
        - ./postgres:/var/lib/postgresql/data
        
    ldap:
      container_name: openldap
      image: osixia/openldap:latest
      restart: always
      environment:
        LDAP_DOMAIN: <域名>
        LDAP_ORGANISATION: nas
        LDAP_ADMIN_PASSWORD: <password>
      volumes:
        - ./ldap/data:/var/lib/ldap
        - ./ldap/config:/etc/ldap/slapd.d
        
    pla:
      container_name: pla
      image: osixia/phpldapadmin:latest
      restart: always
      environment:
        PHPLDAPADMIN_LDAP_HOSTS: openldap
        PHPLDAPADMIN_HTTPS: false
        PHPLDAPADMIN_TRUST_PROXY_SSL: true
      volumes:
        - ./pla:/var/www/phpldapadmin
      depends_on:
        - ldap
  ```

- 启动编排

  ```bash
  sudo docker compose up -d
  ```

- 配置反向代理

  ```json
  https://ldap.域名:443 {
  	reverse_proxy http://pla-ip:80
  }
  https://auth.域名:443 {
      reverse_proxy http://authelia-ip:9091
  }
  ```

## NextCloud配置

- 安装OpenID Connect user backend应用

- 在`管理设置`里的`OpenID Connect`启用OpenID

- 添加Authelia

  > - Identifier: `Authelia`
  > - Client ID: `nextcloud`
  > - Client secret: `insecure_secret`
  > - Discovery endpoint: `https://auth.example.com/.well-known/openid-configuration`
  > - Scope: openid email profile

## 安装jellyfin

- 拉取镜像

  ```bash
  sudo docker pull jellyfin/jellyfin
  ```

- 运行镜像

  ```bash
  # 查询render组id，用于硬件加速
  getent group render | cut -d: -f3
  getent group video | cut -d: -f3
  ```

  ```bash
  sudo docker run -d \
      --name jellyfin \
      --user 33:33 \
      -v /path/to/jellyfin/config:/config \
      -v /path/to/jellyfin/cache:/cache \
      -v /path/to/media:/media \
      --restart always \
      --net host \
      --group-add="render-group-id" \
      --device /dev/dri/renderD128:/dev/dri/renderD128 \
      jellyfin/jellyfin
  ```
  
  ```json
  https://jellyfin.域名:443 {
  	reverse_proxy jellyfin.域名:8096
  }
  ```
  
- 在控制面板-常规-品牌添加以下内容

  ```html
  <form action="https://jellyfin.域名/sso/OID/start/服务提供商id">
    <button class="raised block emby-button button-submit">Login with 服务提供商id</button>
  </form>
  ```

  ```css
  #loginPage .readOnlyContent {
    display: flex;
    flex-direction: column-reverse;
  }
  
  .loginDisclaimerContainer {
    margin-top: 0;
    margin-bottom: 1em;
  }
  
  .loginDisclaimer {
    width: 100%;
    height: 100%;
  }
  ```

- 安装jellyfin sso插件

- 填写配置

  > 1. Visit the [Jellyfin](https://jellyfin.org/) Administration Dashboard.
  > 2. Visit the `Plugins` section.
  > 3. Visit the `Repositories` tab.
  > 4. Click the `+` to add a repository.
  > 5. Enter the following details:
  >    1. Repository Name: `Jellyfin SSO`
  >    2. Repository URL: `https://raw.githubusercontent.com/9p4/jellyfin-plugin-sso/manifest-release/manifest.json`
  > 6. Click `Save`.
  > 7. Click `Ok` to confirm the repository installation.
  > 8. Visit the `Catalog` tab.
  > 9. Select `SSO Authentication` from the `Authentication` section.
  > 10. Click `Install`.
  > 11. Click `Ok` to confirm the plugin installation.
  > 12. Once installed restart [Jellyfin](https://jellyfin.org/).
  > 13. Complete steps 1 and 2 again.
  > 14. Click the `SSO-Auth` plugin.
  > 15. Add a provider with the following settings:
  >     1. Name of the OID Provider: `Authelia`
  >     2. OID Endpoint: `https://auth.example.com`
  >     3. OpenID Client ID: `jellyfin`
  >     4. OID Secret: `insecure_secret`
  >     5. Enabled: Checked
  >     6. Enable Authorization by Plugin: Checked
  >     7. Enable All Folders: Checked
  >     8. Roles: `jellyfin-users`
  >     9. Admin Roles: `jellyfin-admins`
  >     10. Role Claim: `groups`
  >     11. Request Additional Scopes: `groups`
  >     12. Set default username claim: `preferred_username`
  > 16. All other options may remain unchecked or unconfigured.
  > 17. Click `Save`.

## 安装immich

  - 获取docker-compose.yml和example.env
  
    ```bash
    wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
    wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
    # 可选，获取硬件转码配置
    wget -O hwaccel.transcoding.yml https://github.com/immich-app/immich/releases/latest/download/hwaccel.transcoding.yml
    ```
  
  - 编辑`.env`
  
    ```sh
    # You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables
    
    # The location where your uploaded files are stored
    UPLOAD_LOCATION=/custom/path/immich/
    # The location where your database files are stored
    DB_DATA_LOCATION=数据库数据位置
    
    # To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
    TZ=Asia/Shanghai
    
    # The Immich version to use. You can pin this to a specific version like "v1.71.0"
    IMMICH_VERSION=release
    
    # Connection secret for postgres. You should change it to a random password
    # Please use only the characters `A-Za-z0-9`, without special characters or spaces
    DB_PASSWORD=密码
    
    # The values below this line do not need to be changed
    ###################################################################################
    DB_USERNAME=postgres
    DB_DATABASE_NAME=immich
    ```
  
  - 编辑`docker-compose.yml`
  
    ```yaml
    #
    # WARNING: Make sure to use the docker-compose.yml of the current release:
    #
    # https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
    #
    # The compose file on main may not be compatible with the latest release.
    #
    
    name: immich
    
    services:
      immich-server:
        container_name: immich_server
        image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
        extends:
          file: hwaccel.transcoding.yml
          service: 加速后端 # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
        volumes:
          # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
          - ${UPLOAD_LOCATION}:/usr/src/app/upload/upload
          - /etc/localtime:/etc/localtime:ro
        env_file:
          - .env
        ports:
          - 2283:3001
        depends_on:
          - redis
          - database
        restart: always
        healthcheck:
          disable: false
    
      immich-machine-learning:
        container_name: immich_machine_learning
        # For hardware acceleration, add one of -[armnn, cuda, openvino] to the image tag.
        # Example tag: ${IMMICH_VERSION:-release}-cuda
        image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
        # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
        #   file: hwaccel.ml.yml
        #   service: cpu # set to one of [armnn, cuda, openvino, openvino-wsl] for accelerated inference - use the `-wsl` version for WSL2 where applicable
        volumes:
          - model-cache:/cache
        env_file:
          - .env
        restart: always
        healthcheck:
          disable: false
    
      redis:
        container_name: immich_redis
        image: docker.io/redis:6.2-alpine@sha256:2d1463258f2764328496376f5d965f20c6a67f66ea2b06dc42af351f75248792
        healthcheck:
          test: redis-cli ping || exit 1
        restart: always
    
      database:
        container_name: immich_postgres
        image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:90724186f0a3517cf6914295b5ab410db9ce23190a2d9d0b9dd6463e3fa298f0
        environment:
          POSTGRES_PASSWORD: ${DB_PASSWORD}
          POSTGRES_USER: ${DB_USERNAME}
          POSTGRES_DB: ${DB_DATABASE_NAME}
          POSTGRES_INITDB_ARGS: '--data-checksums'
        volumes:
          # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
          - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
        healthcheck:
          test: pg_isready --dbname='${DB_DATABASE_NAME}' --username='${DB_USERNAME}' || exit 1; Chksum="$$(psql --dbname='${DB_DATABASE_NAME}' --username='${DB_USERNAME}' --tuples-only --no-align --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM pg_stat_database')"; echo "checksum failure count is $$Chksum"; [ "$$Chksum" = '0' ] || exit 1
          interval: 5m
          start_interval: 30s
          start_period: 5m
        command: ["postgres", "-c", "shared_preload_libraries=vectors.so", "-c", 'search_path="$$user", public, vectors', "-c", "logging_collector=on", "-c", "max_wal_size=2GB", "-c", "shared_buffers=512MB", "-c", "wal_compression=on"]
        restart: always
    
    volumes:
      model-cache:
    ```

  - 启动compose
  
    ```bash
    sudo docker compose up -d
    ```

  - 配置反向代理
  
    ```bash
    sudo docker network connect immich_default caddy
    ```
  
    ```json
    https://immich.域名:443 {
    	reverse_proxy http://immich-server-ip:3001 # 或http://localhost:2283
    }
    ```
  
  - 进入服务器设置-Oauth设置

    >- Issuer URL: `https://auth.example.com/.well-known/openid-configuration`.
    >- Client ID: `immich`.
    >- Client Secret: `insecure_secret`.
    >- Scope: `openid profile email`.
    >- Button Text: `Login with Authelia`.
    >- Auto Register: Enable if desired.

## 使用Authentik替换Authelia

- 获取docker-compose配置

  ```bash
  wget https://goauthentik.io/docker-compose.yml
  ```

- 编辑`.env`

  ```shell
  PG_PASS=密码
  AUTHENTIK_SECRET_KEY=密钥
  
  AUTHENTIK_ERROR_REPORTING__ENABLED=false
  
  # SMTP Host Emails are sent to
  #AUTHENTIK_EMAIL__HOST=localhost
  #AUTHENTIK_EMAIL__PORT=25
  # Optionally authenticate (don't add quotation marks to your password)
  #AUTHENTIK_EMAIL__USERNAME=
  #AUTHENTIK_EMAIL__PASSWORD=
  # Use StartTLS
  #AUTHENTIK_EMAIL__USE_TLS=false
  # Use SSL
  #AUTHENTIK_EMAIL__USE_SSL=false
  #AUTHENTIK_EMAIL__TIMEOUT=10
  # Email address authentik will send from, should have a correct @domain
  #AUTHENTIK_EMAIL__FROM=authentik@localhost
  
  #COMPOSE_PORT_HTTP=80
  #COMPOSE_PORT_HTTPS=443
  ```

  ```bash
  docker compose pull
  docker compose up -d
  ```

- 访问`http://server-ip:9000/if/flow/initial-setup/`开启初始化流程

## 系统监控

### 安装Portainer

- 拉取镜像

  ```bash
  docker pull portainer/portainer-ce
  ```

- 启动容器

  ```bash
  docker run -d --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
  ```

### 安装Netdata

- 拉取镜像

  ```bash
  docker pull netdata/netdata
  ```

- 运行容器

  ```bash
  docker run -d --name=netdata \
    --pid=host \
    --network=host \
    -v netdataconfig:/etc/netdata \
    -v netdatalib:/var/lib/netdata \
    -v netdatacache:/var/cache/netdata \
    -v /:/host/root:ro,rslave \
    -v /etc/passwd:/host/etc/passwd:ro \
    -v /etc/group:/host/etc/group:ro \
    -v /etc/localtime:/etc/localtime:ro \
    -v /proc:/host/proc:ro \
    -v /sys:/host/sys:ro \
    -v /etc/os-release:/host/etc/os-release:ro \
    -v /var/log:/host/var/log:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /run/dbus:/run/dbus:ro \
    --restart unless-stopped \
    --cap-add SYS_PTRACE \
    --cap-add SYS_ADMIN \
    --security-opt apparmor=unconfined \
    netdata/netdata
  ```

## 使用UIforFreedom替换clash

### 安装

  ```bash
  docker pull ui4freedom/uif:latest # 拉取最新镜像
  docker run --network host --name uif --privileged --restart unless-stopped -d ui4freedom/uif:latest
  ```

### 配置

```bash
docker logs -f uif
# Password: 92c204a9-3934-4976-96f2-7bbcb338ccf0
# Web Address: 0.0.0.0:9527
# API Address: 0.0.0.0:9413
```

打开网址`ip:9527`配置api后端为`ip:9413`

在入站规则中关闭`系统代理`，根据自己的需要配置入站规则，即连接协议和端口等

在出站规则中添加订阅链接，启用节点，完成
