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


<hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://developer.aliyun.com/article/745468" style="font-weight: bold;">[Kubernetes必备知识： pod网络模型]</a></li>
</ul>