# 安装 Kubernetes

## 前提条件

- 每台机器内存 2 GB 及以上。
- 每台机器 CPU 2 核心及以上。
- 集群中的节点不允许有重复的主机名、MAC 地址、product_uuid。
- 集群中的节点网络要互通。
- 集群中的节点时间要同步。
- 安装时建议禁用防火墙，否则需要开通相应端口。
- 禁用交换分区。

参考：

https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/



**设置主机名**

集群中节点的主机名不允许有重复，也不允许为 localhost，且需要确保主机名能 ping 通。

```bash
# 设置主机名
hostnamectl set-hostname k1
# 设置域名解析
echo "`hostname -I | awk '{print $1}'`    $(hostname)" >> /etc/hosts
# 验证
ping `hostname`
```

如果不设置主机名到本地 hosts 中，集群初始化时会报

> [WARNING Hostname]: hostname "k1" could not be reached
> [WARNING Hostname]: hostname "k1": lookup k1 on 114.114.114.114:53: no such host



**关闭防火墙**

建议先关闭所有节点的防火墙，安装成功后再开放相应的端口

```bash
systemctl disable firewalld.service --now
systemctl status firewalld.service
```

如果不关闭防火墙，集群初始化时会报

> [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly



**关闭交换分区**

集群中所有节点都要关闭交换分区，因为在交换分区开启时，无法准确计算 Pod 的内存利用率。

```bash
cp /etc/fstab /etc/fstab_bak
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

# 查看 Swap 是否为空
free -h
```

如果不关闭防火墙，集群初始化时会报

> [WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet

参考：

[New in Kubernetes v1.22: alpha support for using swap memory | Kubernetes](https://kubernetes.io/blog/2021/08/09/run-nodes-with-swap-alpha/)

[Swap Off - why is it necessary? - General Discussions - Discuss Kubernetes](https://discuss.kubernetes.io/t/swap-off-why-is-it-necessary/6879)

## 通过 Sealos 安装

安装前，需要确保 `~/.bashrc` 文件没有 `kubectl` 、`crictl` 命令。

```bash
# 下载并安装 sealos
curl -O https://ghproxy.com/https://github.com/labring/sealos/releases/download/v4.1.3/sealos_4.1.3_linux_amd64.tar.gz
tar zxvf sealos_4.1.3_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin
# 添加命令补全
echo "source <(sealos completion bash)" >> ~/.bashrc
# 安装集群（containerd 容器运行时）
# sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 labring/openebs:v3.3.0 \
     --masters 192.168.17.131 \
     --nodes 192.168.17.132 -p toor
# 安装集群（cri-docker 容器运行时）
sealos run labring/kubernetes-docker:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 labring/openebs:v3.3.0 \
     --masters 192.168.17.131 \
     --nodes 192.168.17.132 -p toor
# 安装其它
sealos run labring/ingress-nginx:4.1.0 labring/metrics-server:v0.6.1
# 卸载 metrics-server
kubectl delete -f /var/lib/sealos/data/default/rootfs/manifests/metrics-server.yaml

# 增加 node 节点
# sealos add --nodes 192.168.17.133
# 增加 master 节点
# sealos add --masters 192.168.17.134
# 删除 node 节点
# sealos delete --nodes 192.168.17.133
# 删除 master 节点
# sealos delete --masters 192.168.17.134
# 清理集群
# sealos reset
```

参考：
https://www.sealos.io/zh-Hans/docs/getting-started/installation

## 通过 Sealer 安装

```bash
# 下载并安装sealer
curl -O https://ghproxy.com/https://github.com/sealerio/sealer/releases/download/v0.8.6/sealer-v0.8.6-linux-amd64.tar.gz
tar zxvf sealer-v0.8.6-linux-amd64.tar.gz && mv sealer /usr/bin
# 添加命令补全
echo "source <(sealer completion bash)" >> ~/.bashrc
# 查看可用的kubernetes镜像
sealer search kubernetes
# 安装kubernetes集群
sealer pull kubernetes:v1.23.8
sealer run kubernetes:v1.23.8 --masters 192.168.17.133 --nodes 192.168.17.134 --passwd toor

# 增加master节点
# sealer join --masters 192.168.0.2
# 增加node节点
# sealer join --nodes 192.168.0.3
# 删除master节点
# sealer delete --masters 192.168.0.2
# 删除node节点
# sealer delete --nodes 192.168.0.3
# 释放集群
# sealer delete -f /root/.sealer/my-cluster/Clusterfile
# 或
# sealer delete --all
# 升级集群（升级失败。。。不支持跨版本升级）
# sealer upgrade kubernetes:v1.24.6 -c my-cluster
```

参考：

https://github.com/sealerio/sealer

## 安装容器运行时

集群的所有节点都要安装容器运行时。主要步骤如下：

1. 处理好前提条件
2. 安装支持 CNI 的容器运行时，如 containerd、cri-o、cri-dockerd（需要先安装 docker）
3. 配置 systemd 取代 cgroupfs 为 cgroup driver
4. 配置 pause 镜像，不然下载不了 `registry.k8s.io/pause` 镜像

参考：https://kubernetes.io/docs/setup/production-environment/container-runtimes

### 前提条件

**转发 IPv4 并让 iptables 看到桥接流量**

> 如果选择安装 Docker，则可省去这步，因为 Docker 已经帮忙配置了

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 使 sysctl 参数无需启动就能生效
sudo sysctl --system

# 验证
lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```



参考：https://kubernetes.io/docs/setup/production-environment/container-runtimes/#forwarding-ipv4-and-letting-iptables-see-bridged-traffic

### cri-dockerd

> 由于 Kubernetes 1.24+ 已经删除 `dockershim` 这个 CRI 兼容层，所以要想继续使用 Docker 作为容器运行时，需要安装这个。cri-dockerd 前身就是 dockershim。

```bash
# 安装前需要先安装 Docker，并设置 systemd 为 cgroup driver，再进行以下操作

# 下载和安装
curl -O https://ghproxy.com/https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.0/cri-dockerd-0.3.0-3.el7.x86_64.rpm
rpm -ivh cri-dockerd-0.3.0-3.el7.x86_64.rpm

# 配置 Pause 镜像
PAUSE_IMAGE_VERSION=`cri-dockerd -h | grep -Eo '"registry.k8s.io/pause:.*"' | grep -Eo '[0-9]+.[0-9]+'`
echo $PAUSE_IMAGE_VERSION
sed -i "s,ExecStart=/usr/bin/cri-dockerd,ExecStart=/usr/bin/cri-dockerd --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:$PAUSE_IMAGE_VERSION," /usr/lib/systemd/system/cri-docker.service

systemctl daemon-reload
systemctl restart cri-docker.service
systemctl status cri-docker.service

# 开机启动
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```



参考：https://github.com/Mirantis/cri-dockerd

## 安装三件套

集群的所有节点都要安装 kubeadm、kubelet 和 kubectl 三件套。这里以安装 Kubernetes 1.25.0 为例。

```bash
# 这里使用阿里的镜像
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 统一 Kubernetes 版本号
VERSION=1.25.0
sudo yum install -y kubelet-$VERSION kubeadm-$VERSION kubectl-$VERSION --disableexcludes=kubernetes

sudo systemctl enable --now kubelet

# 添加 kubectl、kubeadm 命令补全
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "source <(kubeadm completion bash)" >> ~/.bashrc
```



参考：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

## 初始化控制平面节点

集群中所有节点都要添加集群域名解析，否则连不上集群：

```bash
CLUSTER_ENDPOINT=apiserver.cluster.local
echo "192.168.17.135    $CLUSTER_ENDPOINT" >> /etc/hosts
```

在控制平面节点执行以下命令：

```bash
# 不同的容器运行时，CRI 套接字不同
CRI_SOCKET=unix:///var/run/cri-dockerd.sock
#CRI_SOCKET=unix:///run/containerd/containerd.sock
#CRI_SOCKET=unix:///run/crio/crio.sock

IMAGE_REGISTRY=registry.aliyuncs.com/google_containers

# 生成配置文件（以Kubernetes 1.25.0 为例）
# 完整配置见 https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta3
POD_NETWORK=10.244.0.0/16

cat > kubeadm-config.yaml <<EOF
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
imageRepository: $IMAGE_REGISTRY
controlPlaneEndpoint: "$CLUSTER_ENDPOINT:6443"
networking:
  podSubnet: "$POD_NETWORK"
apiServer:
  timeoutForControlPlane: 2m0s
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  criSocket: "$CRI_SOCKET"
EOF

# 下载镜像
kubeadm config images pull --config kubeadm-config.yaml
# 查看下载的镜像
crictl --runtime-endpoint $CRI_SOCKET images

# 初始化集群
kubeadm init --config kubeadm-config.yaml --dry-run
# 这里只是 dry-run 测试一下看看有无错误，如果没有的话就去掉上面的 --dry-run 真正执行
```

若初始化过程中出现问题，则到[集群清理](##集群清理)这步进行重置和清理

## 配置 kubectl

```bash
# 控制平面节点配置
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 验证
kubectl cluster-info

# 工作节点配置
# 将控制平面节点的配置复制到工作节点，如下面的k2节点
# scp -r ~/.kube root@k2:~/
```

参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#optional-controlling-your-cluster-from-machines-other-than-the-control-plane-node

## 安装 CNI 插件

安装 CNI 插件后，集群中的 Pod 才能互相通信，CoreDNS 的状态才会由 `Pending` 变成 `Running` 。

参考：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network

### Calico

```bash
# 配置NetworkManager，集群中所有节点都要配置
cat > /etc/NetworkManager/conf.d/calico.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
EOF

# 注意POD_NETWORK要和上面的保持一致，否则 CoreDNS 不正常
POD_NETWORK=10.244.0.0/16
curl -O https://ghproxy.com/https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f tigera-operator.yaml

curl -o calico-custom-resources.yaml https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/custom-resources.yaml
sed -i "s#192.168.0.0/16#${POD_NETWORK}#" calico-custom-resources.yaml
kubectl create -f calico-custom-resources.yaml

# Wait until each pod has the STATUS of Running.
watch kubectl get pods -n calico-system
```

参考：

https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart

### Flannel

```bash
# 注意POD_NETWORK要和上面的保持一致，否则 CoreDNS 不正常
POD_NETWORK=10.244.0.0/16

curl -O kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
sed -i "s#10.244.0.0/16#${POD_NETWORK}#" kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

参考：

https://github.com/flannel-io/flannel

## 添加节点

```bash
# 打印集群添加节点要执行的命令
kubeadm token create --print-join-command

# 添加工作节点
kubeadm join apiserver.cluster.local:6443 --token qz8jew.4uvg8na1m3gd1s3p --discovery-token-ca-cert-hash sha256:8c51c5a570b182a6b4ca01eba0aa576ab229ebfa0753a890c0f34178d4026b8e

# 添加控制平面节点（比添加工作节点多了个 --control-plane）
kubeadm join apiserver.cluster.local:6443 --token qz8jew.4uvg8na1m3gd1s3p --discovery-token-ca-cert-hash sha256:8c51c5a570b182a6b4ca01eba0aa576ab229ebfa0753a890c0f34178d4026b8e --control-plane

# 查看节点列表
kubectl get nodes
```

参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes

[kubeadm join | Kubernetes](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/)

## CoreDNS 高可用

由于集群节点通常是按顺序初始化的，因此 CoreDNS 很可能只在第一个控制平面节点上运行。为了提供更高的可用性，需要在至少加入一个新节点后重新启动部署 CoreDNS。

```bash
kubectl -n kube-system rollout restart deployment coredns
```

参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes

## 问题排查

在[初始化控制平面节点](##初始化控制平面节点)时出现过以下报错：

```bash
Unfortunately, an error has occurred:
        timed out waiting for the condition

This error is likely caused by:
        - The kubelet is not running
        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
        - 'systemctl status kubelet'
        - 'journalctl -xeu kubelet'

Additionally, a control plane component may have crashed or exited when started by the container runtime.
To troubleshoot, list all containers using your preferred container runtimes CLI.
Here is one example how you may list all running Kubernetes containers by using crictl:
        - 'crictl --runtime-endpoint unix:///var/run/cri-dockerd.sock ps -a | grep kube | grep -v pause'
        Once you have found the failing container, you can inspect its logs with:
        - 'crictl --runtime-endpoint unix:///var/run/cri-dockerd.sock logs CONTAINERID'
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
```

根据上面信息，执行以下命令查看，没得到有用的信息：

```bash
systemctl status kubelet
journalctl -xeu kubelet -f
# 一直报以下错误：
# "Error getting node" err="node \"k1\" not found"
```

执行以下命令查看，发现没有任何容器在运行：

```bash
crictl --runtime-endpoint unix:///var/run/cri-dockerd.sock ps -a | grep kube | grep -v pause
```

以为是 cri-dockerd 出问题，执行以下命令查看：

```bash
systemctl status cri-docker.service
# 一直在刷以下信息：
# level=info msg="Pulling the image without credentials. Image: registry.k8s.io/pause:3.6"
```

结合上面信息，最后得知是下载不了 `registry.k8s.io/pause:3.6` 镜像导致的。

解决方法：

```bash
docker pull registry.aliyuncs.com/google_containers/pause:3.6
docker tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6
```

或者配置容器运行时的 pause 镜像，具体参考：[Container Runtimes | Kubernetes](https://v1-25.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/)

## 集群清理

**移除节点**

```bash
kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets
kubeadm reset
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
ipvsadm -C
kubectl delete node <node name>
```

**重置控制平面节点**

```bash
# 重置控制平面节点
kubeadm reset -f
# 删除CNI插件配置信息
# rm -rf /etc/cni/net.d/
# 删除 kubelet 配置文件（kubeadm 1.21.x 版本会自动清理）
# rm -rf /etc/kubernetes/ /var/lib/kubelet
# 删除 kubectl 配置文件
rm -rf $HOME/.kube/config
```



参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down

[kubeadm reset | Kubernetes](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-reset/)

## 设置控制平面节点能被调度

```bash
# remove the node-role.kubernetes.io/control-plane:NoSchedule taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# add the NoSchedule taint back
# kubectl taint nodes centos-vm node-role.kubernetes.io/master=true:NoSchedule
```

参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation

## 配置 [crictl](https://github.com/kubernetes-sigs/cri-tools)

```bash
# 在 Kubernetes 中是使用 crictl 替代 docker 命令来操作镜像、容器和Pod
# 添加 crictl 命令补全
echo "source <(crictl completion bash)" >> ~/.bashrc

# 配置 runtime-endpoint
crictl config --set runtime-endpoint=unix:///var/run/cri-dockerd.sock
#crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
#crictl config --set runtime-endpoint=unix:///run/crio/crio.sock

# 验证
crictl images
```

参考：

https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/

# Helm

Helm 是 K8s 的包管理工具。

```bash
export HELM_VERSION=v3.10.2
# 二进制方式安装
curl -O https://repo.huaweicloud.com/helm/${HELM_VERSION}/helm-${HELM_VERSION}-linux-amd64.tar.gz
tar -zxvf helm-${HELM_VERSION}-linux-amd64.tar.gz 
sudo install linux-amd64/helm /usr/local/bin/helm
# 添加命令补全
echo "source <(helm completion bash)" >> ~/.bashrc
# 添加仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add azure http://mirror.azure.cn/kubernetes/charts
helm repo update
```

参考：

https://helm.sh/docs/intro/install/

https://helm.sh/docs/intro/using_helm/

# Kubernetes Dashboard

执行以下命令：

```bash
# 根据 Kubernetes 版本选择对应的 Dashboard 版本
# v2.7.0 for Kubernetes 1.25
# v2.6.1 for Kubernetes 1.24
# v2.5.1 for Kubernetes 1.23
VERSION=v2.7.0
curl -o kubernetes-dashboard-$VERSION.yaml https://raw.githubusercontent.com/kubernetes/dashboard/$VERSION/aio/deploy/recommended.yaml

kubectl apply -f kubernetes-dashboard-$VERSION.yaml

# 暴露集群外端口
kubectl -n kubernetes-dashboard patch service kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'
# 查看集群外端口
echo `kubectl -n kubernetes-dashboard get svc kubernetes-dashboard  -o jsonpath="{.spec.ports[0].nodePort}"`
# 访问 https://{nodeIp}:{nodePort}
```

访问后页面会提示输入 Token 或指定 Kubeconfig 路径才能登陆。这里示范一下如何生成 Token。



新建一个文件 `dashboard-adminuser.yaml`，内容为：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```

然后执行以下命令：

```bash
kubectl apply -f dashboard-adminuser.yaml

# 生成登录 TOKEN
# Dashboard v2.6+ and Kubernetes 1.24+
kubectl -n kubernetes-dashboard create token admin-user
# Dashboard v2.5- and Kubernetes 1.23-
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

复制生成的 Token 进去就能登陆成功。



注销登录后，执行以下命令删除创建 Token 时用到的 `ServiceAccount` 和 `ClusterRoleBinding` 。

```bash
kubectl -n kubernetes-dashboard delete serviceaccount admin-user
kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```



参考：

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

https://github.com/kubernetes/dashboard/blob/v2.5.1/docs/user/access-control/creating-sample-user.md

# Metrics Server

Metrics Server 通过 Metrics API 暴露 Kubernetes Metrics，以支持 Horizontal Pod Autoscaler 和 `kubectl top` 功能。缺点是 Metrics Server 只支持每个集群节点最多 30 个 Pod，超过的话会导致 OOM。



通过 YAML 文件安装：

```bash
curl -o kubernetes-metrics-server.yaml https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# 替换一下镜像，不然下载不了
sed -i "s|k8s.gcr.io/metrics-server/metrics-server|registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server|g" kubernetes-metrics-server.yaml
# 添加 --kubelet-insecure-tls 参数到 Metrics Server 中以跳过证书校验，否则无法正常运行
sed -i -e '/metric-resolution/p' -e 's/metric-resolution=15s/kubelet-insecure-tls/' kubernetes-metrics-server.yaml

kubectl apply -f kubernetes-metrics-server.yaml
```

安装后，可以通过 `kubectl top nodes` / `kubectl top pods` 指令查看节点/容器组的资源利用率。执行结果示例：

```bash
# kubectl top nodes
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
c8          107m         5%     723Mi           19%
centos-vm   440m         22%    1424Mi          38%

# kubectl top pods -n kube-system
NAME                                CPU(cores)   MEMORY(bytes)
coredns-68b9d7b887-fn2cq            4m           12Mi
coredns-68b9d7b887-nsh8d            4m           12Mi
etcd-centos-vm                      39m          64Mi
kube-apiserver-centos-vm            128m         411Mi
kube-controller-manager-centos-vm   22m          53Mi
kube-proxy-mpx47                    1m           18Mi
kube-proxy-plggp                    1m           23Mi
kube-scheduler-centos-vm            4m           22Mi
kuboard-74c645f5df-zcq5c            0m           8Mi
metrics-server-7dbf6c4558-4wbdx     1m           14Mi
```

如果执行命令报错，则请查看 [安装前提条件](https://github.com/kubernetes-sigs/metrics-server#requirements) 排查一下有哪些条件没满足导致的。



参考：

https://github.com/kubernetes-sigs/metrics-server

https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server

# OpenEBS

OpenEBS is Kubernetes native Container Attached Storage solution

**通过 Helm 安装**

```bash
helm repo add openebs https://openebs.github.io/charts
helm pull openebs/openebs --untar
helm install --namespace openebs openebs ./openebs --create-namespace
```

**通过 kubectl 安装**

```bash
curl -O https://openebs.github.io/charts/openebs-operator.yaml
kubectl apply -f openebs-operator.yaml
```

**安装后**

```bash
# 设置 openebs-hostpath 为默认 storageclass
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```



参考：

https://openebs.io/docs/user-guides/localpv-hostpath

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/change-default-storage-class/

# Ingress

## NGINX Ingress Controller

> Nginx 社区维护的 Ingress Controller

**Installation with Manifests**

```bash
git clone https://ghproxy.com/https://github.com/nginxinc/kubernetes-ingress.git --branch v2.4.2
cd kubernetes-ingress/deployments

# 1. Configure RBAC
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml
# 2. Create Common Resources
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f common/ingress-class.yaml

kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
kubectl apply -f common/crds/k8s.nginx.org_globalconfigurations.yaml

# 3. Deploy the Ingress Controller
# DaemonSet. Use a DaemonSet for deploying the Ingress Controller on every node or a subset of nodes.
kubectl apply -f daemon-set/nginx-ingress.yaml
# Deployment. Use a Deployment if you plan to dynamically change the number of Ingress Controller replicas.
# kubectl apply -f deployment/nginx-ingress.yaml

# 4. Get Access to the Ingress Controller
# 如果选择部署 DaemonSet，则直接访问部署了 DaemonSet 的节点的 80 和 443 端口
# 如果选择部署 Deployment，则使用下面的 NodePort Service 来访问
# kubectl create -f service/nodeport.yaml

# 5. Testing
cd ../examples/ingress-resources/complete-example
kubectl create -f cafe.yaml
kubectl create -f cafe-secret.yaml
kubectl create -f cafe-ingress.yaml

IC_IP=192.168.17.132
IC_HTTPS_PORT=443
curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/coffee --insecure

# 6. Uninstall
kubectl delete namespace nginx-ingress
kubectl delete clusterrole nginx-ingress
kubectl delete clusterrolebinding nginx-ingress
kubectl delete -f common/crds/
```



**Installation with Helm**

```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm pull nginx-stable/nginx-ingress --version 0.15.1 --untar
# helm install nginx-release ./nginx-ingress --set controller.service.create=false,controller.kind=daemonset,controller.setAsDefaultIngress=true
helm install nginx-release ./nginx-ingress --set controller.service.type=LoadBalancer,controller.kind=deployment,controller.setAsDefaultIngress=true

# Testing Service
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
kubectl create ingress demo-localhost --class=nginx \
  --rule="demo.localdev.me/*=demo:80"

IC_IP=192.168.17.131
IC_HTTP_PORT=`kubectl get svc nginx-release-nginx-ingress -o jsonpath="{.spec.ports[0].nodePort}"`
curl --resolve demo.localdev.me:$IC_HTTP_PORT:$IC_IP http://demo.localdev.me:$IC_HTTP_PORT/
```



参考：

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/

https://github.com/nginxinc/kubernetes-ingress/tree/main/examples/ingress-resources/complete-example

https://github.com/nginxinc/kubernetes-ingress

## Kubernetes Ingress Controller

> Kubernetes 社区维护基于 Nginx 的 Ingress Controller

不推荐，因为镜像要翻墙，而且还要检验SHA256。

```bash
# Using YAML manifests
# v1.5.1 for Kubernetes 1.23+
# download baremetal/deploy.yaml, not cloud/deploy.yaml
INGRESS_VERSION=v1.5.1
curl -o ingress-nginx-$INGRESS_VERSION.yaml https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-$INGRESS_VERSION/deploy/static/provider/baremetal/deploy.yaml

# apply
kubectl apply -f ingress-nginx-$INGRESS_VERSION.yaml

# wait for ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

参考：

https://kubernetes.github.io/ingress-nginx/deploy/

https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

https://github.com/kubernetes/ingress-nginx/

# Istio

参考：https://istio.io/latest/docs/setup/getting-started/

# Envoy

# KubeSphere

```bash
VERSION=v3.3.1
curl https://ghproxy.com/https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/kubesphere-installer.yaml -o kubesphere-installer-$VERSION.yaml
curl https://ghproxy.com/https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/cluster-configuration.yaml -o cluster-configuration-$VERSION.yaml
kubectl apply -f kubesphere-installer-$VERSION.yaml
kubectl apply -f cluster-configuration-$VERSION.yaml

# 查看安装日志，安装成功后会打印控制台地址，浏览器访问即可
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
# Account: admin
# Password: P@88w0rd

```

参考：https://kubesphere.io/docs/v3.3/quick-start/minimal-kubesphere-on-k8s/

# Rainbond

```bash
# 所有节点都要执行这个命令
yum -y install nfs-utils

# 开始安装
helm repo add rainbond https://openchart.goodrain.com/goodrain/rainbond
helm pull rainbond/rainbond-cluster --untar
helm install rainbond ./rainbond-cluster -n rbd-system --create-namespace

# 访问平台
kubectl get rainbondcluster rainbondcluster -n rbd-system -o go-template --template='{{range.spec.gatewayIngressIPs}}{{.}}:7070{{printf "\n"}}{{end}}'
```

卸载

```bash
helm uninstall rainbond -n rbd-system

# Delete PVC
kubectl get pvc -n rbd-system | grep -v NAME | awk '{print $1}' | xargs kubectl delete pvc -n rbd-system

# Delete PV
kubectl get pv | grep rbd-system | grep -v NAME | awk '{print $1}' | xargs kubectl delete pv

# Delete CRD
kubectl delete crd componentdefinitions.rainbond.io \
helmapps.rainbond.io \
rainbondclusters.rainbond.io \
rainbondpackages.rainbond.io \
rainbondvolumes.rainbond.io \
rbdcomponents.rainbond.io \
servicemonitors.monitoring.coreos.com \
thirdcomponents.rainbond.io \
-n rbd-system

# Delete NAMESPACE
kubectl delete ns rbd-system

# 删除 Rainbond 数据目录
rm -rf /opt/rainbond
```



参考：

https://www.rainbond.com/docs/installation/install-with-helm/install-from-kubernetes

https://www.rainbond.com/docs/installation/uninstall

# Kuboard

> Kuboard 是开源 Kubernetes 多集群管理界面

```bash
docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 80:80/tcp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.17.131:80" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -e KUBOARD_ADMIN_DERAULT_PASSWORD=Kuboard123 \
  -v /root/kuboard-data:/data \
  swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v3
```

浏览器访问上面 `KUBOARD_ENDPOINT` 填的地址。

用户名是 admin，密码是上面`KUBOARD_ADMIN_DERAULT_PASSWORD` 填的值。

登录后需要添加 Kubernetes 集群，建议通过 `agent` 方式添加。



参考：https://kuboard.cn/install/v3/install-built-in.html

# Rancher

Rancher 是开源的企业级 Kubernetes 管理平台。

**Installation with docker**

安装时会自动创建一个 Kubernetes 集群，需要登录进 Rancher 控制台才能看到该集群。

```bash
sudo docker run --privileged -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 -v /var/lib/rancher/:/var/lib/rancher/ rancher/rancher:stable
```

默认用户名为 admin，安装后会提示设置密码，这里设置为 admin

**Installation with helm**

安装条件：Kubernetes < v1.25.0，Ingress（建议以 DaemonSet 方式安装）

```bash
# Install cert-manager First
VERSION=v1.7.1
curl -o cert-manager.crds-$VERSION.yaml https://ghproxy.com/https://github.com/cert-manager/cert-manager/releases/download/$VERSION/cert-manager.crds.yaml
kubectl apply -f cert-manager.crds-$VERSION.yaml

helm repo add jetstack https://charts.jetstack.io
helm pull jetstack/cert-manager --version $VERSION --untar
helm install cert-manager ./cert-manager \
  --namespace cert-manager \
  --create-namespace

# Install Rancher
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm search repo rancher-stable/rancher --versions
helm pull rancher-stable/rancher --version=2.7.0 --untar
helm install rancher ./rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.my.org \
  --set ingress.ingressClassName=nginx \
  --set bootstrapPassword=admin

# Verify that the Rancher Server is Successfully Deployed
kubectl -n cattle-system rollout status deploy/rancher

# 浏览器访问以下链接
echo https://rancher.my.org/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')
# 如果访问不了，查看 Ingress 是否已被 AddedOrUpdated，如果不是，说明规则有问题，需要修复一下
kubectl -n cattle-system describe ingress rancher

# Uninstall Rancher
curl -O https://raw.githubusercontent.com/rancher/rancher-cleanup/main/deploy/rancher-cleanup.yaml
kubectl create -f rancher-cleanup.yaml
```



参考：

https://www.rancher.com/quick-start

https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster

https://github.com/rancher/rancher-cleanup

# kube-prometheus

**前提条件**

查看 `/var/lib/kubelet/config.yaml` 文件，确保以下几点：

- `--authentication-token-webhook=true` 或 `authentication.webhook.enabled=true` 
- `--authorization-mode=Webhook` 

**Installation with Manifests**

```bash
# Download
VERSION=0.10.0
curl -o kube-prometheus-$VERSION.tar.gz https://ghproxy.com/https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v$VERSION.tar.gz

tar xvzf kube-prometheus-$VERSION.tar.gz && cd kube-prometheus-$VERSION

# Pull images
docker pull v5cn/prometheus-adapter:v0.9.1
docker tag v5cn/prometheus-adapter:v0.9.1 k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1

docker pull zhengsenlin/kube-state-metrics:v2.3.0
docker tag zhengsenlin/kube-state-metrics:v2.3.0 k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.3.0

# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
kubectl create -f manifests/setup
kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring

# Create remaining resources and wait for them to be available
kubectl create -f manifests/
kubectl wait \
	--for condition=Ready \
	--all Pods \
	--timeout=120s \
	--namespace=monitoring

# Expose NodePort
# the default grafana user:password is admin:admin.
kubectl -n monitoring patch service grafana -p '{"spec":{"type":"NodePort"}}'
kubectl -n monitoring patch service prometheus-k8s -p '{"spec":{"type":"NodePort"}}'
kubectl -n monitoring patch service alertmanager-main -p '{"spec":{"type":"NodePort"}}'

echo `kubectl -n monitoring get svc grafana -o jsonpath="{.spec.ports[0].nodePort}"`
echo `kubectl -n monitoring get svc prometheus-k8s -o jsonpath="{.spec.ports[0].nodePort}"`
echo `kubectl -n monitoring get svc alertmanager-main -o jsonpath="{.spec.ports[0].nodePort}"`

# Uninstall
# kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

**Installation with Helm**

```bash
# 解决部分镜像下载不了的问题
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.3.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.3.0 k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.3.0

docker pull bitnami/kube-state-metrics:2.6.0
docker tag bitnami/kube-state-metrics:2.6.0 registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.6.0

# 下载并安装
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm pull prometheus-community/kube-prometheus-stack --version 41.9.1 --untar
helm install --set grafana.adminPassword=admin prometheus-release ./kube-prometheus-stack

# Expose Grafana NodePort
kubectl patch service prometheus-release-grafana -p '{"spec":{"type":"NodePort"}}'
echo `kubectl get svc prometheus-release-grafana -o jsonpath="{.spec.ports[0].nodePort}"`
# Grafana default adminPassword: prom-operator
```

参考：

https://github.com/prometheus-operator/kube-prometheus

# MySQL

## 单节点 Local Volume

**创建本地存储 StorageClass**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

**创建 PV**

添加本地存储 PV，并通过 storageClassName 属性与 StorageClass 绑定。PVC 绑定 StorageClass 后，会由 StorageClass 动态分配 PV 与 PVC 绑定。注意，PV 一旦与某个 PVC 绑定后，即使该 PVC 删除了，也不会再与其它 PVC 绑定，需要手动删除 PV 及其数据并重新创建 PV 才行。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv1
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    # 需要提前创建好该目录
    path: /mnt/disks/pv1
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            # 筛选符合标签的node，需要根据实际情况填写
            - key: kubernetes.io/hostname
              operator: In
              values:
                - kube-master
                - docker-desktop
```

**创建 MySQL 部署和服务**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      nodePort: 30006
  selector:
    app: mysql
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
kind: Secret
apiVersion: v1
metadata:
  name: mysql-secret
data:
  password: toor
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
          args:
            - --character-set-server=utf8mb4
            - --collation-server=utf8mb4_unicode_ci
      volumes:
        - name: mysql-volume
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

参考：

https://kubernetes.io/docs/concepts/storage/volumes/#local

https://kubernetes.io/docs/concepts/storage/storage-classes/#local

https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

https://kubernetes.io/docs/concepts/configuration/secret/

https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

## 单节点 HostPath Volume

上面的部署方式可以简化，跳过创建`StorageClass` 、`PV` 、`PVC` 这些资源

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      nodePort: 30006
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              # 不知道为啥使用 Secret 时登录老是提示密码不对，所以索性不用
              value: toor
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
          args:
            - --character-set-server=utf8mb4
            - --collation-server=utf8mb4_unicode_ci
      volumes:
        - name: mysql-volume
          hostPath:
            # 主机上的目录，按实际情况调整
            path: /k8sdata/mysql
            # 确保文件夹被创建
            type: DirectoryOrCreate
```

HostPath volume 是有很多安全问题的，官方建议不要用，一定要用的话推荐以只读的方式使用。

参考：

https://kubernetes.io/docs/concepts/storage/volumes/#hostpath

https://www.cnblogs.com/worldinmyeyes/p/14514971.html

## MySQL Operator

```bash
# MySQL Operator for Kubernetes Installation
helm repo add mysql-operator https://mysql.github.io/mysql-operator/
helm pull mysql-operator/mysql-operator --untar
helm install mysql-operator ./mysql-operator --namespace mysql-operator --create-namespace

# MySQL InnoDB Cluster Installation
helm pull mysql-operator/mysql-innodbcluster --version 2.0.7 --untar
helm install mysql-cluster ./mysql-innodbcluster \
        --set credentials.root.user='root' \
        --set credentials.root.password='toor' \
        --set credentials.root.host='%' \
        --set serverInstances=3 \
        --set routerInstances=1 \
        --set tls.useSelfSigned=true
```

参考：https://github.com/mysql/mysql-operator

## Bitnami MySQL Replication

```bash
# 添加 bitnami helm 仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
# 安装
helm pull bitnami/mysql --untar
helm install --set auth.rootPassword=toor,auth.database=app_database mysql-release ./mysql
# 查看 MySQL ROOT 密码
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-release -o jsonpath="{.data.mysql-root-password}" | base64 -d)
echo $MYSQL_ROOT_PASSWORD
# 连接数据库
kubectl exec -it mysql-release-0 -- mysql -uroot -p"$MYSQL_ROOT_PASSWORD"
# 连接主数据库
kubectl exec -it mysql-release-0 -- mysql -h mysql-release.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

# 卸载
helm uninstall mysql-release
```

参考：https://artifacthub.io/packages/helm/bitnami/mysql

# Redis

参考：

https://github.com/bitnami/charts/tree/main/bitnami/redis

https://github.com/bitnami/charts/tree/main/bitnami/redis-cluster