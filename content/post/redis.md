---
title: "Centos7.8 安装Redis6"
date: 2021-10-21T10:53:42+08:00
draft: true
---

# 更新系统并安装所需的工具

```
yum update -y
yum install gcc make wget tcl
```

# 下载并编译安装

1. 下载并解压
   ```
   cd /usr/local/src
   wget https://github.com/redis/redis/archive/6.2-rc2.tar.gz
   tar xf 6.2-rc2.tar.gz
   cd redis-6.2-rc2
   ```
1. 编译安装
   ```
   make test
   make install
   ```

# 添加用户

```
groupadd redis
adduser --system -g redis --no-create-home redis
```

# 设置数据存放位置
```
mkdir -p /var/lib/redis
chown redis: /var/lib/redis
chmod 770 /var/lib/redis
```

# 创建配置文件

```
mkdir /etc/redis
cp /usr/local/src/redis-6.2-rc2/redis.conf /etc/redis/
```

# 修改配置
编辑`redis.conf`文件，修改如下配置
```
stop-writes-on-bgsave-error no

dir /var/lib/redis
```

编辑redis.conf文件，将下面的配置文件修改为
```
supervised auto  => supervised systemd
```

# 创建系统配置文件
在如下目录中添加项目配置文件，`/etc/systemd/system/redis.service`

修改redis系统文件,添加文件`redis.service`
```
[Unit]
[Unit]
Description=Redis Data Store
After=network.target
[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always
[Install]
WantedBy=multi-user.target
```

# Redis服务管理

```
systemctl enable redis
systemctl start redis
systemctl status redis
```

