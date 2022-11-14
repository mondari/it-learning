# 安装 Kubernetes

## 前提条件

## Using Sealos

```bash
# 下载并安装 sealos
curl -O https://github.com/labring/sealos/releases/download/v4.1.3/sealos_4.1.3_linux_amd64.tar.gz && tar zxvf sealos_4.1.3_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin
# 添加命令补全
echo "source <(sealos completion bash)" >> ~/.bashrc
# 安装集群
sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 \
     --masters 192.168.17.131 \
     --nodes 192.168.17.132 -p toor

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

## Using Sealer

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

# Kubernetes Dashboard

执行以下命令：

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
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

# Helm

Helm 是 K8s 的包管理工具。

```bash
# 二进制方式安装
curl -O https://repo.huaweicloud.com/helm/v3.7.2/helm-v3.7.2-linux-amd64.tar.gz
tar -zxvf helm-v3.7.2-linux-amd64.tar.gz
sudo install linux-amd64/helm /usr/local/bin/helm
# 添加命令补全
echo "source <(helm completion bash)" >> ~/.bashrc
# 添加仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

参考：

https://helm.sh/docs/intro/install/

https://helm.sh/docs/intro/using_helm/

# OpenEBS

OpenEBS is Kubernetes native Container Attached Storage solution

参考：https://openebs.io/

# Istio

参考：https://istio.io/latest/docs/setup/getting-started/

# Rancher

Rancher 是开源的企业级 Kubernetes 管理平台。Rancher 在安装时会自动创建一个 Kubernetes 集群，需要登录进 Rancher 控制台才能看到该集群。

```bash
sudo docker run --privileged -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 -v /var/lib/rancher/:/var/lib/rancher/ rancher/rancher:stable
```

默认用户名为 admin，安装后会提示设置密码，这里设置为 admin

参考：https://www.rancher.cn/quick-start/

# MySQL

## 单节点 StorageClass

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
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: false
      volumes:
        - name: mysql-volume
          hostPath:
            # 主机上的目录，按实际情况调整
            path: /k8sdata/mysql
            # 确保文件夹被创建
            type: DirectoryOrCreate
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
  progressDeadlineSeconds: 600
```

参考：

https://kubernetes.io/docs/concepts/storage/volumes/#hostpath

https://www.cnblogs.com/worldinmyeyes/p/14514971.html

## MySQL Operator

参考：https://github.com/mysql/mysql-operator