---
title: 《Kubernetes in action》（一）
catalog: true
date: 2023-08-09 10:00:00
subtitle: Kubernetes 介绍
tags:
    - 《Kubernetes in Action》
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# Kubernetes 介绍

Kubernetes 抽象了数据中心的硬件基础设施，使得对外暴露的只是一个巨大的资源池。Kubernetes 帮助企业标准化了云端部署及内部部署的应用交付方式。

# 1.Kubernetes 系统的需求

 - 从单体应用到微服务
 - 为应用程序提供一个一致的环境
 - 迈向持续交付--DevOps 和无运维

# 2.介绍容器技术

## 2.1 什么是容器

Linux 容器技术允许你在同一台机器上运行多个服务，不仅提供不同的环境给每个服务，而且将他们互相隔离。  

对比虚拟机，容器更加轻量，省去了虚拟机的额外开销。但也自然带来了安全隐患，多个容器共享一个底层内核。  

容器实现隔离机制介绍：

- 第一个是 Linux 命名空间（namespace），它使每个进程只看到它自己的系统视图（文件、进程、网络接口、主机名等）
    - Mount（mnt）
    - Process ID（pid）
    - Network（net）
    - Inter-process communication（ipc）
    - UTS
    - User ID（user）

- 第二个是 Linux 控制组（cgroups），它限制了进程能使用的资源量（CPU、内存、网络宽带等）。
    - 内核功能，限制一个进程或一组进程资源使用

## 2.2 Docker 容器平台介绍

Docker 是一个打包、分发和运行应用程序的平台。

 - 镜像：Docker 镜像里包含了你打包的应用程序极其所依赖的环境。
 - 镜像仓库：Docker 镜像仓库用于存放 Docker 镜像，以便共享这些镜像。
 - 容器：基于 Docker 镜像创建的 Linux 容器。

容器和虚拟机对比  

 ![容器和虚拟机对比](/img/article/kubernetes/vm_docker_description.png)

# 3.Kubernetes 介绍

## 3.1 Kubernetes 的核心功能

Kubernetes是一个容器调度管理系统，整个系统由一个主节点和若干个工作节点组成。

 ![Kubernetes 系统图](/img/article/kubernetes/kubernetes_deployment_platform.png)

 - 帮助开发者聚焦核心应用功能

 - 帮助运维团队获取更高的资源利用率

## 3.2 Kubernetes 集群架构

从硬件级别，一个 Kubernetes 集群由很多节点组成。

 - 主节点：承载 Kubernetes 控制和管理整个集群系统的控制面板

 - 工作节点：运行用户实际部署的应用

 ![Kubernetes 组件架构图](/img/article/kubernetes/kubernetes_architecture.png)

<b>控制面板</b>

控制面板用于控制集群并使它工作。

 - Kubernetes API 服务器：负责各组件的通信
 - Scheduler：负责调度应用
 - Controller Manager：负责集群级别的功能（复制组件、跟踪工作节点，处理节点失败等）
 - Etcd：负责持久化存储集群配置

<b>工作节点</b>

工作节点是运行容器化应用的机器。

 - Docker、rtk 或其他容器类型
 - Kubelet 与 API 服务器通信，管理它所在节点的容器
 - Kubernetes Service Proxy（kube-proxy）负责组件之间的负载均衡网络流量

## 3.4 在 Kubernetes 中运行应用

为了在 Kubernetes 中运行应用，首先需要将应用打包进一个或多个容器镜像。再将镜像推送到镜像仓库，然后将应用的描述发布到 Kubernetes API 服务器。

 ![Kubernetes 运行应用](/img/article/kubernetes/kubernetes_run_application.png)

## 3.5 使用 Kubernetes 的好处

 - 简化应用程序部署
 - 更好地利用硬件
 - 健康检查和自修复
 - 自动扩容
 - 简化应用部署