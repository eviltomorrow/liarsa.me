---
title: Linux 系统安装 Etcd 存储
catalog: true
date: 2023-08-08 16:00:00
subtitle: 
tags:
- 存储
- Linux
- Etcd
categories:
- 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/etcd.jpg
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# Linux 系统安装 Etcd 存储

## 准备

- CentOS 8.2 系统
- Etcd 安装包

## 步骤

### 1. 下载 Etcd 安装包

```shell
$ mkdir -p /usr/local/app/etcd
$ cd /usr/local/app/etcd
$ ETCD_VER=v3.5.2
$ GOOGLE_URL=https://storage.googleapis.com/etcd
$ GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
$ DOWNLOAD_URL=${GOOGLE_URL}
$ curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o etcd-${ETCD_VER}-linux-amd64.tar.gz
```

### 2. 解压 Etcd 安装包

```shell
$ tar -zxvf etcd-v3.5.2-linux-amd64.tar.gz
```

### 3. 创建 Etcd 用户组和用户

```shell
$ groupadd --system etcd
$ useradd -s /sbin/nologin --system -g etcd etcd
```

### 4. 安装和启动

```shell
$ cd etcd-v3.5.2-linux-amd64/
$ ln -s /usr/local/app/etcd/etcd-v3.5.2-linux-amd64/etcd /usr/local/bin/etcd
$ ln -s /usr/local/app/etcd/etcd-v3.5.2-linux-amd64/etcdctl /usr/local/bin/etcdctl
$ mkdir -p /var/lib/etcd/
$ mkdir /etc/etcd
$ chown -R etcd:etcd /var/lib/etcd/
$ vim /etc/systemd/system/etcd.service
## 粘贴以下内容
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment=ETCD_DATA_DIR=/var/lib/etcd
Environment=ETCD_NAME=%m
ExecStart=/usr/local/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target

$ systemctl daemon-reload
$ systemctl start etcd.service
```