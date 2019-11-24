## Kubernetes 基本概念

### Master 和 Node

Master：主节点，负责管理整个集群。Master 一般不用来运行应用程序实例。

Node：工作节点，负责运行应用程序实例。一个虚拟机或物理机就是一个节点，我们可以在一台物理机上创建多个虚拟机，每个虚拟机作为一个 Node；也可以将每台物理机作为一个 Node。每个工作节点都会有一个 Kubelet 代理和 Docker 环境，前者负责和 Master 通信并管理工作节点，后者是运行 Docker 容器的必要环境。

![img](面试题 - Docker.assets/module_01_cluster.svg) 

参考：https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

---

### Deployment

Deployment：部署。创建或更新应用程序实例。Deployment 是交给 Master 来运行，由 Master 根据 Deployment 信息来控制 Node 去创建和更新应用程序实例及实例个数。

参考：https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/

 ![img](面试题 - Docker.assets/module_02_first_app.svg) 

---

### Pod 和 Node

Pod：是 K8s 在容器的基础上抽象出来的概念，由一个或多个容器、共享存储（卷）和IP地址组成。Pod 是 K8s 的原子单元，K8s是通过Pod来创建、运行和管理容器。

 ![img](面试题 - Docker.assets/module_03_pods.svg) 

Pod 是运行在 Node 上，默认只能在集群内访问，对外不暴露IP地址，宿主机也访问不了。但是可以通过 kubectl proxy 命令创建代理来转发请求或通过 Service 来开启外部访问。

 ![img](面试题 - Docker.assets/module_03_nodes.svg)

参考：https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/ 

---

### Service

Service：是 K8s 在 Pod 的基础上抽象出来的概念，由一个或多个 Pod 组成，允许外部流量访问，并提供负载均衡和服务发现。

 ![img](面试题 - Docker.assets/module_04_services.svg)

Service 可以通过 Pod 上的 Label（标签）来对 Pod 进行分组：

 ![img](面试题 - Docker.assets/module_04_labels.svg) 

参考：https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

### 服务伸缩和滚动更新

服务伸缩：https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/ 

滚动更新：https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/ 

