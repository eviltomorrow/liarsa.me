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

 - HTTP GET 探针对容器的 IP 地址执行 HTTP GET 请求。（2xx,3xx 代表成功）
 - TCP 套接字探针舱室与容器指定端口建立 TCP 连接。
 - Exec 探针在容器内执行任意命令，并检查命令的退出状态码。（0 代表成功）

使用 http 存活探针

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
    - image: luksa/kubia-unhealthy
      name: kubia
      livenessProbe:
        httpGet:
          path: /
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 60
```

<b>提示：务必记得设置一个初始延迟来说明应用程序的启动时间。否则探针将在容器启动时立即开始探测容器，这通常会失败导致 Pod 不断重启。</b>

<font style="color: #4F94CD;">推出代码 137 表示进程被外部信号终止，退出代码 128+9（SIGKILL）、推出代码 143 对应 128+15（SIGTERM）。</font>

创建有效的存活探针

 - 存活探针应该检查什么
 - 保持探针轻量
 - 无须在探针中实现重试循环

4.2 了解 ReplicationController

ReplicationController 是一种 Kubernetes 资源，可确保它的 pod 始终保持运行状态。它的工作是确保 pod 的数量始终与其标签选择器匹配。

RepliactionController 的三部分：

  - label selector（标签选择器），用于确定 ReplicationController 作用域中有哪些 pod
  - replica count（副本个数），指定应运行的 pod 数量。
  - pod template（pod 模板），用于创建新的 pod 的副本。

只有变更副本数目的个数会影响现有 pod，更改标签选择器和 pod 模板对现有 pod 没有影响。

ReplicationController 的好处：

 - 确保一个 pod（或多个 pod 副本）持续运行，方法是现有 pod 丢失时启动一个新的 pod。
 - 集群节点发生故障时，它将为故障节点上运行的所有 pod（即受 ReplicationController 控制的节点上的那些 pod）创建替代副本。
 - 能轻松实现 pod 的水平伸缩。


<!-- <hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://developer.aliyun.com/article/745468" style="font-weight: bold;">[Kubernetes必备知识： pod网络模型]</a></li>
</ul> -->