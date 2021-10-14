---
title: "Centos中部署Laravel项目配置"
date: 2021-10-13T11:37:06+08:00
draft: false
tags: ["服务器","部署"]
categories: ["服务器"]
---


# 安装以及配置Nginx服务器

## 安装

```shell
sudo dnf install nginx
```

## 启动
在安装完之后，使用下面两条语句来启动Nginx

```shell
$ sudo systemctl enable nginx
$ sudo systemctl start  nginx
```

## 监测

运行如下的命令来监测Nginx是否正常运行

```shell
$ curl  localhost:80
```

## 管理

当然，我们也可以通过如下的命令来管理Nginx服务

```shell
$ sudo systemctl stop nginx  #停止服务
$ sudo systemctl start nginx #开启服务
$ sudo systemctl restart nginx #重启服务
$ sudo systemctl reload nginx #重新加载配置
$ sudo systemctl disable nginx #禁用nginx开机自启动
$ sudo systemctl enable nginx #开启nginx开机自启动
```

## 配置
Nginx的配置文件位于`/etc/nginx`文件夹下面

- `/etc/nginx/nginx.conf`该文件是主配置文件
- `/etc/nginx/conf.d`该文件夹是我们项目的配置文件夹

Nginx的项目文件夹位于`/usr/share/nginx/html`目录下，
我们可以把项目放置于该目录下

## Laravel项目的简单配置
关于Laravel项目的Nginx配置，你可以查看[这个文档](https://laravel.com/docs/8.x/deployment#nginx)

1. 设置好项目之后，需要为项目设置特定的权限

   ```shell
   chmod -R 777 storage
   chmod -R 777 bootstrap/cache
   ```
1. 设置项目的拥有者为nginx
   ```shell
   chown nginx:nginx -R /usr/share/nginx/your_project
   ```





# 安装以及配置MariaDB

## 安装
```
sudo dnf install mariadb-server
```
## 启动
安装完成之后，我们可以通过如下的语句启动我们的数据库服务

```
sudo systemctl start mariadb
```

## 监测
通过运行如下命令监测数据库服务是否启动

```shell
sudo systemctl status mariadb
```

## 管理
当然，我们也可以通过如下的命令来管理Nginx服务

```shell
$ sudo systemctl stop mariadb  #停止服务
$ sudo systemctl start mariadb #开启服务
$ sudo systemctl restart mariadb #重启服务
$ sudo systemctl disable mariadb #禁用mariadb开机自启动
$ sudo systemctl enable mariadb #开启mariadb开机自启动
```

## 配置
运行如下命令进行安全配置

```shell
sudo mysql_secure_installation
```

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

# 安装以及配置PHP7.4
由于系统自带的php版本比较老，所以我们要添加新的版本

1. 添加并使用Remi仓库
   ```
   dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
   ```

1. 列出仓库中的PHP版本
   ```shell
   dnf module list php
   ```
   
1. 选择自己需要安装的版本
    ```shell
    dnf module enable php:remi-7.4
    ```

1. 安装php以及自己需要的php扩展
   ```
   dnf install php php-cli php-common ...
   ```

1. 检查安装的php版本
   ```
   php -v
   ```

1. 检查已经安装的php扩展
   ```
   php -m 
   ```

## 配置使用php-fpm

1. 安装php-fpm扩展
   ```
   dnf install php-fpm
   ```

1. 启动php-fpm
   ```
   systemctl start php-fpm
   ```

1. 修改php配置文件
   ```
   vim /etc/php-fpm.d/www.con
   ```
   
   将其中的部分配置修改如下：
   ```
   listen = /run/php-fpm/www.sock
   listen.owner = nginx
   listen.group = nginx
   listen.mode  = 066
   ```

1. 修改php.ini文件
   ```
   vim /etc/php.ini
   ```
   将其中的部分参数修改如下：
   ```
   date.timezone = Asia/Shanghai
   cgi.fix_pathinfo=0
   ```

1. 修改www.conf文件
   ```
   vim /etc/php-fpm.d/www.conf
   ```
   取消下面的注释
   ```
   security.time_extensions = .php .php3 .php4 .php5 .php7
   ```

   

# 安装Git

## 安装

```shell
yum install git
```

## 配置

```shell
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"
```
## 生成ssh key

```shell
ssh-keygen -o
```
将生成的key添加到你的代码仓库账号，就可以直接拉取代码了

# 安装Composer
1. 下载composer
   ```
   curl -sS https://getcomposer.org/installer | php
   ```

1. 移动到用户目录
   ```
   mv composer.phar /usr/local/bin/composer
   ```

1. 添加权限
   ```
   chmod +x /usr/local/bin/composer
   ```



