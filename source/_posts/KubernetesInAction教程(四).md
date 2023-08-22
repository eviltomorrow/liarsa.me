---
title: 《Kubernetes in action》（四）
catalog: true
date: 2023-08-15 09:00:00
subtitle: 副本机制和其他控制器：部署托管的 pod
tags:
    - 《Kubernetes in Action》
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# 副本机制和其他控制器：部署托管的 pod

## 4.1. 保持 pod 健康

Kubernetes 可以通过存活探针（liveness probe）检查容器是否还在运行。可以为 pod 中的每个容器单独指定存活探针。如果探测失败，Kubernetes 将定期执行探针并重启容器。

 Kubernetes 包括三种探针：

 - HTTP GET 探针

<hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://developer.aliyun.com/article/745468" style="font-weight: bold;">[Kubernetes必备知识： pod网络模型]</a></li>
</ul>