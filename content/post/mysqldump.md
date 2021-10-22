---
title: "使用Mysqldump，crontab 执行数据库定时备份"
date: 2021-10-22T20:31:55+08:00
draft: false
tags: ["服务器","部署","maraidb"]
categories: ["服务器"]
---

# mariadb-dump指令

在MariaDB 10.4.6中，可以使用mariadb-dump代替`mysqldump`,
`mysqldump`提供了非常多的可选参数来进行管理。完整的参数可以参考[mysqldump文档](https://mariadb.com/kb/en/mysqldump/)。

一个常用的备份语句如下：
```shell
mysqldump [options] db_name [tbl_name ...]
```

# 备份步骤拆解
对于数据备份来说，我们将其拆解为三个步骤

1. 连接数据库: 对于数据库的连接而言，我们需要提供数据库地址，用户名，密码三个参数，三个参数对应的指令分别是`--host`,`-u`,`-p`
   ```
   mysqldump --host=localhost -utest -ptext
   ```

2. 指定需要备份的数据,除了可以备份数据哭外，还可以备份指定的数据库`--databases`，数据表`-T`,`--tables`
     ```
     mysqldump --databases=db1,db2

     mysqldump --tables=tb1,tb2
     ```

3. 指定保存的文件格式以及文件路径,默认情况下将保存为`.sql`的文件形式，如下：
   ```
   mysql options > /your_path/file_name
   ```
   在Linux操作系统中，我们可以通过管道操作符，将文件保存为我们指定的文件格式，比如`gz`格式的文件

完成示例
```
mysqldump -utest -ptest --host=localhost --databases=test1 > /data/backup/test1.sql
```

# shell脚本
我们不能总是手动去执行这条命令，因此我们可以考虑将其写成脚本的形式，然后自动执行

```bash
#!/bin/bash

#定义备份路径
BACKUP_PATH=/data/mariadb/mysql

CURRENT_TIME=$(date +%Y%m%d_%H%M%S)

[! -d "$BACKUP_PATH"] && madir -p "$BACKUP_PATH"

#定义数据库连接参数
HOST=localhost

DB_USER=test

DB_PW=pwd

#设定备份命令
DATABASE=test1

#设置文件名
FILE_GZ=${BACKUP_PATH}/$CURRENT_TIME.$DATABASE.sql.gz

# 执行命令
/usr/local/bin/mysqldump -u${DB_USER} -p${DB_PW} --host=$HOST -q -R --databases $DATABASE | gzip > $FILE_GZ

```

# 定时任务
给脚本赋予可执行权限

```shell
chmod u+x mysqlBack.sh
```

设置定时执行时间,具体的时间设置，可以参考这个[crontab](https://www.tutorialspoint.com/unix_commands/crontab.htm)使用文档
```
crontab -e
* * * * * mysqlBack.sh
```









