---
title: php laravel使用docker部署
date: 2018-04-21T20:01:46.000Z
description: php laravel使用docker部署
keywords: php linux docker
tags:
  - php
  - linux
  - linux
categories:
  - blog
collections:
  - blog

---

### 前言

使用的云服务器到期了，买的时候打折的力度的比较大，但是续费的时候就非常昂贵了，只得用新用户换一家厂商的云服务器。但是可怕的问题就来了，我又得重新搭建一次开发环境。每次搭建php的开发环境都十分麻烦。为了下一次被迫迁移方便一点，我果断决定用docker来部署我的小站点，也方便下一次迁移，所以特意记录一下我的操作流程，其中还是有一些需要注意的问题。我就遇到了一些奇怪的问题。

### 操作步骤：

1.制作镜像。

nginx和mysql都直接使用官方镜像就好，不需要特殊操作。php可能需要安装一些第三方php扩展，我使用的是laravel框架，因此需要一些php扩展。其它框架，可能需要根据情况，安装一些其它扩展。php的Dockerfile如下

```
FROM php:7.2-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd &&  docker-php-ext-install pdo_mysql \
    && docker-php-ext-install zip
```
其它扩展安装参考官方说明https://hub.docker.com/_/php/

2.配置docker-compose文件

我是用的docker-compose去启动整个镜像，docker-compose.yml文件如下
```
version: '2.2'

services:

  nginx:
    image: nginx:1.15.2-alpine
    hostname: nginx
    container_name: nginx
    volumes:
      - /var/www:/var/www
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - 80:80
      - 443:443

  php:
    image: registry.cn-hangzhou.aliyuncs.com/fzy/php7.2
    hostname: php
    container_name: php-7.2
    volumes:
      - /var/www:/var/www
    ports:
      - 9000:9000

  mysql:
    image: mysql:5.6
    environment:
      MYSQL_ROOT_PASSWORD: 520520fzy
    volumes:
      - /var/mysql-data:/var/lib/mysql
    ports:
      - 3306:3306
```
这里需要注意的是，nginx和php都需要映射到外界目录。

3.修改nginx.conf文件

将php迁移到docker部署，nginx.conf的配置文件跟我们平常是使用的还是有些不太一样。

```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  test1.sleepan.com;
        root         /var/www/html;
        index        index.php index.html index.htm;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
           try_files $uri $uri/ =404;
           client_max_body_size  500m;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
          location = /50x.html {
       }
	location ~ \.php$ {

          fastcgi_index   index.php;
          fastcgi_pass    php:9000;
          fastcgi_param   SCRIPT_FILENAME    /var/www/html$fastcgi_script_name;
          fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
          include         fastcgi_params;
          client_max_body_size  500m;
       }
    }
}
```

```
fastcgi_pass    php:9000;
```
这一行，原本我们在物理机的时候，是写成 127.0.0.1:9000，但是启用docker后，php和nginx有各自的网络地址，因此这里要改为php。因为我们使用了docker-compose部署，这里的php指向的就是我们部署php所在docker的地址。

4.启动

需要在docker-compose.yml的同级目录下，执行docker-compose up -d这个命令，去启动容器。关闭容器可用命令docker-compose down命令。