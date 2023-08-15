---
title: Linux 系统安装 MySQL 数据库
catalog: true
date: 2023-08-08 15:00:00
subtitle: 
tags:
- 数据库
- Linux
- MySQL
categories:
- 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/mysql.jpg
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# Linux 系统安装 MySQL 数据库

## 准备

- Linux 系统
- MySQL 安装包

## 步骤

### 1. 下载 MySQL 安装包（mysql-8.0.xx-linux-glibc2.12-x86_64.tar.xz）

```html
https://dev.mysql.com/downloads/mysql/
```

### 2. 将 MySQL 安装包解压到 /usr/local/app 目录

```shell
$ mkdir -p /usr/local/app
$ cd /usr/local/app
# 解压 tar -zxvf mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz
$ mv mysql-8.0.28-linux-glibc2.12-x86_64/ mysql
$ pwd
/usr/local/app
$ ll
drwxr-xr-x 9 root root 4096 3月  10 13:23 mysql
```

### 3. 创建 MySQL 用户组和用户

```shell
$ groupadd mysql
$ useradd -r -g mysql mysql
```

### 4. 准备基础目录

```shell
$ cd /usr/local/app/mysql
$ mkdir data tmp log run
```

### 5. 创建 my.cnf 文件

```shell
$ touch my.cnf
[client]
port = 3306
socket = /usr/local/app/mysql/run/mysql.socket
default-character-set = utf8

[mysqld]
pid-file = /usr/local/app/mysql/run/mysql.pid
user = mysql
server-id = 1
port = 3306
character-set-server = utf8mb4
authentication_policy = mysql_native_password

basedir = /usr/local/app/mysql
datadir = /usr/local/app/mysql/data
tmpdir = /usr/local/app/mysql/tmp
socket = /usr/local/app/mysql/run/mysql.socket
log_error = /usr/local/app/mysql/log/error.log
sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION,PIPES_AS_CONCAT,ANSI_QUOTES'
```

### 6. 修改目录权限

```shell
$ chown -R mysql .
$ chgrp -R mysql .
```

### 7. 初始化数据库

```shell
$ cd /usr/local/app/mysql/bin
$ ./mysqld --defaults-file="../my.cnf" --initialize
```

### 8. 记录 mysql 初始化密码

```shell
$ cat /usr/local/app/mysql/log/error.log
2022-03-10T05:40:52.646435Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2022-03-10T05:40:52.646481Z 0 [Warning] [MY-010918] [Server] 'default_authentication_plugin' is deprecated and will be removed in a future release. Please use authentication_policy instead.
2022-03-10T05:40:52.646495Z 0 [System] [MY-013169] [Server] /usr/local/app/mysql/bin/mysqld (mysqld 8.0.28) initializing of server in progress as process 6445
2022-03-10T05:40:52.658096Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-03-10T05:40:53.444036Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-03-10T05:40:55.768104Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: p/tq4LHa-zV+
# 最后一行有密码： p/tq4LHa-zV+， 修改 root 密码时使用。
```

### 9. 收回目录权限

```shell
$ cd /usr/local/app/mysql
$ chown -R root .
$ chgrp -R root .
$ chown -R mysql data/ log/ run/ tmp/ my.cnf
$ chgrp -R mysql data/ log/ run/ tmp/ my.cnf
```

### 10. 复制配置文件

```shell
$ cp /usr/local/app/mysql/my.cnf  /etc/my.cnf
$ cp /usr/local/app/mysql/support-files/mysql.server /etc/init.d/mysql
$ chmod a+x /etc/init.d/mysql
$ vim /etc/init.d/mysql 
# 修改如下两行
  basedir="/usr/local/app/mysql"
  datadir="/usr/local/app/mysql/data"
```

### 11. 启动 MySQL，并修改密码

```shell
$ service mysql start
Starting MySQL.. SUCCESS!
$ ln -s /usr/local/app/mysql/bin/mysql /usr/local/bin/mysql
$ mysql -u root -p
# 如果出现以下错误，请执行 sudo ln -s /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5
mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
```

### 12. 修改 root 密码

```shell
$ alter user 'root'@'localhost' identified by 'root';
```

