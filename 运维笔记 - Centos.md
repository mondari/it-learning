[TOC]



## 用户名和密码

- 用户名：root
- 密码：toor

## 发行文档

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/

## Linux 发行版对比

|            | Centos 7                         | Centos 8                                    | Ubuntu Server 2018 LTS                                       | Ubuntu Server 2020 LTS | Arch Linux 2020.08.01 | 备注                            |
| ---------- | -------------------------------- | ------------------------------------------- | ------------------------------------------------------------ | ---------------------- | --------------------- | ------------------------------- |
| 内核       | 3.10                             | 4.18                                        | 4.15                                                         | 5.4                    | 5.7.11                |                                 |
| 包管理     | yum                              | dnf 替代 yum                                |                                                              | apt, snap              |                       |                                 |
| 包仓库     | Base, Extras, Updates            | BaseOS, AppStream, Extras                   |                                                              |                        |                       |                                 |
| 网络管理器 | NetworkManager, nmcli, nmtui     | 同左                                        | [netplan.io](https://netplan.io/) 基于 NetworkManager 和 Systemd-networkd | 同左                   |                       |                                 |
| 网络工具包 | ip, ss                           | 同左                                        | ifconfig, netstat                                            |                        |                       |                                 |
| 网络包过滤 | iptables                         | nftables                                    |                                                              |                        |                       | 都是内核的 netfilter 框架的成员 |
| 防火墙     | firewalld, firewall-cmd          | 同左                                        |                                                              | ufw                    |                       |                                 |
| 域名解析   |                                  | systemd-resolved                            |                                                              |                        |                       |                                 |
| 文件系统   | XFS                              | XFS                                         |                                                              | ext4                   |                       |                                 |
| 显示服务器 | [X.org](https://www.x.org/wiki/) | [Wayland](https://wayland.freedesktop.org/) |                                                              |                        |                       |                                 |
| Web 控制台 | 默认无                           | Cockpit                                     |                                                              |                        |                       |                                 |
| 容器管理   | Docker 1.13                      | Podman                                      | docker.io 19.03                                              | docker.io 19.03        |                       |                                 |
| 容器编排   | Kubernetes 1.5.2                 | -                                           |                                                              |                        |                       |                                 |
| MySQL      | -                                | 8.0                                         |                                                              | 8.0                    |                       |                                 |
| MariaDB    | 5.5                              | 10.3                                        |                                                              | 10.3                   |                       |                                 |
| PostgreSQL | 9.2                              | 10.6                                        |                                                              | 12                     |                       |                                 |
| Redis      | -                                | 5.0                                         |                                                              | 5.0                    |                       |                                 |
| OpenJDK    | 8, 11                            | 8, 11                                       | 8, 11                                                        | 8, 11                  |                       |                                 |
| Maven      | -                                | 3.5                                         |                                                              | 3.6.3                  |                       |                                 |
| Python     | 3.6                              | 3.8                                         |                                                              | 3.8                    |                       |                                 |
| Anaconda   | 21                               | 29                                          |                                                              | -                      |                       |                                 |
| PHP        | 5.4                              | 7.2                                         | 7.2                                                          | 7.4                    |                       |                                 |
| Nodejs     | -                                | 10.19                                       |                                                              | 10.19                  |                       |                                 |
| Nginx      | -                                | 1.14                                        |                                                              | 1.17                   |                       |                                 |
| httpd      | 2.4.6                            | 2.4.37                                      |                                                              |                        |                       |                                 |

参考：https://www.archlinux.org/releng/releases/

## Centos Stream 安装源

在用 Centos Stream CD 来安装 Centos Stream 时，需要指定安装源才能继续安装，否则只能使用 DVD 来安装。

推荐使用阿里的安装源：http://mirrors.aliyun.com/centos/8-stream/BaseOS/x86_64/os/

## 配置 Bash

### .inputrc

```bash
// 设置 bash 大小写不敏感
$ echo "set completion-ignore-case on">>~/.inputrc
```

### 设置命令别名

```bash
$ vi ~/.bashrc
alias ll='ls -alFh --color'
```

ls 命令的 -F 选项可以让目录在后面加 “/”的形式显示，方便区分

## 常用包

yum groups install Development\ Tools（内含 gcc, git, cmake, perl）
net-tools.x86_64（内含 netstat, ifconfig, route，注意，该工具包已经被 iproute 工具包代替）
bridge-utils（内含 brctl 网桥管理工具）
yum-cron
bash-completion
mlocate.x86_64
java-1.8.0-openjdk-devel.x86_64（内含 Java 诊断工具）
epel-release（EPEL仓库有Python3）
yum install python-pip（默认安装pyhon-pip2）
pip install --upgrade pip（更新pip）
open-vm-tools.x86_64（[VMware 虚拟机包](https://github.com/vmware/open-vm-tools)）
psmisc（内含 pstree）

## 设置静态IP地址

```bash
nmcli connection modify ens33 ipv4.addresses 192.168.17.136/24
nmcli connection modify ens33 ipv4.gateway 192.168.17.2
nmcli connection modify ens33 ipv4.dns 114.114.114.114
nmcli connection modify ens33 ipv4.method manual
nmcli connection up ens33
```

## 配置EPEL镜像

```bash
// 备份(如有配置其他epel源)

$ mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
$ mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup

// 下载新repo 到/etc/yum.repos.d/
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

```

## 配置PyPi镜像

```bash
$ mkdir ~/.pip
$ cat > ~/.pip/pip.conf <<EOF
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

## neofetch

neofetch 是用来打印操作系统信息和 ASCII 版的操作系统 LOGO 图片

```bash
$ curl -o /etc/yum.repos.d/konimex-neofetch-epel-7.repo https://copr.fedorainfracloud.org/coprs/konimex/neofetch/repo/epel-7/konimex-neofetch-epel-7.repo

$ yum install neofetch
```

参考：https://github.com/dylanaraps/neofetch/wiki/Installation#fedora--rhel--centos--mageia

## Docker

### 安装 Docker CE

#### Centos

```bash
# Uninstall old versions
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
                  
# Install required packages                  
sudo yum install -y yum-utils
  
# Set up the stable repository.  
# 这里改用阿里云的 docker-ce.repo 文件
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 并将仓库地址替换
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
    
# Install the latest version of Docker Engine - Community and containerd    
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 启动服务设为开机启动
sudo systemctl enable --now docker

# Verify
sudo docker version
```

参考：https://docs.docker.com/engine/install/centos/

#### Fedora

```bash
# Uninstall old versions
sudo dnf remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine
                  
# Install required packages
sudo dnf -y install dnf-plugins-core
  
# Set up the stable repository.  
# 这里改用阿里云的 docker-ce.repo 文件
sudo dnf config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/fedora/docker-ce.repo
    
# 并将仓库地址替换
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo

# Install the latest version of Docker Engine - Community and containerd    
sudo dnf install docker-ce docker-ce-cli containerd.io

# 启动服务设为开机启动
sudo systemctl enable --now docker

# Verify
sudo docker version
```

参考：https://docs.docker.com/engine/install/fedora/

### 设置 Docker Hub 镜像加速器

```bash
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://1nj0zren.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://registry.docker-cn.com"
    ]
}
EOF
# systemd 重新加载配置
sudo systemctl daemon-reload
# docker 重启
sudo systemctl restart docker
# 验证
docker info
```

**Docker Hub 镜像加速器列表**

| 镜像加速器                                                   | 镜像加速器地址                       | 专属加速器       | 其它加速                                                     |
| ------------------------------------------------------------ | ------------------------------------ | ---------------- | ------------------------------------------------------------ |
| [Docker 中国官方镜像](https://links.jianshu.com/go?to=https%3A%2F%2Fdocker-cn.com%2Fregistry-mirror) | `https://registry.docker-cn.com`     |                  | Docker Hub                                                   |
| [DaoCloud 镜像站](https://links.jianshu.com/go?to=https%3A%2F%2Fdaocloud.io%2Fmirror) | `http://f1361db2.m.daocloud.io`      | 可登录，系统分配 | Docker Hub                                                   |
| [Azure 中国镜像](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FAzure%2Fcontainer-service-for-azure-china%2Fblob%2Fmaster%2Faks%2FREADME.md%2322-container-registry-proxy) | `https://dockerhub.azk8s.cn`         |                  | Docker Hub、GCR、Quay                                        |
| [科大镜像站](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.ustc.edu.cn%2Fhelp%2Fdockerhub.html) | `https://docker.mirrors.ustc.edu.cn` |                  | Docker Hub、[GCR](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fustclug%2Fmirrorrequest%2Fissues%2F91)、[Quay](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fustclug%2Fmirrorrequest%2Fissues%2F135) |
| [阿里云](https://links.jianshu.com/go?to=https%3A%2F%2Fcr.console.aliyun.com) | `https://xxx.mirror.aliyuncs.com`    | 需登录，系统分配 | Docker Hub                                                   |
| [七牛云](https://links.jianshu.com/go?to=https%3A%2F%2Fkirk-enterprise.github.io%2Fhub-docs%2F%23%2Fuser-guide%2Fmirror) | `https://reg-mirror.qiniu.com`       |                  | Docker Hub、GCR、Quay                                        |
| [网易云](https://links.jianshu.com/go?to=https%3A%2F%2Fc.163yun.com%2Fhub) | `https://hub-mirror.c.163.com`       |                  | Docker Hub                                                   |
| [腾讯云](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F457%2F9113) | `https://mirror.ccs.tencentyun.com`  |                  | Docker Hub                                                   |

参考：[Docker Hub 镜像加速器](https://www.jianshu.com/p/5a911f20d93e)

### 设置 cgroup driver

推荐 systemd 取代 cgroupfs 为 cgroup driver：

```bash
# 检查 Cgroup Driver 是否为 systemd
docker info

# 不是则进行以下配置
# 在前面设置镜像加速器的配置上作更改
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
      "https://1nj0zren.mirror.aliyuncs.com",
      "https://docker.mirrors.ustc.edu.cn",
      "http://f1361db2.m.daocloud.io",
      "https://registry.docker-cn.com"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 开启远程访问

```bash
# 开启远程访问
sed -i "s#-H#-H tcp://0.0.0.0:2375 -H#g" /usr/lib/systemd/system/docker.service
# 重新加载配置文件
systemctl daemon-reload
# 重启服务
systemctl restart docker.service
# 查看端口是否开启
netstat -anp|grep 2375
# curl查看是否生效（docker info 命令就是读取该接口）
curl http://127.0.0.1:2375/info
# 开启防火墙端口
firewall-cmd --add-port=2375/tcp --permanent && firewall-cmd --reload
# Windows主机修改环境变量即可用 IDEA 连接
# DOCKER_HOST=tcp://REMOTE_DOCKER_IP:2375
```

### 开启 IPv4 转发以访问外网

**一般 Docker 安装后会自动开启 IPv4 转发功能，无需手动开启。**

如果不开启 IPv4 转发，Docker 容器将无法访问外网，会提示 “WARNING: IPv4 forwarding is disabled. Networking will not work”。

```bash
// 检查是否已经开启（为1表示已开启）
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

// 开启
$ vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
// 重新加载
$ sysctl -p /etc/sysctl.conf

// 或者通过以下命令开启 IPv4 转发
$ sysctl -w net.ipv4.ip_forward=1
```

## Docker Compose

### 安装 Docker Compose

```bash
# 下载 docker-compose（这里使用 daocloud 的镜像加快下载速度）
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.27.1/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 创建软连接
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 安装命令补全
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.27.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
# 验证
docker-compose version
# 使用 docker-compose 启动容器
docker-compose -f docker-compose.yml up -d
```

参考：https://docs.docker.com/compose/install/

### 部署 ELK

elk.yaml

```bash
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.4.1
    container_name: elasticsearch
    environment:
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #设置使用jvm内存大小
    volumes:
      - elastic-data:/usr/share/elasticsearch/data #数据目录挂载（需要创建目录并赋予777权限）
    ports:
      - 9200:9200
    restart: always
  kibana:
    image: kibana:7.4.1
    container_name: kibana
    links:
      - elasticsearch:es #用es这个域名访问elasticsearch服务
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - "elasticsearch.hosts=http://es:9200" #设置访问elasticsearch的地址
    ports:
      - 5601:5601
    restart: always
volumes:
  elastic-data:
```

### 部署微服务

microservice.yml

```yaml
version: '3'
services:
  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos-standalone
    ports:
      - 8848:8848
    restart: on-failure
    volumes: 
      - nacos-data
  seata-server:
    image: seataio/seata-server:latest
    container_name: seata-server
    hostname: seata-server
    ports:
      - 8091:8091
    environment:
      - SEATA_PORT=8091
    expose:
      - 8091
  zipkin:
    image: openzipkin/zipkin #zipkin-slim镜像也可以
    container_name: zipkin
    restart: always
    ports:
      - 9411:9411
volumes:
  nacos_data:
```

## Kubernetes

### 安装前步骤

安装 Minikube 和 Kubernetes 前先执行这一步操作，且所有节点都必须执行这步操作。

```bash
# 安装仓库（这里使用阿里的镜像）
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# SELinux运行模式切换为宽容模式
setenforce 0
# 关闭 SELinux（可选）
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 卸载旧版本
yum remove -y kubelet kubeadm kubectl
# 安装三件套（注意 kubelet 版本号不能高于 kube-apiserver，且只能比 kube-apiserver 低两个小版本）
export kubeversion=1.20.1
yum install -y kubelet-${kubeversion} kubeadm-${kubeversion} kubectl-${kubeversion}
# 确保 kubelet 的 cgroupDriver 为 systemd
sed -i "s/cgroupfs/systemd/g" /var/lib/kubelet/config.yaml
# 设置开机启动并启动 kubelet 服务
systemctl enable kubelet && systemctl start kubelet

# 安装 kubectl、kubeadm 命令补全
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "source <(kubeadm completion bash)" >> ~/.bashrc
```

如果此时执行 `systemctl status kubelet` 命令，将得到 kubelet 启动失败的错误提示，请忽略此错误，因为必须完成后续步骤中 `kubeadm init` 的操作，kubelet 才能正常启动

### 安装 Minikube

minikube 能在macOS、Linux和Windows上快速建立一个本地Kubernetes集群，帮助开发者开发 Kubernetes 应用。

```bash
# 安装 minikube（使用阿里镜像）
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.16.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

# 安装 minikube 命令补全
echo "source <(minikube completion bash)" >> ~/.bashrc

// 启动 minikube 集群
minikube start
// 查看 minikube 集群状态
minikube status
// 停止 minikube 集群
minikube stop
// 删除 minikube 集群
minikube delete
// 升级 minikube 版本前需要执行以下命令
minikube delete && rm -rf ~/.minikube
```

`minikube start --driver=none` 执行结果如下：

```bash
[root@localhost ~]# minikube start --driver=none

minikube start --driver=none
* minikube v1.13.0 on Centos 7.8.2003
* Using the none driver based on user configuration
* Starting control plane node minikube in cluster minikube
* Running on localhost (CPUs=2, Memory=3770MB, Disk=37874MB) ...
* OS release is CentOS Linux 7 (Core)
* Preparing Kubernetes v1.19.0 on Docker 19.03.13 ...
* Configuring local host environment ...
*
! The 'none' driver is designed for experts who need to integrate with an existing VM
* Most users should use the newer 'docker' driver instead, which does not require root!
* For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/
*
! kubectl and minikube configuration will be stored in /root
! To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:
*
  - sudo mv /root/.kube /root/.minikube $HOME
  - sudo chown -R $USER $HOME/.kube $HOME/.minikube
*
* This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
* Verifying Kubernetes components...
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" by default

```

参考：

https://github.com/AliyunContainerService/minikube

[Minikube - Kubernetes本地实验环境](https://developer.aliyun.com/article/221687)

[15分钟在笔记本上搭建 Kubernetes + Istio开发环境](https://developer.aliyun.com/article/672675)



https://mirrors.huaweicloud.com/

https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.3e221b11zwdZGz

### 安装 Kubernetes

#### 设置主机名

所有节点的主机名不允许相同，也不允许为 localhost

```bash
# 设置主机名
hostnamectl set-hostname centos-vm
# 查看主机名
hostnamectl
# 设置域名解析
echo "127.0.0.1   $(hostname)" >> /etc/hosts
```


#### 禁用交换分区

```bash
# 关闭所有的 Swap
swapoff -a
# 注释掉 Swap 行
cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

#### 配置防火墙

```bash
firewall-cmd --add-port=6443/tcp --add-port=2379-2380/tcp --add-port=10250-10252/tcp --add-port=443/tcp --add-port=4443/tcp --permanent
firewall-cmd --reload
```

- 6443 是 apiserver 的端口
- 2379-2380 是 etcd 服务端和客户端端口
- 10250 是 kubelet 端口
- 10251 是 kube-scheduler 端口
- 10252 是 kube-controller-manager 端口

防火墙需要开启的端口参考：https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports

#### 配置 containerd

```bash
# 检查 overlay 和 br_netfilter 模块是否被加载
lsmod | grep -E "overlay|br_netfilter"
# 若没有，则显式加载
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter


# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


## Configure containerd
mv /etc/containerd/config.toml /etc/containerd/config.toml.backup
containerd config default | sudo tee /etc/containerd/config.toml

sed -i "s#k8s.gcr.io#registry.aliyuncs.com/k8sxio#g"  /etc/containerd/config.toml
sed -i '/containerd.runtimes.runc.options/a\ \ \ \ \ \ \ \ \ \ \ \ SystemdCgroup = true' /etc/containerd/config.toml
sed -i "s#https://registry-1.docker.io#https://registry.cn-hangzhou.aliyuncs.com#g"  /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
```

参考：

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#letting-iptables-see-bridged-traffic

https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

https://github.com/kubernetes/website/blob/master/content/zh/docs/setup/production-environment/container-runtimes.md

https://kuboard.cn/install/install-k8s.html#%E5%AE%89%E8%A3%85containerd-kubelet-kubeadm-kubectl

#### 配置 apiserver 环境变量

所有的节点上配置以下环境变量和 apiserver 的域名

```bash
export MASTER_IP=192.168.17.134
export APISERVER_NAME=apiserver.demo
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
```

- **APISERVER_NAME** 不能是 master 的 hostname
- **APISERVER_NAME** 必须全为小写字母、数字、小数点，不能包含减号
- **POD_SUBNET** 所使用的网段不能与 ***master节点/worker节点*** 所在的网段重叠。该字段的取值为一个 [CIDR](https://kuboard.cn/glossary/cidr.html) 值，如果您对 CIDR 这个概念还不熟悉，请仍然执行 export POD_SUBNET=10.100.0.1/16 命令，不做修改

#### 配置 master

**初始化 master 节点**

```bash
export POD_SUBNET=10.100.0.1/16
curl -sSL https://kuboard.cn/install-script/v1.20.x/init_master.sh | sh -s 1.20.1
```

这里是使用 Kuboard 的脚本来安装，参考：[使用kubeadm安装kubernetes_v1.20.x](https://kuboard.cn/install/install-k8s.html#%E5%88%9D%E5%A7%8B%E5%8C%96-master-%E8%8A%82%E7%82%B9)

**检查 master 初始化结果**

```bash
# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态
watch kubectl get pod -n kube-system -o wide

# 查看 master 节点初始化结果
kubectl get nodes -o wide
```

**master 节点生成 token**

```bash
[root@centos-vm ~]# kubeadm token create --print-join-command
kubeadm join apiserver.demo:6443 --token rtji8k.i5zdrr1xs0tolckg     --discovery-token-ca-cert-hash sha256:befa1a2369ffdb2be73b857ec2964d8c63d9ac50efa49c140ae84e206a949723
```

#### 配置 worker

在 worker 节点执行上面输出的命令。建议加上 `--v=5` ，方便定位问题：

```bash
kubeadm join apiserver.demo:6443 --token rtji8k.i5zdrr1xs0tolckg     --discovery-token-ca-cert-hash sha256:befa1a2369ffdb2be73b857ec2964d8c63d9ac50efa49c140ae84e206a949723 --v=5
```

检查 worker 初始化结果

```bash
// 在 master 节点执行
[root@centos-vm ~]# kubectl get nodes
NAME        STATUS   ROLES                  AGE   VERSION
c8          Ready    <none>                 8d    v1.20.2
centos-vm   Ready    control-plane,master   9d    v1.20.2
```

#### 配置 kubectl

**kubectl 如果没配置好，执行的时候会报 `The connection to the server localhost:8080 was refused - did you specify the right host or port?` 错误**

如果当前节点是 master 节点，执行以下操作：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

如果当前节点是 worker 节点，则复制 master 节点的配置文件 /etc/kubernetes/admin.conf 或 `.kube/config` 到当前节点的 `$HOME/.kube/config` 。



查看 kubectl 与 apiserver 的版本：

```bash
[root@centos-vm ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:28:09Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.1", GitCommit:"c4d752765b3bbac2237bf87cf0b1c2e307844666", GitTreeState:"clean", BuildDate:"2020-12-18T12:00:47Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

- **Client Version**  表示的是 kubectl 的版本，如上所示是 v1.20.2；
- **Server Version** 表示的是 apiserver 的版本，如上所示是 v1.20.1。



查看集群状态：

```bash
[root@cstream ~]# kubectl cluster-info
Kubernetes control plane is running at https://apiserver.demo:6443
KubeDNS is running at https://apiserver.demo:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://apiserver.demo:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

#### 参考

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

https://kuboard.cn/install/install-k8s.html

https://docker_practice.gitee.io/zh-cn/kubernetes/setup/kubeadm.html

### 重置节点

```bash
kubeadm reset -f
rm -rf /etc/cni/net.d/
# 删除 kubelet 配置文件
rm -rf /etc/kubernetes/ /var/lib/kubelet 
# 删除 kubectl 配置文件（注意 kubectl 将无法连接到 master）
rm -rf $HOME/.kube/config
```

参考：https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset/

### 移除 worker 节点

```bash
# 这里假设 worker 节点名为 c8
kubectl drain c8 --delete-local-data --force --ignore-daemonsets
kubectl delete node c8
# 然后再重置 worker 节点
```

参考：https://blog.csdn.net/fanren224/article/details/86610799

### 解除 master 不能部署 Pod 限制

执行以下命令查看 master 的污点（下面的 `centos-vm` 是 master 的名称）：

```bash
[root@centos-vm ~]# kubectl describe nodes centos-vm |grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

上面的 **NoSchedule** 污点导致了 master 不能运行 Pod。



执行以下命令消除 master 的污点，让所有节点都能调度 Pod：

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```



如果想要让 master 恢复不能部署 Pod，执行以下命令（下面的 `centos-vm` 是 master 的名称）：

```bash
kubectl taint nodes centos-vm node-role.kubernetes.io/master=true:NoSchedule
```



参考：

1. https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#%E6%8E%A7%E5%88%B6%E5%B9%B3%E9%9D%A2%E8%8A%82%E7%82%B9%E9%9A%94%E7%A6%BB
2. https://docker_practice.gitee.io/zh-cn/kubernetes/setup/kubeadm.html#master-%E8%8A%82%E7%82%B9%E9%BB%98%E8%AE%A4%E4%B8%8D%E8%83%BD%E8%BF%90%E8%A1%8C-pod


### 安装 metrics-server

metrics-server 是 Kubernetes 中的一个重要组件，但是却不是内置组件。Kubernetes 许多特性都依赖于 metrics server。

安装了 metrics-server 后，就可以通过 `kubectl top nodes` / `kubectl top pods` 指令，查看节点/容器组的资源利用率。执行结果如下：

```bash
[root@centos-vm ~]# kubectl top nodes
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
c8          107m         5%     723Mi           19%
centos-vm   440m         22%    1424Mi          38%

[root@centos-vm ~]# kubectl top pods -n kube-system
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

## Cockpit

Cockpit 是 Linux 的 Web 控制台。Centos 8 在安装时可以选择安装该服务。

```bash
# 安装 cockpit:
yum install cockpit

# 启动 cockpit 服务:
systemctl enable --now cockpit.socket

# 打开防火墙:
firewall-cmd --permanent --zone=public --add-service=cockpit
firewall-cmd --reload
```

访问  https://ip-address:9090，用户名和密码就是系统用户的账号和密码

## Portainer

Portainer 是 Docker 的 Web 管理界面。

```bash
docker volume create portainer_data
# 旧版本
docker run -dp 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer:latest
# 新版本
docker run -dp 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
# 另外还需要确保开启 IPv4 转发
```

注意：端口9000是Portainer用于UI访问的通用端口。端口8000专门由边缘代理用于反向隧道功能。如果不打算使用边缘代理，则不需要公开端口8000

访问 http://localhost:9000，首次访问会提示设置用户名和密码（至少8位），分别设置为 admin 和 portainer。



参考：

1. https://www.portainer.io/installation/
2. http://www.senra.me/docker-management-panel-series-portainer/
3. https://docs.kvasirsg.com/centos-7/prefilight-configuration/how-to-enable-ip-forwarding

## Rancher

Rancher 开源的企业级 Kubernetes 管理平台。

### 通过 docker 安装

```bash
sudo docker run --privileged -d --name rancher --restart=unless-stopped -p 80:80 -p 443:443 -v /var/lib/rancher/:/var/lib/rancher/ rancher/rancher:stable
```

用户名默认为 admin，安装后会提示设置密码，这里设置为 admin


## MySQL

### 通过 docker 安装

```bash
// 创建存储卷
docker volume create mysql-data
// 使用 mysql 镜像（无需配置防火墙端口）
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=toor -v mysql-data -d mysql:8.0 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
// 使用 mysql-server 镜像（镜像最小，但没有 mysqlbinlog 工具）
docker run -d --name=mysql-server -p 3306:3306 -e MYSQL_ROOT_HOST=% -e MYSQL_ROOT_PASSWORD=toor -v mysql-data mysql/mysql-server --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
// 使用 mysql-80-centos7 镜像（能配置的环境变量更多，内置文本编辑器）
docker run -d --name mysql_database -e MYSQL_ROOT_PASSWORD=toor -e MYSQL_CHARSET=utf8mb4 -e MYSQL_COLLATION=utf8mb4_unicode_ci -v mysql-data -p 3306:3306 centos/mysql-80-centos7
```

参考：

https://hub.docker.com/_/mysql

https://hub.docker.com/r/mysql/mysql-server

https://hub.docker.com/r/centos/mysql-80-centos7

### 通过 yum 安装

#### Centos 8

Centos 8 的 AppStream 仓库下有 mysql-server.x86_64 的包

```bash
$ yum install mysql-server.x86_64
// 启动并设置开机启动
$ systemctl enable --now mysqld.service 
// 登录
$ mysql -uroot
```

#### Centos 7

Centos 7 默认不带 MySQL，但是有 MariaDB 替代。如果要安装 MySQL，需要添加仓库。

1. 从 https://dev.mysql.com/downloads/repo/yum/ 下载 rpm 仓库包

2. 安装 rpm 仓库包：`yum install mysql80-community-release-el7-{version-number}.noarch.rpm` 

3. 检查仓库是否已添加并开启：`yum repolist enabled | grep "mysql.*-community.*"`

4. 安装并登录 MySQL 服务器：
    ```bash
    $ yum install -y mysql-community-server.x86_64
    // 查看临时密码
    $ grep 'temporary password' /var/log/mysqld.log 
    // 登录
    $ mysql -uroot -p
    ```
    
    

### MySQL 主从同步搭建

在 Docker 中创建两个 MySQL 实例：

```bash
docker run --name mysql1 -p 33061:3306 -e MYSQL_ROOT_PASSWORD=toor -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
docker run --name mysql2 -p 33062:3306 -e MYSQL_ROOT_PASSWORD=toor -d mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

这两个 MySQL 实例的规划如下：

- mysql1 作为主库，其容器IP地址为：`172.17.0.2:3306`
- mysql2 作为从库，其容器IP地址为：`172.17.0.3:3306`

#### 配置主数据库

登录主库 MySQL，创建从库账户：

```sql
CREATE USER canal IDENTIFIED BY 'canal';
GRANT REPLICATION SLAVE ON *.* TO 'canal'@'%';
-- GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

主库添加以下配置：

```conf
[mysqld]
# 配置集群唯一标识
server-id=1
# 配置 binlog 位置（开启 binlog）
log-bin=/var/lib/mysql/binlog
# 配置要同步的数据库
binlog-do-db=canal
# 配置 binlog 格式
binlog_format=ROW
```

由于 mysql 容器没有文本编辑器，所以需要将容器的配置文件复制到宿主机，在宿主机配置完后，再复制回去，最后再重启容器：

```bash
docker cp mysql1:/etc/mysql/mysql.conf.d/mysqld.conf .
vi mysqld.conf
docker cp mysqld.conf mysql1:/etc/mysql/mysql.conf.d/
docker restart mysql1
```

重启完后，再次登录主库 MySQL，查看主库状态：

```bash
mysql> show master status;
+---------------+----------+------------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB     | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+------------------+------------------+-------------------+
| binlog.000001 |      154 | canal            |                  |                   |
+---------------+----------+------------------+------------------+-------------------+
1 row in set (0.00 sec)
```

记录 FIle 和 Position 字段的值，从库的配置需要用到。




#### 配置从数据库

从库添加以下配置：

```conf
[mysqld]
# 配置集群唯一标识（不能和主库一样）
server-id=2

### 以下配置酌情添加 ###
# 从库只读
read_only = 1
```

将配置文件拷贝到从库容器并重启，然后登录从库 MySQL，执行以下命令：

```bash
mysql> change master to master_host='172.17.0.2',master_port=3306,master_user='canal',master_password='canal',master_log_file='binlog.000001',master_log_pos=154;
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: canal
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000001
          Read_Master_Log_Pos: 1132
               Relay_Log_File: 57fe6c60c76f-relay-bin.000002
                Relay_Log_Pos: 1295
        Relay_Master_Log_File: binlog.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
......
```

其中 Slave_IO_Running 和 Slave_SQL_Running 都为 Yes，从库的配置才算成功。如果前者为 No，则有可能是 master_log_file 配置的文件名不对；如果是后者为 No，则有可能是同步时出错，具体问题具体分析和解决。



后续如果要暂停成为从库，执行以下命令：

```bash
mysql> stop slave;
```

如果从库的数据库有脏数据，主库的 binlog 同步到从库有可能会出错，尝试登录主库重置(清空)所有 binlog 日志：

```
mysql> reset master;
```



参考：

1. [借力 Docker ，三分钟搞定 MySQL 主从复制！](https://cloud.tencent.com/developer/article/1533955)
2. [MySQL的binlog日志](https://www.cnblogs.com/martinzhang/p/3454358.html)

### MySQL PXC 集群搭建

#### percona-xtradb-cluster:8.0

这里以创建3个 PXC 节点（分别为 pxc-node1、pxc-node2、pxc-node3）为例：

```bash
// 创建一个文件夹
mkdir -p ~/pxc-docker/config
// 在新建的文件夹中创建配置文件
cat > ~/pxc-docker/config/custom.cnf <<EOF
[mysqld]
ssl-ca = /cert/ca.pem
ssl-cert = /cert/server-cert.pem
ssl-key = /cert/server-key.pem

[client]
ssl-ca = /cert/ca.pem
ssl-cert = /cert/client-cert.pem
ssl-key = /cert/client-key.pem

[sst]
encrypt = 4
ssl-ca = /cert/ca.pem
ssl-cert = /cert/server-cert.pem
ssl-key = /cert/server-key.pem
EOF

// 再创建一个存放证书的文件夹
mkdir -m 777 -p ~/pxc-docker/cert
// 创建证书
docker run --name pxc-cert --rm -v ~/pxc-docker/cert:/cert \
percona/percona-xtradb-cluster:8.0 mysql_ssl_rsa_setup -d /cert

// 创建 docker 网络
docker network create pxc-network

// 启动第一个节点（注意在官方教程的基础上加了“-v ~/pxc-docker/cert:/cert”参数，不加会报错）
docker run -d \
  -e MYSQL_ROOT_PASSWORD=toor \
  -e CLUSTER_NAME=pxc-cluster \
  --name=pxc-node1 \
  --net=pxc-network \
  -v ~/pxc-docker/cert:/cert \
  -v ~/pxc-docker/config:/etc/percona-xtradb-cluster.conf.d \
  -p 33061:3306 \
  percona/percona-xtradb-cluster:8.0
// 启动其它节点（注意加了“-e CLUSTER_JOIN=pxc-node1”参数）
docker run -d \
  -e MYSQL_ROOT_PASSWORD=toor \
  -e CLUSTER_NAME=pxc-cluster \
  -e CLUSTER_JOIN=pxc-node1 \
  --name=pxc-node2 \
  --net=pxc-network \
  -v ~/pxc-docker/cert:/cert \
  -v ~/pxc-docker/config:/etc/percona-xtradb-cluster.conf.d \
  -p 33062:3306 \
  percona/percona-xtradb-cluster:8.0  
docker run -d \
  -e MYSQL_ROOT_PASSWORD=toor \
  -e CLUSTER_NAME=pxc-cluster \
  -e CLUSTER_JOIN=pxc-node1 \
  --name=pxc-node3 \
  --net=pxc-network \
  -v ~/pxc-docker/cert:/cert \
  -v ~/pxc-docker/config:/etc/percona-xtradb-cluster.conf.d \
  -p 33063:3306 \
  percona/percona-xtradb-cluster:8.0  
```



#### percona-xtradb-cluster:5.7

这里以创建3个 PXC 节点（分别为 node1、node2、node3）为例：

```bash
// 拉取镜像，注意不能是8.0，因为8.0安装方式变了
docker pull percona/percona-xtradb-cluster:5.7
// 镜像名称太长，重命名一下
docker tag percona/percona-xtradb-cluster:5.7 pxc
// 创建子网
docker network create --subnet=172.20.0.0/24 pxc-network
// 创建数据卷
docker volume create --name v1
docker volume create --name v2
docker volume create --name v3
// 启动第一个节点
docker run -d -p 33061:3306 -e MYSQL_ROOT_PASSWORD=toor -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=toor -v v1:/var/lib/mysql --name=node1 --network=pxc-network --ip 172.20.0.2 pxc
// 第一个节点启动完后再启动其它节点（注意参数加了“-e CLUSTER_JOIN=node1”）
docker run -d -p 33062:3306 -e MYSQL_ROOT_PASSWORD=toor -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=toor -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql --name=node2 --network=pxc-network --ip 172.20.0.3 pxc
docker run -d -p 33063:3306 -e MYSQL_ROOT_PASSWORD=toor -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=toor -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --name=node3 --network=pxc-network --ip 172.20.0.4 pxc
```



参考：

1. https://www.cnblogs.com/wanglei957/p/11819547.html
2. https://www.percona.com/doc/percona-xtradb-cluster/LATEST/install/docker.html#pxc-docker-container-running
3. https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/docker.html#pxc-docker-container-running

### 安装后配置

配置密码和远程访问权限：

```bash
$ CREATE USER 'root'@'%' IDENTIFIED BY 'toor';
$ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
$ FLUSH PRIVILEGES;
```

配置防火墙开放端口：

```bash
$ firewall-cmd --add-service=mysql --permanent
$ firewall-cmd --reload
$ firewall-cmd --list-service --permanent
```

查询已有用户：

```bash
mysql> use mysql;
mysql> select user,authentication_string,host from user;
```

删除已有用户：

```bash
mysql> drop user 'some_user@%'
```

如果忘记MySQL密码，执行以下操作：

```bash
$ vim /etc/my.cnf
// 在[mysqld]后加上如下语句
skip-grant-tables
$ systemctl restart mysqld
$ mysql -uroot
mysql> use mysql;
mysql> update user set authentication_string='' where user='root';
mysql> quit;
$ vim /etc/my.cnf
// 删除刚才添加的语句
skip-grant-tables
$ systemctl restart mysqld
$ mysql -uroot
mysql> ALTER USER 'root'@'%' IDENTIFIED BY 'pass4wOrd!';
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'pass4wOrd!';
```

参考：https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html

## MariaDB

### 通过 yum 安装

Centos 7 默认不带 MySQL，但是有 MariaDB 替代

```bash
$ yum install mariadb-server.x86_64
$ systemctl enable --now mariadb
```

安装后配置同 MySQL

## PostgreSQL

### 通过 docker-compose 安装

deploy-postgres.yml

```bash
# Use postgres/example user/password credentials
version: '3.1'

services:
  db:
    image: postgres:9.6
    container_name: postgres
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    volumes:
      - postgres-data
  adminer:
    image: adminer
    container_name: adminer
    restart: always
    ports:
      - 8080:8080

volumes:
  postgres-data:
```

参考：

https://hub.docker.com/_/postgres

https://hub.docker.com/_/adminer

### 通过 docker 安装

```bash
// 安装 Postgres
docker run -d --name postgres \
    -p 5432:5432 \
    -e "POSTGRES_USER=postgres" \
    -e "POSTGRES_PASSWORD=postgres" \
    -e "POSTGRES_DB=postgres" \
    -v /data/postgres:/var/lib/postgresql/data \
    postgres:9.6
```

参考：https://hub.docker.com/_/postgres

### 通过 yum 安装

```bash
$ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ yum install postgresql11 postgresql11-server
$ /usr/pgsql-11/bin/postgresql-11-setup initdb
$ systemctl enable --now postgresql-11
```

## Redis

### 通过 yum 安装

只支持 Centos 8

```bash
$ yum install redis.x86_64
$ systemctl enable --now redis.service
```

### 通过 docker 安装

```bash
// start a redis instance
$ docker run --name redis -p 6379:6379 -v redis-data -d redis:6-alpine
// or start with persistent storage
$ docker run --name redis -p 6379:6379 -v redis-data -d redis:6-alpine redis-server --appendonly yes

// 也可以使用另个镜像
$ docker run -d -p 6379:6379 -v redis-data --name redis_database -e REDIS_PASSWORD=strongpassword centos/redis-5-centos7
```

### 通过源码包安装

Centos 7 不带 Redis，只能通过源码包编译

```
$ curl -O http://download.redis.io/releases/redis-5.0.5.tar.gz
$ tar xvzf redis-5.0.5.tar.gz
$ cd redis-5.0.5
$ make distclean install 
$ yum install tcl.x86_64
$ make distclean test
```

### 安装后配置

默认 Redis 不支持远程连接，需要配置

```bash
$ vi /etc/redis.conf
bind 0.0.0.0
```

## RabbitMQ

### 通过 docker 安装

```bash
$ sudo docker run -d --name rabbitmq \
-p 5672:5672 -p 15672:15672 \
-e RABBITMQ_DEFAULT_USER=rabbitmq -e RABBITMQ_DEFAULT_PASS=rabbitmq \
-v rabbitmq-data rabbitmq:3-management-alpine

```

如要安装延迟队列插件，先下载好插件，放入 Dockerfile 文件所在目录，Dockerfile 内容如下：

```dockerfile
FROM rabbitmq:3.8.2-management
COPY ["rabbitmq_delayed_message_exchange-3.8.0.ez" , "/plugins/"]
RUN rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

构建镜像：

```bash
docker build -t rabbitmq:3.8.2-management .
```



### 通过 yum 安装

```bash
// import the new PackageCloud key that will be used starting December 1st, 2018 (GMT)
$ rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey

// import the old PackageCloud key that will be discontinued on December 1st, 2018 (GMT)
$ rpm --import https://packagecloud.io/gpg.key

// install erlang repository
$ curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash

// install rabbitmq repository
$ curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

$ yum install erlang.x86_64
$ yum install rabbitmq-server.noarch

// enable rabbitmq_management
$ rabbitmq-plugins enable rabbitmq_management

// autostart rabbitmq when system start
$ chkconfig rabbitmq-server on

// As an administrator, start and stop the server as usual:
$ /sbin/service rabbitmq-server start
$ /sbin/service rabbitmq-server stop

// download the example config file from website https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example
// and write it to file /etc/rabbitmq/rabbitmq.conf
// uncomment the line "loopback_users.guest = false" so you can use the guest user to login from anywhere on the network
// start the server and then visit http://HOST_IP:15672/
// the credential is below
// username:guest
// password:guest

// display application environment
$ rabbitmqctl environment

// system service logs can be inspected using
$ journalctl --system | grep rabbitmq


```



>  Starting RabbitMQ 3.7.16 on Erlang 22.0.5
>  Copyright (C) 2007-2019 Pivotal Software, Inc.
>  Licensed under the MPL.  See https://www.rabbitmq.com/
>  2019-06-22 03:06:36.608 [info] <0.219.0>
>  node           : rabbit@bogon
>  home dir       : /var/lib/rabbitmq
>  config file(s) : (none)
>  cookie hash    : 6TgR01uSocy6MRgruCqmLg==
>  log(s)         : /var/log/rabbitmq/rabbit@bogon.log
>       : /var/log/rabbitmq/rabbit@bogon_upgrade.log
>  database dir   : /var/lib/rabbitmq/mnesia/rabbit@bogon

## MongoDB

### 通过 docker 安装

```bash
$ sudo docker run --name mongo -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=mongo -e MONGO_INITDB_ROOT_PASSWORD=mongo \
-v /data/mongo:/data/db \
mongo:latest
```

### 通过 docker-compose 安装

deploy-mongo.yml

```yaml
# Use mongo/mongo as user/password credentials
version: '3.1'

services:
  mongo:
    image: mongo
    container_name: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongo
      MONGO_INITDB_ROOT_PASSWORD: mongo
    networks:
      - mongo-net
    volumes:
      - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    container_name: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: mongo
      ME_CONFIG_MONGODB_ADMINPASSWORD: mongo
      ME_CONFIG_BASICAUTH_USERNAME: mongo #开启 Basic 登录
      ME_CONFIG_BASICAUTH_PASSWORD: mongo
    depends_on:
      - mongo
    networks:
      - mongo-net

networks:
  mongo-net:
volumes:
  mongo-data:
```

参考：

https://hub.docker.com/_/mongo

https://hub.docker.com/_/mongo-express

### 通过 yum 安装

```bash
$ vim /etc/yum.repos.d/mongodb-org-4.0.repo
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

$ yum install -y mongodb-org

$ systemctl enable --now mongod.service
// or
$ chkconfig mongod on
$ service mongod start
$ service mongod stop

$ vim /etc/mongod.conf
net:
  port: 27017
  bindIp: 0.0.0.0 或 bindIpAll
```

## Elasticsearch

### 通过 docker 安装

安装 elasticsearch

```bash
// 安装 elasticsearch
$ docker network create elasticsearch
$ docker volume create elastic-data
$ docker run --name elasticsearch \
-p 9200:9200 -p 9300:9300 \
--net elasticsearch \
-e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "http.cors.enabled=true" \
-e "http.cors.allow-origin=*" \
-e "http.cors.allow-headers=X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization" \
-e "http.cors.allow-credentials=true" \
-v elastic-data:/usr/share/elasticsearch/data \
-d elasticsearch:7.4.1
```

安装 kibana 和 logstash

```bash
// 安装 kibana
$ docker run --name kibana -p 5601:5601 --net elasticsearch -d kibana:7.4.1
// 安装 logstash
$ docker run --rm -it -v ~/pipeline/:/usr/share/logstash/pipeline/ --name logstash -d logstash:7.4.2 
// 方法一，提供 logstash.conf 文件
$ docker run --rm -it -v ~/settings/:/usr/share/logstash/config/ --name logstash -d logstash:7.4.2 
// 方法二，提供 logstash.yml 文件
$ docker run --rm -it -v ~/settings/logstash.yml:/usr/share/logstash/config/ --name logstash -d logstash.yml logstash:7.4.2
```



参考：

https://hub.docker.com/_/elasticsearch

https://hub.docker.com/_/kibana

### 监控工具

```bash
// 这个监控工具需要保证ES在同一Docker Network才能使用 “http://elasticsearch:9200” 连接
$ docker run -p 9800:9800 --net elasticsearch --name elastichd -d containerize/elastichd
// 这两款监控工具只能通过 http://centos-vm://9200 打开（centos-vm是ES服务在宿主机的域名，即虚拟机在宿主机的域名，因为ES服务部署在虚拟机上）
$ docker run -p 9100:9100 --net elasticsearch --name elasticsearch-head -d mobz/elasticsearch-head:5
$ docker run -p 1358:1358 --net elasticsearch --name dejavu -d appbaseio/dejavu
```

参考：

https://hub.docker.com/r/mobz/elasticsearch-head

https://github.com/360EntSecGroup-Skylar/ElasticHD 

https://hub.docker.com/r/appbaseio/dejavu



### logstash.conf 配置文件示例

```conf
input {
	 stdin { }
     beats {
         port => 5044
         type => beats
     }
     tcp {
         port => 5000
         type => syslog
     }
 }
 filter {
 }
 output {
     elasticsearch { 
         hosts => ["elasticsearch:9200"]
     }
     stdout { codec => rubydebug }
 }
```

参考：https://www.elastic.co/guide/en/logstash/current/docker-config.html



### 通过 Docker Compose 安装集群

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.4.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

启动后访问

```bash
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```



参考：https://www.elastic.co/guide/en/elasticsearch/reference/7.4/docker.html

## InfluxDB

### 通过 docker 安装

```bash
docker run --name influxdb -p 8086:8086 influxdb:2.0.4
```

启动后第一次访问 [localhost:8086](http://localhost:8086/)，会提示设置**Username**、**Password**、**Organization Name**、**Bucket Name**，这里统一设置为 influxdb。

参考：https://docs.influxdata.com/influxdb/v2.0/get-started

## GitLab

gitlab-ce 的镜像大概 1GB 左右，而 Gogs 为 100MB 左右，占用的CPU资源也比 Gogs 高。

### 通过 docker 安装

```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /data/gitlab/config:/etc/gitlab \
  --volume /data/gitlab/logs:/var/log/gitlab \
  --volume /data/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

只能通过上面配置的 hostname 来访问 gitlab，即通过 http://gitlab.example.com 来访问，所以需要本地配置好 hosts。

首次登录会提示修改密码，这里修改为 gitlabpass，默认用户名为 root。

### 通过 docker-compose 安装

deploy-gitlab.yml

```bash
version: '3.1'

services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.example.com'
        gitlab_rails['gitlab_shell_ssh_port'] = 8022
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '8022:22'
    volumes:
      - '/data/gitlab/config:/etc/gitlab'
      - '/data/gitlab/logs:/var/log/gitlab'
      - '/data/gitlab/data:/var/opt/gitlab'

```



参考：https://docs.gitlab.com/omnibus/docker/#set-up-the-volumes-location

## Gogs

### 硬件要求

至少双核CPU、512MB内存。Gogs 要求安装 MySQL、PostgreSQL、SQLite3、MSSQL 或 TiDB。

### 通过 docker 安装

```
mkdir -p /var/gogs
docker run -d --rm --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs
// 浏览器打开 Gogs 的 Web 页面，一定要设置好域名，不要设置为 localhost，会影响 git clone
// 建议设置域名为 gogs.example.com，应用URL设置为 http://gogs.example.com:10080/
// 具体配置也可以到容器里更改
```

参考：

https://github.com/gogs/gogs

https://github.com/gogs/gogs/tree/main/docker

## Gitea

### 通过 docker-compose 安装

deploy-gitea.yml

```bash
version: '2'
services:
  web:
    image: gitea/gitea:1.12.4
    volumes:
      - /data/gitea:/data
    ports:
      - "3000:3000"
      - "8022:22"
    depends_on:
      - db
    restart: always
  db:
    image: mariadb:10
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
    volumes:
      - /data/gitea-db/:/var/lib/mysql
```

安装后的配置跟 gogs 一样。系统界面也跟 gogs 相似，应该是基于 gogs。

参考：

https://hub.docker.com/r/gitea/gitea

https://docs.gitea.io/en-us/install-with-docker/

## Jenkins

### 通过 docker 安装

1. ```bash
   $ docker volume create jenkins-data
   $ docker run \
     -u root \
     -d \
     -p 8080:8080 \
     -p 50000:50000 \
     -v jenkins-data:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     --name jenkins-blueocean \
     jenkinsci/blueocean:
   
   ```
   
3. note the admin password dumped on log: d844e5d059554c85b1012f942109226c

4. open a browser on http://localhost:8080 or http://localhost:8080/blue

## Nexus3

Nexus3 用于搭建Maven私有仓库

### 通过 docker 安装

```bash
$ docker volume create --name nexus-data
$ docker run -d -p 8083:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3:3.23.0
$ curl http://localhost:8083/
```

默认用户名为 admin，密码存在 admin.password 文件中

```bash
cat /var/lib/docker/volumes/nexus-data/_data/admin.password
```

首次登录会提示修改密码，这里将密码改为 `admin` 。



安装后配置：

1. 开启**匿名访问**私有仓库。这样不用用户名和密码也可以下载私仓的 jar 包。
2. maven-central 要配置阿里云镜像。默认是从 maven 官方下载，速度很慢。

参考：https://hub.docker.com/r/sonatype/nexus3

## SonarQube

通过

## Docker Registry

### 通过 docker 安装

```bash
$ docker volume create --name registry-data
$ docker run -d -p 5000:5000 --name registry -v registry-data:/var/lib/registry registry:2.7
```

使用示例：

```
$ docker pull ubuntu
$ docker tag ubuntu localhost:5000/ubuntu
$ docker push localhost:5000/ubuntu
$ curl localhost:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

PS：目前不知道 Docker 私仓的镜像删除方法，使用时请慎用。

配置 Docker 允许非 `HTTPS` 方式推送镜像，编辑 `/etc/docker/daemon.json` 中为如下内容：

```json
{
  "registry-mirror": [
    "https://dockerhub.azk8s.cn"
  ],
  "insecure-registries": [
    "192.168.199.100:5000"
  ]
}
```



参考：

https://hub.docker.com/_/registry

https://docs.docker.com/registry/deploying/



## Nginx


### 通过 yum 安装

参考：https://nginx.org/en/linux_packages.html

### 通过源码包安装

```bash
// 安装必要的依赖
yum install -y pcre-devel.x86_64 openssl-devel.x86_64
// 配置
./configure --prefix=/usr/local/nginx --with-debug
// 编译和安装
make & make install
```

参考：https://nginx.org/en/docs/configure.html

### 通过 docker-compose 安装

创建文件 deploy-nginx.yml

```yaml
nginx:
  image: nginx:1.17
  container_name: nginx
  restart: always
  ports:
  - 8080:80
  - 443:443
  volumes:
  - /data/nginx/html:/usr/share/nginx/html #静态资源根目录挂载
  - /data/nginx/log:/var/log/nginx #日志目录挂载
  - /data/nginx/nginx.conf:/etc/nginx/nginx.conf #配置目录挂载（需要先创建配置文件）
```

### Dockerfile 构建 Tengine 镜像

默认 Nginx 和 Tengine 未编译 ngx_http_proxy_connect_module 模块，不支持 HTTPS 正向代理，所以这里手动编译一个 Tengine 镜像。

```dockerfile
FROM alpine:3.12

ENV TENGINE_VERSION=2.3.2

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && apk update && apk add --no-cache --virtual .build-deps gcc libc-dev make linux-headers gd-dev geoip-dev curl && apk add --no-cache --virtual .nginx-deps pcre-dev openssl-dev zlib-dev libxslt-dev && curl -L "http://tengine.taobao.org/download/tengine-$TENGINE_VERSION.tar.gz" -o tengine.tar.gz && mkdir -p /usr/src && tar -xzC /usr/src -f tengine.tar.gz && rm tengine.tar.gz && cd /usr/src/tengine-$TENGINE_VERSION && ./configure --add-module=./modules/ngx_http_proxy_connect_module && make -j$(getconf _NPROCESSORS_ONLN) && make install && ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx && apk del .build-deps && ln -sf /dev/stdout /usr/local/nginx/logs/access.log && ln -sf /dev/stderr /usr/local/nginx/logs/error.log

EXPOSE 443 80

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
```

构建镜像：

```
docker build -t tengine .
```



参考：

https://hub.docker.com/layers/axizdkr/tengine/latest/images/sha256-379c26e6d4d45760853c6af9fdc3a59e37436383b8e3b6e844340e41b9c477bf?context=explore

https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile

## OpenResty

### 通过 yum 安装

```bash
# add the yum repo:
wget https://openresty.org/package/centos/openresty.repo
sudo mv openresty.repo /etc/yum.repos.d/

# update the yum index:
sudo yum check-update

# 安装 openresty
sudo yum install -y openresty
# 安装命令行工具resty，包管理工具opm
sudo yum install -y openresty-resty openresty-opm 
# 安装 restydoc
sudo yum install -y openresty-doc
```

OpenResty 的包源地址：https://opm.openresty.org/

参考：

https://openresty.org/cn/linux-packages.html

https://github.com/openresty/opm

### 通过源码包安装

安装前的准备

```bash
yum install pcre-devel openssl-devel gcc curl
```

编译安装

```bash
# 解压
tar -xzvf openresty-{VERSION}.tar.gz
# 配置
./configure --prefix=/usr/local/openresty
# 编译安装
make && make install

# 多核 make 加 -j 选项，如下
make -j && make install
```

参考：https://openresty.org/cn/installation.html

### 通过 docker 安装

```bash
docker run -dp 80:80 --name openresty openresty/openresty:alpine
```

## Kong

Kong 是一个基于 OpenResty 的 API 网关，支持 Postgres 和 Cassandra 数据库。

### 通过 docker 安装

```bash
// 安装 Postgres
docker run -d --name kong-database \
    -p 5432:5432 \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_PASSWORD=kong" \
    -e "POSTGRES_DB=kong" \
    -v /data/postgres:/var/lib/postgresql/data \
    postgres:9.6

// 初始化数据库
docker run --rm \
    --link kong-database:kong-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_PG_USER=kong" \
    -e "KONG_PG_PASSWORD=kong" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong kong migrations bootstrap
    
// 安装 Kong
docker run -d --name kong \
    --link kong-database:kong-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_PG_USER=kong" \
    -e "KONG_PG_PASSWORD=kong" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong
```

如果更改了 Kong 的配置，可以使用以下命令重新加载 Kong

```bash
$ docker exec -it kong kong reload
```

测试

```bash
# 创建服务
curl -i -X POST \
--url http://localhost:8001/services/ \
--data 'name=baidu' \
--data 'url=https://www.baidu.com'

# 创建路由
curl -i -X POST \
--url http://localhost:8001/services/baidu/routes \
--data 'name=baidu' \
--data 'paths[]=/v1/baidu'

# 访问测试，确认路由规则
curl -i -X GET \
    --url http://localhost:8000/v1/baidu

# 关联插件到路由对象实例baidu
curl -i -X POST \
    --url http://localhost:8001/routes/baidu/plugins \
    --data "name=key-auth" \
    --data "config.key_names=apikey" 

# 创建消费者
curl -d "username=customer" http://localhost:8001/consumers/

# 创建消费者密钥
curl -X POST http://localhost:8001/consumers/customer/key-auth -d ''

# 查看并获得密钥
curl http://localhost:8001/consumers/customer/key-auth

# 消费者使用密钥访问
curl -i -X GET \
    --url http://localhost:8000/v1/baidu \
    --header "apikey: 5b2fCdjk2FcPhp5aVN45umjhml2kzPgE"	
```



安装 Konga

```bash
// 初始化数据库（其中 172.17.0.3 为 postgres 数据库的 IP 地址）
docker run --rm pantsel/konga:latest \
-c prepare -a postgres -u postgres://kong:kong@172.17.0.3:5432/konga

// 安装 Konga
docker run -d --name konga -p 1337:1337 \
--link kong-database \
--link kong \
-e "TOKEN_SECRET=somerandomstring" \
-e "DB_ADAPTER=postgres" \
-e "DB_HOST=kong-database" \
-e "DB_USER=kong" \
-e "DB_PASSWORD=kong" \
-e "DB_DATABASE=konga" \
-e "NODE_ENV=production" \
pantsel/konga

// 浏览器打开 konga 的 Web 页面，然后注册并登录账号，用户名和密码为limondar，
// 接着创建 Kong 的连接，Kong 的管理地址为：http://kong:8001
```

参考：

https://hub.docker.com/_/kong

https://hub.docker.com/r/pantsel/konga

https://github.com/pantsel/konga

## Nacos

### 通过源码包安装

```bash
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
sh startup.sh -m standalone
```

参考：https://nacos.io/zh-cn/docs/quick-start.html

### 通过 docker 安装

单机嵌入式数据库模式：

```bash
$ docker run --name nacos-standalone -e MODE=standalone -e PREFER_HOST_MODE=hostname -p 8848:8848 -d nacos/nacos-server:latest
```

或

```bash
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
export NACOS_VERSION=latest
docker-compose -f example/standalone-derby.yaml up -d
```

### 测试

命令行执行以下命令：

```bash
# 服务注册
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'

# 服务发现
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'

# 发布配置
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"

# 获取配置
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
```



Nacos 控制台地址：http://127.0.0.1:8848/nacos/，默认用户名和密码为：nacos/nacos

Prometheus 地址：http://127.0.0.1:9090/graph

Grafana 地址：http://127.0.0.1:3000，默认用户名和密码为：admin/admin



参考：

https://github.com/nacos-group/nacos-docker/blob/master/README_ZH.md

https://nacos.io/zh-cn/docs/monitor-guide.html

## Seata

### 通过 docker 安装

```bash
$ docker run --name seata-server -p 8091:8091 seataio/seata-server:latest -d
```

参考：https://hub.docker.com/r/seataio/seata-server

## Consul

### 通过 docker 安装

```bash

```



参考：

[Docker 部署 Consul，多数据中心](https://www.jianshu.com/p/df3ef9a4f456)

[Docker中创建Consul集群](https://www.jianshu.com/p/067154800683)

## Zipkin

### 通过 docker 安装

```bash
$ docker run -d -p 9411:9411 openzipkin/zipkin --name zipkin
```

参考：https://github.com/openzipkin/zipkin/tree/master/docker

## Zookeeper

### 通过 docker 安装

```bash
docker run --name zookeeper -p 2181:2181 -d zookeeper
```

### 通过 docker-compose 安装

 `docker-compose -f zookeeper.yml up -d`

示例 `zookeeper.yml` 文件：

```yaml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
```



参考：https://hub.docker.com/_/zookeeper

## Kafka

### 通过 docker-compose 安装

kafka-compose.yml

```yaml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: localhost
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_BROKER_ID: 1
      KAFKA_CREATE_TOPICS: "stream-in:1:1,stream-out:1:1"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # 与Docker守护进程通信
      - /etc/localtime:/etc/localtime:ro # 容器时间同步宿主机的时间
      - /etc/timezone:/etc/timezone:ro # 容器时区同步宿主机的时区
```



参考：

1. https://hub.docker.com/r/wurstmeister/kafka
2. https://kafka.apache.org/quickstart
3. https://www.jianshu.com/p/ac03f126980e

## Prometheus

### 配置示例

prometheus.yml

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 1m
scrape_configs:
- job_name: prometheus
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost:9090
```

## FastDFS

建议不要在桥接网络中运行 FastDFS，因为 tracker 服务会返回 storage 服务在桥接网络的 ip，如果 fastdfs 客户端不在该桥接网络，就会无法连接上 storage 服务

### 使用 season/fastdfs 镜像

**启动 tracker 服务**

```bash
sudo docker run -ti -d --name tracker -v ~/tracker_data:/fastdfs/tracker/data --net=host season/fastdfs tracker
```

**启动 storage 服务**

```bash
sudo docker run -ti -d --name storage -v ~/storage_data:/fastdfs/storage/data -v ~/store_path:/fastdfs/store_path --net=host -e TRACKER_SERVER="<Tracker服务的IP地址>:22122" season/fastdfs storage
```

由于 tracker 服务连接的是 host 网络，所以"<Tracker服务的IP地址>"这里填主机的 IP 地址即可。host 网络不支持使用域名

### 使用 ygqygq2/fastdfs-nginx 镜像

该镜像包含 nginx 服务，方便访问和下载上传的文件

```bash
sudo docker run -dit --network=host -p 22122:22122 -p 80:80 --name tracker -v /var/fdfs/tracker:/var/fdfs ygqygq2/fastdfs-nginx:latest tracker
sudo docker run -dit --network=host -p 23000:23000 -p 8080:8080 -p 8888:8888 --name storage0 -e TRACKER_SERVER="<Tracker服务的IP地址>:22122" -v /var/fdfs/storage0:/var/fdfs ygqygq2/fastdfs-nginx:latest storage
```

端口关系：

22122：tracker 服务的端口
80：tracker 服务 nginx 端口（负载均衡 storage 的 nginx 服务）
23000：storage 服务的端口
8080：storage 容器中 HTTP 文件访问端口（跟Nginx保持一致）

### 配置防火墙

```bash
sudo firewall-cmd --add-port=22122/tcp --add-port=23000/tcp --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```



参考：https://hub.docker.com/r/season/fastdfs

## [goproxy](https://gitee.com/snail/proxy)

安装 goproxy 高性能代理

```bash
curl -L http://mirrors.host900.com:9090/snail007/goproxy/install_auto.sh | bash  
>>> installing ...

>>> install done, thanks for using snail007/goproxy free_9.5

>>> install path /usr/bin/proxy

>>> configuration path /etc/proxy

>>> uninstall just exec : rm /usr/bin/proxy && rm -rf /etc/proxy

>>> How to using? Please visit : https://snail007.github.io/goproxy/manual/zh/

```

安装 goproxy 的 管理面板 ProxyAdmin 

```bash
curl -L http://mirrors.host900.com:9090/snail007/proxy_admin_free/install_auto.sh | bash  
>>> install done, thanks for using snail007/proxy-admin

>>> install path /usr/local/bin/proxy-admin

>>> configuration path /etc/gpa

>>> uninstall just exec : proxy-admin uninstall

>>> please visit : http://YOUR_IP:32080/ username: root, password: 123

>>> How to using? Please visit : https://snail007.github.io/goproxy/manual/zh/

```

安装成功后，打开浏览器访问：[http://127.0.0.1:32080](http://127.0.0.1:32080/) , 首次默认账号是root，密码是123，登录后记得第一时间修改。

## xxl-job-admin

### 通过 docker 安装

```bash
$ docker run -e PARAMS="--spring.datasource.username=root --spring.datasource.password=toor --spring.datasource.url=jdbc:mysql://centos-vm:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin -d xuxueli/xxl-job-admin:2.2.0
```



## 踩坑记录

### *查看系统硬件配置

### 查看系统版本

- `uname` 命令查看内核版本（通用）

    在 Centos 8 下执行结果：
    
    ```bash
    [root@localhost ~]# uname -a
Linux localhost.localdomain 4.18.0-193.14.2.el8_2.x86_64 #1 SMP Sun Jul 26 03:54:29 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
    ```

    其中：
    
    - Linux：内核名称。这里表示 Linux 内核名称
    - localhost.localdomain：本机域名。本机域名是可以通过 `/etc/hostname` 文件或 `hostnamectl` 命令修改
    - `4.18.0-193.14.2.el8_2.x86_64`：内核发行版本。这里是基于 RedHat 8 的 Linux 内核
    - `#1 SMP Sun Jul 26 03:54:29 UTC 2020`：内核发行版的发布时间
    - x86_64：第一个 x86_64 表示机器硬件名
    - x86_64：第二个 x86_64 表示处理器类型。这里是64位处理器
    - x86_64：第三个 x86_64 表示硬件平台
    - GNU/Linux：操作系统名。这里表示 Linux 系统

- `cat /etc/os-release` 命令查看系统发行版（RedHat、Arch 系统）

  ```bash
  [root@localhost ~]# cat /etc/os-release
  NAME="CentOS Linux"
  VERSION="8 (Core)"
  ID="centos"
  ID_LIKE="rhel fedora"
  VERSION_ID="8"
  PLATFORM_ID="platform:el8"
  PRETTY_NAME="CentOS Linux 8 (Core)"
  ANSI_COLOR="0;31"
  CPE_NAME="cpe:/o:centos:centos:8"
  HOME_URL="https://www.centos.org/"
  BUG_REPORT_URL="https://bugs.centos.org/"
  
  CENTOS_MANTISBT_PROJECT="CentOS-8"
  CENTOS_MANTISBT_PROJECT_VERSION="8"
  REDHAT_SUPPORT_PRODUCT="centos"
  REDHAT_SUPPORT_PRODUCT_VERSION="8"
  ```

- `cat /etc/issue` 命令查看系统发行版（RedHat 系统没用）

  在 Centos 8 下执行结果

  ```bash
  [root@localhost ~]# cat /etc/issue
  \S
  Kernel \r on an \m
  ```

  在 Ubuntu 20 下执行结果

  ```bash
  ubuntu@ubuntu:~$ cat /etc/issue
  Ubuntu 20.04.1 LTS \n \l
  ```

- `lsb_release -a` 命令查看系统发行版（Ubuntu 系统）

  ```bash
  ubuntu@ubuntu:~$ lsb_release -a
  No LSB modules are available.
  Distributor ID: Ubuntu
  Description:    Ubuntu 20.04.1 LTS
  Release:        20.04
  Codename:       focal
  ```

  该数据其实是来自 `/etc/lsb-release` 文件

  ```bash
  ubuntu@ubuntu:~$ cat /etc/lsb-release
  DISTRIB_ID=Ubuntu
  DISTRIB_RELEASE=20.04
  DISTRIB_CODENAME=focal
  DISTRIB_DESCRIPTION="Ubuntu 20.04.1 LTS"
  ```

  

### 环境变量设置

相关命令：

- `export` 命令显示当前系统定义的所有环境变量
- `echo $PATH` 命令输出当前的 `PATH` 环境变量的值
- `export PATH=$PATH:$HOME/bin` 命令设置环境变量 `PATH`

相关文件：

- /etc/profile 和 /etc/profile.d/：系统级环境变量和启动程序定义
- /etc/bashrc、/etc/bash.bashrc：系统级函数和别名定义
- ~/.bash_profile、~/.profile：用户级环境变量和启动程序定义
- ~/.bashrc：用户级函数和别名定义

参考：[Linux环境变量配置全攻略](https://www.cnblogs.com/youyoui/p/10680329.html)

### 设置时区

- 使用 `timedatectl set-timezone Asia/Shanghai`

- 使用 `tzselect`

```
[root@localhost ~]# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.
#? 5
Please select a country.
 1) Afghanistan           18) Israel                35) Palestine
 2) Armenia               19) Japan                 36) Philippines
 3) Azerbaijan            20) Jordan                37) Qatar
 4) Bahrain               21) Kazakhstan            38) Russia
 5) Bangladesh            22) Korea (North)         39) Saudi Arabia
 6) Bhutan                23) Korea (South)         40) Singapore
 7) Brunei                24) Kuwait                41) Sri Lanka
 8) Cambodia              25) Kyrgyzstan            42) Syria
 9) China                 26) Laos                  43) Taiwan
10) Cyprus                27) Lebanon               44) Tajikistan
11) East Timor            28) Macau                 45) Thailand
12) Georgia               29) Malaysia              46) Turkmenistan
13) Hong Kong             30) Mongolia              47) United Arab Emirates
14) India                 31) Myanmar (Burma)       48) Uzbekistan
15) Indonesia             32) Nepal                 49) Vietnam
16) Iran                  33) Oman                  50) Yemen
17) Iraq                  34) Pakistan
#? 9
Please select one of the following time zone regions.
1) Beijing Time
2) Xinjiang Time
#? 1

The following information has been given:

        China
        Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:      Fri May 22 22:42:54 CST 2020.
Universal Time is now:  Fri May 22 14:42:54 UTC 2020.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
        TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai

```

### 根分区扩容

1. 先用 `fdisk` 给根分区扩容

   ```bash
   fdisk /dev/sda
   > p #1.打印分区表
   > d #2.删除分区（分区也行）
   > n #3.新增分区
   > t #4.修改分区类型为8e(8e表示LVM，默认是不创建LVM分区)
   > w #5.写分区表
   > q #6.退出
   partprobe # 7.或重启机器
   ```
   
2. 然后给物理卷PV扩容

   ```bash
   pvresize /dev/sda2
   ```

3. 再给逻辑卷LV扩容

   ```bash
   lvextend /dev/centos/root /dev/sda2
   ```

4. 最后给文件系统扩容

   - 如果是 XFS 文件系统（Centos 7 的默认文件系统）
   
       ```bash
       xfs_growfs /dev/mapper/centos-root
       ```

   - 如果是 Ext4 文件系统（Centos 6 的默认文件系统）

       ```bash
       // 先检查并修复文件系统错误
       e2fsck -f /dev/sda2 
       // 再扩容
       resize2fs /dev/mapper/centos-root 
       // 或执行这个命令
       resize2fs /dev/sda2
       // 如果在救援模式下，需要重新挂载根目录为读写模式才能扩容
       mount -o remount,rw /
       ```
   
   

### 设置网卡自动启动

Centos 安装后本地网卡不会自动启动并获取IP地址，需要使用 `nmtui` 来配置

参考：https://wiki.centos.org/FAQ/CentOS7#Why_does_my_Ethernet_not_work_unless_I_log_in_and_explicitly_enable_it.3F

### NetworkManager 导致网卡无法获取 IP

NetworkManager 异常导致网卡设备处以 unmanaged 状态，且无法恢复为 managed 状态，可以尝试以下操作：

```bash
systemctl stop network-manager.service(Ubuntu)
systemctl stop NetworkManager.service(Centos)
// 清除 NetworkManager 异常状态
rm -f /var/lib/NetworkManager/NetworkManager.state
systemctl start network-manager
```

### 防火墙开放端口

```bash
# 开放端口
firewall-cmd --add-port=80/tcp --zone=public --permanent
# 重新加载配置
firewall-cmd --reload
# 查询是否开放
firewall-cmd --query-port=80/tcp
# 查询所有开放端口
firewall-cmd --list-port
```

### 防火墙端口映射

使用 firewall-cmd 配置：

```bash
#添加端口映射（其中将本地6379端口映射到172.17.0.2容器的6379端口）
firewall-cmd --add-forward-port=port=6379:proto=tcp:toport=6379:toaddr=172.17.0.2
#删除端口映射
firewall-cmd --remove-forward-port=port=6379:proto=tcp:toport=6379:toaddr=172.17.0.2
```

使用 iptables 配置：

```bash
# 允许流量进入容器（该规则链在 FORWARD 规则链中）
iptables -A DOCKER -p tcp -d 172.17.0.2 --dport 6379 -j ACCEPT
# 添加DNAT，外部网络才能访问容器（该规则链在 PREROUTING 规则链中）
iptables -t nat -A DOCKER -p tcp --dport 6379 -j DNAT --to-destination 172.17.0.2:6379
# 添加MASQUERADE，容器才能访问外部网络
iptables -t nat -A POSTROUTING -p tcp -s 172.17.0.2 -d 172.17.0.2 --dport 6379 -j MASQUERADE
# 删除上面的配置
iptables -D DOCKER <number>
iptables -t nat -D DOCKER <number>
iptables -t nat -D POSTROUTING <number>
```



### 为什么 Centos7 不带 ipconfig 和 netstat 命令？

Centos 7 默认不安装 `net-tools` 工具包，推荐使用 `ip` 和 `ss` 命令替代。

参考：[What have you done with ifconfig/netstat?](https://wiki.centos.org/FAQ/CentOS7#What_have_you_done_with_ifconfig.2Fnetstat.3F)

### 查看端口占用情况

```bash
netstat -tunlp | grep 80
# 或
ss -tunpl | grep 80
```

netstat 的选项如下：

- -t --tcp：只显示 TCP
- -u --udp：只显示 UDP
- -a --all：显示所有，即 TCP 和 UDP
- -l --listening：只显示监听状态的连接
- -n --numeric：全部显示数字，不要解析为名称
- -p --programs：显示 Socket 的程序名称和 PID

