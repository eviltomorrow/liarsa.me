---
title: 《Kubernetes in action》（三）
catalog: true
date: 2023-08-10 10:00:00
subtitle: Pod：运行于 Kubernetes 中的容器
tags:
    - 《Kubernetes in Action》
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# Pod：运行于 Kubernetes 中的容器

Pod 是 Kubernetes 中最为重要的核心概念，而其他对象仅仅是在管理、暴露 Pod 或被 Pod 使用。

## 1. 介绍 Pod

通常情况下，一个 Pod 只包含一个容器，当一个 Pod 包含多个容器时，这些容器总是运行在同一个工作节点上。

 ![一个 Pod 中的所有容器只能运行在一个节点上](/img/article/kubernetes/pod_node_deploy.png)

容器被设计为每个容器只运行一个进程（除非进程本身产生子进程）。否则，管理他们的运行，日志等将是我们的责任。由于不能将多个进程聚集在一个单独的容器中，我们需要另一种更高级的结构来将容器绑定在一起，并将他们作为一个单元进行管理，这就是 Pod 背后的根本原理。

同一个 Pod 中的所有容器共享以下 Linux 命名空间：

 - Network
 - UTS
 - IPC

所以，他们共享相同的主机名和网络接口（共享相同的 IP 地址和端口空间），能通过 IPC 进行通信。

Kubernetes 集群中的所有 Pod 都在同一个共享网络地址空间中，这就意味着每个 Pod 都可以通过其他 Pod 的 IP 地址来实现相互访问。而不需要 NAT 网关。

![Pod 平面网络](/img/article/kubernetes/pod_platform_network.png)


<hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://developer.aliyun.com/article/745468" style="font-weight: bold;">[Kubernetes必备知识： Pod网络模型]</a></li>
</ul>