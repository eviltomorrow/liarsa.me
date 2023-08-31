---
title: Kubernetes 设计与架构
catalog: true
date: 2023-08-31 07:00:00
subtitle: 
tags:
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes-observability.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# Kubernetes 设计与架构

## 1. Kubernetes 核心设计与架构

Kubernetes 由 Master 和 Node 两种节点构成。这两种角色分别对应控制节点和计算节点。其中，控制节点，即 Master 节点，由 3 个紧密协作的独立组建组合而成：

 - 负责 API 服务的 kube-apiserver
 - 负责调度的 kube-scheduler
 - 负责容器编排的 kube-controller-manager

整个集群的持久化数据，则由 kube-apiserver 处理后保存在 etcd 中。

计算节点上最核心的部分，是一个名为 kubelet 的组件。kubelet 主要负责同容器运行时交互，而这种交互所依赖的是一个称作 CRI（container runtime interface）的远程调用接口，该接口定义了容器运行时的各项核心操作。

kubelet 还通过 grpc 协议同一个叫做 Device Plugin 的插件进行交互。这个插件，是 Kubernetes 项目用来管理 GPU 等宿主机无力设备的主要组件。

kubelet 的另一个重要功能，是调用网络插件（CNI，container networking interface）和存储插件（CSI，container storage interface）为容器配置网络和持久化存储。

 ![Kubernetes 项目的全剧架构](/img/illustration/kubernetes/kubernetes_design1.png)


## 2. Kubernetes 核心能力与项目定位

Kubernetes 项目最主要的设计思想就是，以统一的方式抽象底层基础设施能力（比如计算、存储、网络），定义任务编排的各种关系（比如亲密关系、访问关系、代理关系），将这些抽象以声明式 API 的方式对外暴露，从而运行平台构建者基于这些抽象进一步构建自己的 PaaS 乃至任何上层平台。

 ![Kubernetes 项目核心功能“全景图”](/img/illustration/kubernetes/kubernetes_progress.png)

<hr/>
<b>参考：</b>
<ul>
    <li>[1] 张磊.深入剖析 Kubernetes[M].北京:人民邮电出版社，2021.3:17</li>
</ul>