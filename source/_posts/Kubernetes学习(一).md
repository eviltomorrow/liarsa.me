---
title: Kubernetes 学习（一）
catalog: true
date: 2023-08-09 10:00:00
subtitle: 《Kubernetes in action》笔记（一）
tags:
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
 - 迈向持续交付：DevOps 和无运维

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

 ![容器和虚拟机对比](/img/article/kubernetes/vm-vs-docker.png)