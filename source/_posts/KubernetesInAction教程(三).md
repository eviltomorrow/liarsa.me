---
title: 《Kubernetes in action》（三）
catalog: true
date: 2023-08-10 10:00:00
subtitle: pod：运行于 Kubernetes 中的容器
tags:
    - 《Kubernetes in Action》
    - 容器
    - Kubernetes
categories:
    - 技术
index_img: https://liarsa-me.oss-cn-beijing.aliyuncs.com/img/logo/kubernetes.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---

# pod：运行于 Kubernetes 中的容器

pod 是 Kubernetes 中最为重要的核心概念，而其他对象仅仅是在管理、暴露 pod 或被 pod 使用。

## 1. 介绍 pod

通常情况下，一个 pod 只包含一个容器，当一个 pod 包含多个容器时，这些容器总是运行在同一个工作节点上。

 ![一个 pod 中的所有容器只能运行在一个节点上](/img/illustration/kubernetes/pod_node_deploy.png)

容器被设计为每个容器只运行一个进程（除非进程本身产生子进程）。否则，管理他们的运行，日志等将是我们的责任。由于不能将多个进程聚集在一个单独的容器中，我们需要另一种更高级的结构来将容器绑定在一起，并将他们作为一个单元进行管理，这就是 pod 背后的根本原理。

同一个 pod 中的所有容器共享以下 Linux 命名空间：

 - Network
 - UTS
 - IPC

所以，他们共享相同的主机名和网络接口（共享相同的 IP 地址和端口空间），能通过 IPC 进行通信。

Kubernetes 集群中的所有 pod 都在同一个共享网络地址空间中，这就意味着每个 pod 都可以通过其他 pod 的 IP 地址来实现相互访问。而不需要 NAT 网关。

![pod 平面网络](/img/illustration/kubernetes/pod_platform_network.png)

## 2. 以 YAML 或 JSON 描述文件创建 pod

### 2.1. pod 定义的组成部分：

 - apiVersion: Kubernetes API 版本。
 - kind: 描述的资源类型。
 - metadata: 包括名称、命名空间、标签和关于该容器的其他信息。
 - spec: 包含 pod 内容的实际说明。
 - status: 包括运行中的 pod 的当前信息。

### 2.2. 创建一个简单的 YAML 描述文件（kubia-manual.yaml）

```yaml
apiVersion: v1
kind: pod
metadata:
  name: kubia-manual
spec:
  containers:
    - image: luksa/kubia
      name: kubia
      ports:
        - containerPort: 8080
          protocol: TCP
```

### 2.3. 使用 kubectl create 来创建 pod

```sh
# 创建 pod
$ kubectl create -f kubia-manaul.yaml

# 查看 pod 列表
$ kubectl get pods
```

### 2.4. 查看应用程序日志

```sh
$ kubectl logs -f ${pod_name}
```

### 2.5. 向 pod 发送请求

```sh
$ kubectl port-forward ${pod_name} 8888:8080
```

## 3. 使用标签组织 pod

通过标签来组织 pod 和所有其他 kubernetes 对象。

### 3.1. 介绍标签

标签是可以附加到资源的任意键值对，用以选择具有该确切标签的资源。只要标签的 key 在资源内是唯一的，一个资源便可以拥有多个标签。

### 3.2. 创建 pod 时指定标签

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
spec:
  containers:
    - image: luksa/kubia
      name: kubia
      ports:
        - containerPort: 8080
          protocol: TCP
```

查看标签
```sh
$ kubectl get pods --show-labels
```

过滤标签
```sh
$ kubectl get pod -L ${label_name},${label_name}
```

### 3.3. 修改现有 pod 标签

添加标签
```sh
$ kubectl label pod kubia-manaul ${label_name}=${label_value}
```

修改现有标签
```sh
$ kubectl label pod kubia-manaul-v2 ${label_name}=${label_value} --overwrite
```

删除现有标签（注意后面的减号-）
```sh
$ kubectl label pod ${label_name}-
```

### 3.4. 通过标签选择器列出 pod 子集

标签选择器允许我们选择标记有特定标签的 pod 子集，并对这些 pod 进行操作。可以说标签选择器是一种能够根据是否包含有特定值的特定标签来过滤资源的准则。

 - 包含（或不包含）使用特定键的标签
 - 包含具有特定键和值的标签
 - 包含有特定键的标签，但其值与我们指定的不同

使用标签选择器
```sh
$ kubectl get pod -l '${label_name}=${label_value},...'

$ kubectl get pod -l '${label_name},...'

$ kubectl get pod -l '!${label_name},...'  # 单引号是必须，放置 bash shell 解析 !

$ kubectl get pod -l '${label_name}!=${label_value},...'

$ kubectl get pod -l '${label_name} in (${label_value1}, ${label_value2}, ...),...'

$ kubectl get pod -l '${label_name} notin (${label_value1}, ${label_value2}, ...),...'
```

### 3.5. 使用标签分类工作节点

将 pod 调度到特定接点
```sh
$ kubectl label node ${node_name} gpu=true
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
    - image: luksa/kubia
      name: kubia
      ports:
        - containerPort: 8080
          protocol: TCP
```
注意：标签选择器应考虑符合特定标准的逻辑节点组。

### 3.6. 注解 pod

注解是键值对。可以容纳更多信息，主要用于工具使用，提供对象说明。

添加或修改注解

```sh
$ kubectl annotate pod ${pod_name} ${annotation_name}=${annotation_value}
```

### 3.7. 使用命名空间对资源进行分组

Kubernetes 命名空间简单地为对象名称提供一个作用域。

创建命名空间

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
```

```sh
$ kubectl create namespace ${namespace_name}

$ kubectl create -f kubia-manaul.yaml -n ${namespace_name}
```

### 3.8. 停止和移除 pod

```sh
$ kubectl delete pod ${pod_name}
 
$ kubectl delete pod -l ${label_name}=${label_value}

$ kubectl delete ns ${namespace_name} # 删除整个命名空间

$ kubectl delete pod --all # 删除当前命名空间中的所有 pod,

$ kubectl delete all --all # 删除当前命名空间中几乎所有的对象（派出 secret）
```



<hr/>
<b>参考：</b>
<ul>
    <li>[1] <a href="https://developer.aliyun.com/article/745468" style="font-weight: bold;">[Kubernetes必备知识： pod网络模型]</a></li>
</ul>