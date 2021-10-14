---
title: "使用acme.sh申请certbot免费https证书"
date: 2021-10-13T23:20:07+08:00
draft: false
tags: ["服务器","部署","https"]
categories: ["服务器"]
---

1. 安装acme.sh到用户目录
   ```
    curl  https://get.acme.sh | sh -s email=my@example.com
   ```

1. 设置别名
   ```
   alias acme.sh=~/.acme.sh/acme.sh
   ```

1. 生成证书，由于我已经运行了一个项目，直接让读取nginx的配置文件来生成证书
   ```
   acme.sh --issue -d you_domain.com --nginx
   ```
   运行完成之后，会在acme.sh目录下面创建如下的文件
   ```
    Your cert is in: ~/.acme.sh/you_domain/you_domain.cer
    Your cert key is in: ~/.acme.sh/you_domain/you_domain.key
    The intermediate CA cert is in: ~/.acme.sh/you_domain/ca.cer
    And the full chain certs is there: ~/.acme.sh/you_domain/fullchain.cer
   ```

1. 安装证书
   ```
   acme.sh --install-cert -d you_domain \
   --key-file       /path/to/keyfile/in/nginx/key.pem  \
   --fullchain-file /path/to/fullchain/nginx/cert.pem \
   --reloadcmd     "service nginx force-reload"
   ```

1. 启用证书自动升级
   ```
   acme.sh --upgrade --auto-upgrade 0
   ```

1. 在Nginx中为项目启用https
   ```nginx
   server {
    listen 443 ssl;
    server_name your_domain;

    ssl_certificate /path/to/fullchain/nginx/cert.pem;
    ssl_certificate_key /path/to/keyfile/in/nginx/key.pem;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ....
   ```