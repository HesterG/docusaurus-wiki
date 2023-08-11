---
title: Kubernetes Review
date: 2020-02-23 10:01
tags:
  - Kubernetes
  - Tech Study
categories:
  - - Tech
    - Kubernetes
---

# Kubernetes Review

[Study Source](https://www.youtube.com/channel/UCdkGV51Nu0unDNT58bHt9bg)

![Quick General Intro](https://user-images.githubusercontent.com/17645053/75114786-11c41d80-5627-11ea-9e03-009bda02ad1b.png)

![Kubernetes](https://user-images.githubusercontent.com/17645053/75114532-b85aef00-5624-11ea-9a9a-3f2a70ed5728.png)

**Master**

**1 etcd** is a persistent, lightweight, distributed, key-value data store developed by [CoreOS](https://www.wikiwand.com/en/CoreOS) that reliably stores the configuration data of the cluster, representing the overall state of the cluster at any given point of time. Other components watch for changes to this store to bring themselves into the desired state

Hold all metadata?

**2 API server** is a key component and serves the Kubernetes [API](https://www.wikiwand.com/en/Application\_programming\_interface) using [JSON](https://www.wikiwand.com/en/JSON) over [HTTP](https://www.wikiwand.com/en/Hypertext\_Transfer\_Protocol), which provides both the internal and external interface to Kubernetes.\[[25\]](https://www.wikiwand.com/en/Kubernetes#citenotedointro25)\[[34\]](https://www.wikiwand.com/en/Kubernetes#citenote134) The API server processes and validates [REST](https://www.wikiwand.com/en/Representational\_state\_transfer) requests and updates state of the [API](https://www.wikiwand.com/en/Application\_programming\_interface) objects in [etcd](https://github.com/coreos/etcd), thereby allowing clients to configure workloads and containers across Worker nodes.

Commands?

**3 Scheduler** is the pluggable component that selects which node an unscheduled pod (the basic entity managed by the scheduler) should run on based on resource availability. Scheduler tracks resource utilization on each node to ensure that workload is not scheduled in excess of the available resources. For this purpose, the scheduler must know the resource requirements, resource availability and a variety of other user-provided constraints and policy directives such as quality-of-service, affinity/anti-affinity requirements, data locality and so on. In essence, the scheduler’s role is to match resource "supply" to workload "demand".

Detection?

**4 Controller manager** is the process in which the core Kubernetes controllers like DaemonSet Controller and Replication Controller run. The controllers communicate with the API server to create, update and delete the resources they manage (pods, service endpoints, etc.)

The one who gives the commands?

**Node/Worker/Minion**

**Kubelet** is responsible for the running state of each node, ensuring that all containers on the node are healthy. It takes care of starting, stopping, and maintaining application containers organized into pods as directed by the control plane.\[[25\]](https://www.wikiwand.com/en/Kubernetes#citenotedointro25)\[[36\]](https://www.wikiwand.com/en/Kubernetes#citenote36)

Kubelet monitors the state of a pod and if not in the desired state, the pod will be redeployed to the same node. The node status is relayed every few seconds via heartbeat messages to the master. Once the master detects a node failure, the Replication Controller observes this state change and launches pods on other healthy nodes.

**Container** resides inside a Pod. The container is the lowest level of a micro-service which holds the running application, the libraries and their dependencies. Containers can be exposed to the world through an external IP address.

**Kube-proxy** is an implementation of a [network proxy](https://www.wikiwand.com/en/Proxy\_server) and a [load balancer](https://www.wikiwand.com/en/Load\_balancing\_\(computing\)), and it supports the service abstraction along with other networking operation.\[[25\]](https://www.wikiwand.com/en/Kubernetes#citenotedointro25) It is responsible for routing traffic to the appropriate container based on IP and port number of the incoming request

* A [**reverse proxy**](https://www.nginx.com/resources/glossary/reverse-proxy-server) accepts a request from a client, forwards it to a server that can fulfill it, and returns the server’s response to the client.
* A [**load balancer**](https://www.nginx.com/resources/glossary/load-balancing) distributes incoming client requests among a group of servers, in each case returning the response from the selected server to the appropriate client.

**cAdvisor** is an agent that monitors and gathers resource usage and performance metrics such as CPU, memory, file and network usage of containers on each node.

![337641-20160121153024500-830400402](https://user-images.githubusercontent.com/17645053/75114543-cf014600-5624-11ea-970c-028f5543f3c2.png)

![337641-20160121153025562-961251045](https://user-images.githubusercontent.com/17645053/75114548-d6c0ea80-5624-11ea-9796-5343e0fcf4e4.png)

![337641-20160121153022422-1270227504](https://user-images.githubusercontent.com/17645053/75114561-f5bf7c80-5624-11ea-8266-550e7c3209d4.png)

在分布式系统中，部署，调度，伸缩一直是最为重要的也最为基础的功能。Kubernets 就是希望解决这一序列问题的。

https://yeasy.gitbooks.io/docker\_practice/content/kubernetes/quickstart.html

**快速上手**

目前，Kubenetes 支持在多种环境下的安装，包括本地主机（Fedora）、云服务（Google GAE、AWS 等）。然而最快速体验 Kubernetes 的方式显然是本地通过 Docker 的方式来启动相关进程。

下图展示了在单节点使用 Docker 快速部署一套 Kubernetes 的拓扑。

![k8s-singlenode-docker](https://user-images.githubusercontent.com/17645053/75114565-03750200-5625-11ea-84be-574cb0dc521c.png)

图 1.21.2.1 - 在 Docker 中启动 Kubernetes

**Kubernetes依赖Etcd服务来维护所有主节点的状态。**

**启动Etcd服务。**

docker run --net=host -d gcr.io/google\_containers/etcd:2.0.9 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data

**启动主节点**

启动kubelet。

```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock gcr.io/google_containers/hyperkube:v0.17.0 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests
```

**启动服务代理**

docker run -d --net=host --privileged gcr.io/google\_containers/hyperkube:v0.17.0 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2

**测试状态**

在本地访问 8080 端口，可以获取到如下的结果：

```bash
$ curl 127.0.0.1:8080
 {
  "paths": [
   "/api",
   "/api/v1beta1",
   "/api/v1beta2",
   "/api/v1beta3",
   "/healthz",
   "/healthz/ping",
   "/logs/",
   "/metrics",
   "/static/",
   "/swagger-ui/",
   "/swaggerapi/",
   "/validate",
   "/version"
  ]
 }
```

**查看服务**

所有服务启动后，查看本地实际运行的 Docker 容器，有如下几个。

```shell
CONTAINER ID    IMAGE                    COMMAND        CREATED       STATUS       PORTS        NAMES
 ee054db2516c    gcr.io/google_containers/hyperkube:v0.17.0  "/hyperkube schedule  2 days ago     Up 1 days                k8s_scheduler.509f29c9_k8s-master-127.0.0.1_default_9941e5170b4365bd4aa91f122ba0c061_e97037f5
 3b0f28de07a2    gcr.io/google_containers/hyperkube:v0.17.0  "/hyperkube apiserve  2 days ago     Up 1 days                k8s_apiserver.245e44fa_k8s-master-127.0.0.1_default_9941e5170b4365bd4aa91f122ba0c061_6ab5c23d
 2eaa44ecdd8e    gcr.io/google_containers/hyperkube:v0.17.0  "/hyperkube controll  2 days ago     Up 1 days                k8s_controller-manager.33f83d43_k8s-master-127.0.0.1_default_9941e5170b4365bd4aa91f122ba0c061_1a60106f
 30aa7163cbef    gcr.io/google_containers/hyperkube:v0.17.0  "/hyperkube proxy --  2 days ago     Up 1 days                jolly_davinci
 a2f282976d91    gcr.io/google_containers/pause:0.8.0     "/pause"        2 days ago     Up 2 days                k8s_POD.e4cc795_k8s-master-127.0.0.1_default_9941e5170b4365bd4aa91f122ba0c061_e8085b1f
 c060c52acc36    gcr.io/google_containers/hyperkube:v0.17.0  "/hyperkube kubelet  2 days ago     Up 1 days                serene_nobel
 cc3cd263c581    gcr.io/google_containers/etcd:2.0.9     "/usr/local/bin/etcd  2 days ago     Up 1 days                happy_turing
```

这些服务大概分为三类：主节点服务、工作节点服务和其它服务。

**主节点服务**

* apiserver 是整个系统的对外接口，提供 RESTful 方式供客户端和其它组件调用；
* scheduler 负责对资源进行调度，分配某个 pod 到某个节点上；
* controller-manager 负责管理控制器，包括 endpoint-controller（刷新服务和 pod 的关联信息）和 replication-controller（维护某个 pod 的复制为配置的数值）。

**工作节点服务**

* kubelet 是工作节点执行操作的 agent，负责具体的容器生命周期管理，根据从数据库中获取的信息来管理容器，并上报 pod 运行状态等；
* proxy 为 pod 上的服务提供访问的代理。

**其它服务**

* Etcd 是所有状态的存储数据库；
* gcr.io/google\_containers/pause:0.8.0 是 Kubernetes 启动后自动 pull 下来的测试镜像。
