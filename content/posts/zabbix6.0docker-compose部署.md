---
title: "zabbix6.0+docker-compose部署"
date: 2024-10-27T15:43:25+08:00
tags:
  - zabbix
draft: false
type: "post"
---

zabbix6.0 已是新LTS版本。根据zabbix-docker上的trunk版本来搭建zabbix6.0。

根据踩坑，记录docker-compose 执行后遇到的一些问题。

## zabbix 6.0 LTS已发布
本文中的镜像为当时为zabbix6.0预发布版本（trunk）。目前zabbix6.0LTS版本已发布。
请结合 [官方镜像](https://hub.docker.com/u/zabbix) 

## 部署

主机安装好docker、docker-compose。

文件包含:[env_vars](https://url21.ctfile.com/f/2802921-534381915-0158c8)  ,

[docker-compose.yml](https://url21.ctfile.com/f/2802921-534383227-cb4ab7 )。

*密码6387*

下载完成后解压到同一目录，  并执行`docker-compose up -d`

这时候可以看到各服务拉取镜像并启动。

## docker-compose 文件内容

```
version: '3.5'
services:
  m-server:
    container_name: m-server
    image: zabbix/zabbix-server-mysql:alpine-trunk
    restart: always
    ports:
      - "10051:10051"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zbx_env/usr/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts:ro
      - ./zbx_env/usr/lib/zabbix/externalscripts:/usr/lib/zabbix/externalscripts:ro
      - ./zbx_env/var/lib/zabbix/export:/var/lib/zabbix/export:rw
      - ./zbx_env/var/lib/zabbix/modules:/var/lib/zabbix/modules:ro
      - ./zbx_env/var/lib/zabbix/enc:/var/lib/zabbix/enc:ro
      - ./zbx_env/var/lib/zabbix/ssh_keys:/var/lib/zabbix/ssh_keys:ro
      - ./zbx_env/var/lib/zabbix/mibs:/var/lib/zabbix/mibs:ro
    env_file:
      - ./env_vars/.env_db_mysql
      - ./env_vars/.env_srv
    secrets:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_ROOT_PASSWORD
    #   - client-key.pem
    #   - client-cert.pem
    #   - root-ca.pem
    depends_on:
      - mysql-server
    networks:
      zbx_net_backend:
        aliases:
          - m-erver
          - m-server-mysql
          - m-server-alpine-mysql
          - m-server-mysql-alpine
      zbx_net_frontend: null
        #  devices:
        #   - "/dev/ttyUSB0:/dev/ttyUSB0"
    stop_grace_period: 30s
    sysctls:
      - net.ipv4.ip_local_port_range=1024 65000
      - net.ipv4.conf.all.accept_redirects=0
      - net.ipv4.conf.all.secure_redirects=0
      - net.ipv4.conf.all.send_redirects=0
    labels:
      com.zabbix.description: "Zabbix server with MySQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "m-server"
      com.zabbix.dbtype: "mysql"
      com.zabbix.os: "alpine"
  m-web-nginx-mysql:
    container_name: m-web-nginx-mysql
    #  image: zabbix/zabbix-web-nginx-mysql:alpine-trunk
    image: chenwl2016/m-web-nginx-mysql:v1-alpine-trunk
    ports:
      - "8082:8080"
      - "8443:8443"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zbx_env/etc/ssl/nginx:/etc/ssl/nginx:ro
      - ./zbx_env/usr/share/zabbix/modules/:/usr/share/zabbix/modules/:ro
    env_file:
      - ./env_vars/.env_db_mysql
      - ./env_vars/.env_web
    secrets:
      - MYSQL_USER
      - MYSQL_PASSWORD
    #   - client-key.pem
    #   - client-cert.pem
    #   - root-ca.pem
    depends_on:
      - mysql-server
      - m-server
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:8080/" ]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    networks:
      zbx_net_backend:
        aliases:
          - m-web-nginx-mysql
          - m-web-nginx-alpine-mysql
          - m-web-nginx-mysql-alpine
      zbx_net_frontend: null
    stop_grace_period: 10s
    sysctls:
      - net.core.somaxconn=65535
    labels:
      com.zabbix.description: "Zabbix frontend on Nginx web-server with MySQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "m-frontend"
      com.zabbix.webserver: "nginx"
      com.zabbix.dbtype: "mysql"
      com.zabbix.os: "alpine"
  m-agent:
    container_name: m-agent
    image: zabbix/zabbix-agent:alpine-trunk
    restart: always
    ports:
      - "10050:10050"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zbx_env/etc/zabbix/zabbix_agentd.d:/etc/zabbix/zabbix_agentd.d:ro
      - ./zbx_env/var/lib/zabbix/modules:/var/lib/zabbix/modules:ro
      - ./zbx_env/var/lib/zabbix/enc:/var/lib/zabbix/enc:ro
      - ./zbx_env/var/lib/zabbix/ssh_keys:/var/lib/zabbix/ssh_keys:ro
    env_file:
      - ./env_vars/.env_agent
    privileged: true
    pid: "host"
    networks:
      zbx_net_backend:
        aliases:
          - zabbix-agent
          - zabbix-agent-passive
          - zabbix-agent-alpine
    stop_grace_period: 5s
    labels:
      com.zabbix.description: "Zabbix agent"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "zabbix-agentd"
      com.zabbix.os: "alpine"
  mysql-server:
    container_name: mysql-server
    image: mysql:8.0
    restart: always
    security_opt:
      - seccomp:unconfined
    ports:
      - "3316:3306"
    command:
      - mysqld
      - --character-set-client=utf8mb4
      - --character-set-connection=utf8mb4
      - --character-set-results=utf8mb4
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_bin
      - --default-authentication-plugin=mysql_native_password
    #   - --require-secure-transport
    #   - --ssl-ca=/run/secrets/root-ca.pem
    #   - --ssl-cert=/run/secrets/server-cert.pem
    #   - --ssl-key=/run/secrets/server-key.pem
    volumes:
      - ./zbx_env/var/lib/mysql:/var/lib/mysql:rw
    env_file:
      - ./env_vars/.env_db_mysql
    secrets:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_ROOT_PASSWORD
    #   - server-key.pem
    #   - server-cert.pem
    #   - root-ca.pem
    stop_grace_period: 1m
    networks:
      zbx_net_backend:
        aliases:
          - mysql-server
          - m-database
          - mysql-database
  db_data_mysql:
    image: busybox
    volumes:
      - ./zbx_env/var/lib/mysql:/var/lib/mysql:rw

networks:
  zbx_net_frontend:
    driver: bridge
    driver_opts:
      com.docker.network.enable_ipv6: "false"
    ipam:
      driver: default
      config:
        - subnet: 172.16.238.0/24
  zbx_net_backend:
    driver: bridge
    driver_opts:
      com.docker.network.enable_ipv6: "false"
    internal: true
    ipam:
      driver: default
      config:
        - subnet: 172.16.239.0/24
secrets:
  MYSQL_USER:
    file: ./env_vars/.MYSQL_USER
  MYSQL_PASSWORD:
    file: ./env_vars/.MYSQL_PASSWORD
  MYSQL_ROOT_PASSWORD:
    file: ./env_vars/.MYSQL_ROOT_PASSWORD

#  client-key.pem:
#    file: ./env_vars/.ZBX_DB_KEY_FILE
#  client-cert.pem:
#    file: ./env_vars/.ZBX_DB_CERT_FILE
#  root-ca.pem:
#    file: ./env_vars/.ZBX_DB_CA_FILE
#  server-cert.pem:
#    file: ./env_vars/.DB_CERT_FILE
#  server-key.pem:
#    file: ./env_vars/.DB_KEY_FILE

```

其中有基础更改，主要是使用的镜像非zabbix镜像。数据库考虑暴露3316端口（但是这里没有成功）
使用非官方容器的目的是遇到一些情况。

### zabbix 图字体口口口
这个问题之前有写过，可以查看之前那篇[《解决zabbix5字体中文口口乱码》](https://blog.csdn.net/u011105410/article/details/118364266)。主要就是zabbix提供的字体**DejaVuSans.ttf** 不支持中文。通过无版权字体替换此字体。生成自己的镜像。

### 数据库字符集问题
![iShot2022-01-0700.13.16](https://img-blog.csdnimg.cn/img_convert/a90d61cd41027a2fc827c981cf1f8af7.png)
在测试中，我发现我有一些item显示的中文是????，开始以为字体问题，经过多方查询，发现这个问题是字符集问题。 也就是数据库这块的配置。具体大家自行科普，我这里主要就是设置了mysql8中字符集的配置，默认改成utf8mb4。
在官方的mysql镜像中，可以配置两处字符集，但无法设置全面。导致中文出现????。

结束～

祝好。
