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

## 通过 Sealos 安装

```bash
# 下载并安装 sealos
curl -O https://github.com/labring/sealos/releases/download/v4.1.3/sealos_4.1.3_linux_amd64.tar.gz
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
curl -O https://github.com/sealerio/sealer/releases/download/v0.8.6/sealer-v0.8.6-linux-amd64.tar.gz
tar zxvf sealer-v0.8.6-linux-amd64.tar.gz && mv sealer /usr/bin
# 添加命令补全
echo "source <(sealer completion bash)" >> ~/.bashrc
# 查看可用的kubernetes镜像
sealer search kubernetes
# 安装kubernetes集群
# sealer pull kubernetes:v1.23.8
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

## [cri-dockerd](https://github.com/Mirantis/cri-dockerd)

由于 Kubernetes 1.24+ 已经删除 `dockershim` 这个 CRI 兼容层，所以要想继续使用 Docker 作为容器运行时，需要安装这个。cri-dockerd 前身就是 dockershim。

## 配置 [crictl](https://github.com/kubernetes-sigs/cri-tools)

```bash
# 在 Kubernetes 中是使用 crictl 替代 docker 命令来操作镜像、容器和Pod
# 添加 crictl 命令补全
echo "source <(crictl completion bash)" >> ~/.bashrc
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
helm repo add stable https://charts.ost.ai
helm repo update
```

参考：

https://helm.sh/docs/intro/install/

https://helm.sh/docs/intro/using_helm/

https://charts.ost.ai/

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
helm repo update
helm install --namespace openebs --name openebs openebs/openebs
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
helm repo update
helm install nginx-release nginx-stable/nginx-ingress --set controller.service.type=NodePort

# 解决部分镜像下载不了的问题
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.3.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.3.0 k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.3.0
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

# Rancher

Rancher 是开源的企业级 Kubernetes 管理平台。Rancher 在安装时会自动创建一个 Kubernetes 集群，需要登录进 Rancher 控制台才能看到该集群。

```bash
sudo docker run --privileged -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 -v /var/lib/rancher/:/var/lib/rancher/ rancher/rancher:stable
```

默认用户名为 admin，安装后会提示设置密码，这里设置为 admin

参考：https://www.rancher.cn/quick-start/

# kube-prometheus-stack

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



参考：

https://github.com/prometheus-operator/kube-prometheus

# EFK

Elasticsearch+Fluentd+Kibana



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

参考：https://github.com/mysql/mysql-operator

## Bitnami MySQL

部署 MySQL 主从复制集群

```bash
# 添加 bitnami helm 仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
# 安装
helm install mysql-release bitnami/mysql
# 查看状态
kubectl get pods -w --namespace default
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

参考：

https://artifacthub.io/packages/helm/bitnami/mysql

# Redis