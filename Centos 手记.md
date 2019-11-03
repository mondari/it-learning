[TOC]



# Centos 配置

## 用户名和密码

username: root
password: toor

## .inputrc

```
// 设置 bash 大小写不敏感
$ echo "set completion-ignore-case on">>~/.inputrc
```

## 安装的 yum 包

yum groups install Development Tools（内含 gcc, make, git, cmake）
net-tools.x86_64（内含 ifconfig, netstat, route）
bash-completion.x86_64
mlocate.x86_64
vim (默认没有 vim)
java-1.8.0-openjdk.x86_64
java-1.8.0-openjdk-devel.x86_64（内含 Java 诊断工具）

## neofetch

```shell
curl -o /etc/yum.repos.d/konimex-neofetch-epel-7.repo https://copr.fedorainfracloud.org/coprs/konimex/neofetch/repo/epel-7/konimex-neofetch-epel-7.repo

yum install neofetch
```



## Docker

### 安装 docker-ce

```shell
// Uninstall old versions
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
// Install required packages                  
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
// Set up the stable repository.  
// 这里改用阿里云的 docker-ce 仓库
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
    
// Install the latest version of Docker Engine - Community and containerd    
$ sudo yum install docker-ce docker-ce-cli containerd.io

// 将 Docker 设为开机启动
$ sudo systemctl enable docker

// Start Docker
$ sudo systemctl start docker

// Verify
$ sudo docker run hello-world
```

### 设置 Docker Hub 镜像加速器

```shell
sudo mkdir -p /etc/docker
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
sudo systemctl daemon-reload
sudo systemctl restart docker
// 验证
docker info
```

### Docker Hub 镜像加速器列表

| 镜像加速器                                                   | 镜像加速器地址                       | 专属加速器[？](#) | 其它加速[？](#)                                              |
| ------------------------------------------------------------ | ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| [Docker 中国官方镜像](https://links.jianshu.com/go?to=https%3A%2F%2Fdocker-cn.com%2Fregistry-mirror) | `https://registry.docker-cn.com`     |                   | Docker Hub                                                   |
| [DaoCloud 镜像站](https://links.jianshu.com/go?to=https%3A%2F%2Fdaocloud.io%2Fmirror) | `http://f1361db2.m.daocloud.io`      | 可登录，系统分配  | Docker Hub                                                   |
| [Azure 中国镜像](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FAzure%2Fcontainer-service-for-azure-china%2Fblob%2Fmaster%2Faks%2FREADME.md%2322-container-registry-proxy) | `https://dockerhub.azk8s.cn`         |                   | Docker Hub、GCR、Quay                                        |
| [科大镜像站](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.ustc.edu.cn%2Fhelp%2Fdockerhub.html) | `https://docker.mirrors.ustc.edu.cn` |                   | Docker Hub、[GCR](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fustclug%2Fmirrorrequest%2Fissues%2F91)、[Quay](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fustclug%2Fmirrorrequest%2Fissues%2F135) |
| [阿里云](https://links.jianshu.com/go?to=https%3A%2F%2Fcr.console.aliyun.com) | `https://.mirror.aliyuncs.com`       | 需登录，系统分配  | Docker Hub                                                   |
| [七牛云](https://links.jianshu.com/go?to=https%3A%2F%2Fkirk-enterprise.github.io%2Fhub-docs%2F%23%2Fuser-guide%2Fmirror) | `https://reg-mirror.qiniu.com`       |                   | Docker Hub、GCR、Quay                                        |
| [网易云](https://links.jianshu.com/go?to=https%3A%2F%2Fc.163yun.com%2Fhub) | `https://hub-mirror.c.163.com`       |                   | Docker Hub                                                   |
| [腾讯云](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F457%2F9113) | `https://mirror.ccs.tencentyun.com`  |                   | Docker Hub                                                   |

参考：[Docker Hub 镜像加速器](https://www.jianshu.com/p/5a911f20d93e)



### 开启远程访问

```shell
[root@docker]# vim /usr/lib/systemd/system/docker.service
#修改ExecStart这行
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
#重新加载配置文件
[root@docker]# systemctl daemon-reload
#重启服务
[root@docker]# systemctl restart docker.service
#查看端口是否开启
[root@docker]# netstat -anp|grep 2375
#curl查看是否生效
[root@docker]# curl http://127.0.0.1:2375/info
#Windows主机修改环境变量即可用 IDEA 连接
DOCKER_HOST=tcp://DOCKER_HOST_IP:2375
```

### 安装 Docker Compose

```shell
// 下载可执行文件
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
// 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose
// 创建软连接
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
// 安装命令补全
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
// 验证
docker-compose --version

```

## docker-compose.yml安装常用服务

## PostgreSQL

### 通过 yum 安装

```
$ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ yum install postgresql11
$ yum install postgresql11-server
$ /usr/pgsql-11/bin/postgresql-11-setup initdb
$ systemctl enable postgresql-11
$ systemctl start postgresql-11
```

## MySQL

### 通过 docker 安装

mysql-server 版（也可以使用 docker hub 官方 mysql 镜像，不过镜像更大）

```shell
// 单机MySQL。通过 -v /opt/mysql/data:/var/lib/mysql 选项可指定宿主机数据卷
docker run -d --name=mysql-server -p=3306:3306 -e MYSQL_ROOT_HOST=% -e MYSQL_ROOT_PASSWORD=toor mysql/mysql-server
```



mysql-cluster 版

```shel
docker network create cluster --subnet=192.168.0.0/16
docker run -d --net=cluster --name=management1 --ip=192.168.0.2 mysql/mysql-cluster ndb_mgmd
docker run -d --net=cluster --name=ndb1 --ip=192.168.0.3 mysql/mysql-cluster ndbd
docker run -d --net=cluster --name=ndb2 --ip=192.168.0.4 mysql/mysql-cluster ndbd
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 -p=3306:3306 -e MYSQL_ROOT_HOST=% -e MYSQL_ROOT_PASSWORD=toor mysql/mysql-cluster mysqld
```



### 通过 yum 安装

```
$ yum install -y mysql-community-server.x86_64 (来自 mysql80-community 仓库)
$ CREATE USER 'root'@'%' IDENTIFIED BY 'pass4wOrd!';
$ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
$ FLUSH PRIVILEGES;
```

如果忘记MySQL密码，执行以下操作：

```bash
$ vim /etc/my.cnf
# 在[mysqld]后加上如下语句
# skip-grant-tables
$ systemctl restart mysqld
$ mysql -uroot
mysql> use mysql;
mysql> update user set authentication_string='' where user='root';
mysql> quit;
$ vim /etc/my.cnf
# 删除刚才添加的语句
# skip-grant-tables
$ systemctl restart mysqld
$ mysql -uroot
mysql> ALTER USER 'root'@'%' IDENTIFIED BY 'pass4wOrd!';
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'pass4wOrd!';
```

## MongoDB

### 通过 yum 安装

```shell
$ vim /etc/yum.repos.d/mongodb-org-4.0.repo
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc

$ yum install -y mongodb-org

$ systemctl enable mongod.service
$ systemctl start mongod
$ systemctl stop mongod
# or
$ chkconfig mongod on
$ service mongod start
$ service mongod stop

$ vim /etc/mongod.conf
net:
  port: 27017
  bindIp: 0.0.0.0 或 bindIpAll
```

## Redis

### 通过 tar 包安装

```
$ curl -O http://download.redis.io/releases/redis-5.0.5.tar.gz
$ tar xvzf redis-5.0.5.tar.gz
$ cd redis-5.0.5
$ make distclean install 
$ yum install tcl.x86_64
$ make distclean test

$ vim redis.conf
bind 0.0.0.0

```



## RabbitMQ

### 通过 yum 安装

```bash
# import the new PackageCloud key that will be used starting December 1st, 2018 (GMT)
$ rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey

# import the old PackageCloud key that will be discontinued on December 1st, 2018 (GMT)
$ rpm --import https://packagecloud.io/gpg.key

# install erlang repository
$ curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash

# install rabbitmq repository
$ curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

$ yum install erlang.x86_64
$ yum install rabbitmq-server.noarch

# enable rabbitmq_management
$ rabbitmq-plugins enable rabbitmq_management

# autostart rabbitmq when system start
$ chkconfig rabbitmq-server on

# As an administrator, start and stop the server as usual:
$ /sbin/service rabbitmq-server start
$ /sbin/service rabbitmq-server stop

# download the example config file from website https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example
# and write it to file /etc/rabbitmq/rabbitmq.conf
# uncomment the line "loopback_users.guest = false" so you can use the guest user to login from anywhere on the network
# start the server and then visit http://HOST_IP:15672/
# the credential is below
# username:guest
# password:guest

# display application environment
$ rabbitmqctl environment

# system service logs can be inspected using
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

## Elasticsearch

```shell
$ docker network create elasticsearch
$ docker run -d --name elasticsearch --net elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch
// 或者使用官方镜像，不过一定要加上版本号（镜像更大）
$ docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.4.1
```



## Gogs

### 通过 docker 安装

```
$ docker pull gogs/gogs
$ mkdir -p /var/gogs
$ docker run -d --rm --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs
$ docker start gogs

```

## Jenkins

### 通过 docker 安装

1. ```bash
   docker pull jenkinsci/blueocean
   
   ```

2. ```bash
   docker run \
   -u root \
     --rm \
     -d \
     -p 8080:8080 \
     -p 50000:50000 \
     -v jenkins-data:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     --name jenkins-blueocean \
     jenkinsci/blueocean
   
   ```

3. note the admin password dumped on log: d844e5d059554c85b1012f942109226c

4. open a browser on http://localhost:8080 or http://localhost:8080/blue

## 踩坑记录

### 根分区扩容

1. 先在救援模式下用 `fdisk` 给根分区扩容

2. 然后给物理卷PV扩容

   ```bash
   pvresize /dev/sda2
   
   
   ```

3. 再给逻辑卷LV扩容

   ```bash
   lvextend /dev/centos/root /dev/sda2
   
   
   ```

4. 最后再给XFS文件系统扩容（Centos 7 的文件系统是 XFS）

   ```bash
   xfs_growfs /dev/mapper/centos-root 
   
   
   ```

   

### NetworkManager 导致网卡无法获取 IP

NetworkManager 异常导致网卡设备处以 unmanaged 状态，且无法恢复为 managed 状态，可以尝试以下操作：

```
systemctl stop network-manager
// 清除 NetworkManager 异常状态
rm /var/lib/NetworkManager/NetworkManager.state
systemctl start network-manager
```

