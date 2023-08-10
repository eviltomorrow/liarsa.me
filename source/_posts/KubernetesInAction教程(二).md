---
title: 《Kubernetes in action》（二）
catalog: true
date: 2023-08-09 15:00:00
subtitle: 开始使用 Kubernetes 和 Docker
tags:
    - 《Kubernetes in Action》
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# 开始使用 Kubernetes 和 Docker

## 1. 运行 Hello world 容器

```shell
$ docker run busybox echo "Hello world"
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
3f4d90098f5b: Pull complete 
Digest: sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
Status: Downloaded newer image for busybox:latest
Hello world
```

## 2. 查看集群是否正常工作
```shell
$ kubectl cluster-info 
Kubernetes control plane is running at https://192.168.133.100:6443
CoreDNS is running at https://192.168.133.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```