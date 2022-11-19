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
curl -O https://github.com/labring/sealos/releases/download/v4.1.3/sealos_4.1.3_linux_amd64.tar.gz && tar zxvf sealos_4.1.3_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin
# 添加命令补全
echo "source <(sealos completion bash)" >> ~/.bashrc
# 安装集群（containerd 容器运行时）
sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 \
     --masters 192.168.17.131 \
     --nodes 192.168.17.132 -p toor
# 安装集群（cri-docker 容器运行时）
# sealos run labring/kubernetes-docker:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 \
     --masters 192.168.17.131 \
     --nodes 192.168.17.132 -p toor
# 安装其它
sealos run labring/openebs:v3.3.0 labring/ingress-nginx:4.1.0

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
curl -O https://github.com/sealerio/sealer/releases/download/v0.8.6/sealer-v0.8.6-linux-amd64.tar.gz &&
tar zxvf sealer-v0.8.6-linux-amd64.tar.gz && mv sealer /usr/bin
# 添加命令补全
echo "source <(sealer completion bash)" >> ~/.bashrc
# 安装kubernetes集群
sealer run kubernetes:v1.19.8 --masters 192.168.17.133 --nodes 192.168.17.134 --passwd toor

# 增加master节点
sealer join --masters 192.168.0.2
# 增加node节点
sealer join --nodes 192.168.0.3
# 删除master节点
sealer delete --masters 192.168.0.2
# 删除node节点
sealer delete --nodes 192.168.0.3
# 释放集群
sealer delete -f /root/.sealer/[cluster-name]/Clusterfile
# 或
sealer delete --all
```

参考：
https://github.com/sealerio/sealer/blob/main/docs/README_zh.md

## [cri-dockerd](https://github.com/Mirantis/cri-dockerd)

## 后置操作

```bash
# 在 Kubernetes 中是使用 crictl 替代 docker 命令来操作镜像、容器和Pod
# 添加 crictl 命令补全
echo "source <(crictl completion bash)" >> ~/.bashrc
```

参考：

https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/

# Kubernetes Dashboard

执行以下命令：

```bash
export DASHBOARD_VERSION=v2.6.1
curl -o dashboard-${DASHBOARD_VERSION}.yaml https://raw.githubusercontent.com/kubernetes/dashboard/${DASHBOARD_VERSION}/aio/deploy/recommended.yaml
kubectl apply -f recommended.yaml
kubectl proxy
```

然后浏览器打开链接 http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

页面会提示输入 Token 或指定 Kubeconfig 路径才能登陆。这里示范一下如何生成 Token。

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
kubectl -n kubernetes-dashboard create token admin-user
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

# Metrics Server

参考：https://github.com/kubernetes-sigs/metrics-server

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

HostPath volume 是有很多安全问题的，官方建议不要用，硬要用的话推荐以只读的方式使用。

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