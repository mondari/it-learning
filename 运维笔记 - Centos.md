[TOC]



## 用户名和密码

- 用户名：root
- 密码：toor

## Centos vs Ubuntu Server

|            | Centos 7                     | Centos 8                  | Ubuntu Server 2018 LTS | Ubuntu Server 2020 LTS | 备注                            |
| ---------- | ---------------------------- | ------------------------- | ---------------------- | ---------------------- | ------------------------------- |
| Kernel     | 3.10                         | 4.18                      | 4.15                   | 5.4                    |                                 |
| 包管理     | yum                          | dnf 替代 yum              |                        | apt                    |                                 |
| 包仓库     | Base, Extras, Updates        | BaseOS, AppStream, Extras |                        |                        |                                 |
| 网络管理   | NetworkManager, nmcli, nmtui | 同左                      |                        |                        |                                 |
| 网络工具   | ip, ss                       | 同左                      |                        | ifconfig, netstat      |                                 |
| 网络包过滤 | iptables                     | nftables                  |                        |                        | 都是内核的 netfilter 框架的成员 |
| 防火墙     | firewalld, firewall-cmd      | 同左                      |                        | ufw                    |                                 |
| 文件系统   | XFS                          | 同左                      |                        | ext4                   |                                 |
| 显示服务器 | X.org                        | Wayland                   |                        |                        |                                 |
| Web 控制台 | 默认无                       | Cockpit                   |                        |                        |                                 |
| 容器管理   | Docker 1.13                  | Podman                    |                        |                        |                                 |
| 容器编排   | Kubernetes 1.5.2             | -                         |                        |                        |                                 |
| MySQL      | -                            | 8.0                       |                        |                        |                                 |
| MariaDB    | 5.5                          | 10.3                      |                        |                        |                                 |
| PostgreSQL | 9.2                          | 10.6                      |                        | 11                     |                                 |
| Redis      | -                            | 5                         |                        |                        |                                 |
| OpenJDK    | 8, 11                        | 8, 11                     |                        |                        |                                 |
| Maven      | -                            | 3.5                       |                        |                        |                                 |
| Python     | 3.6                          | 3.8                       |                        |                        |                                 |
| Anaconda   | 21                           | 29                        |                        |                        |                                 |
| PHP        | 5.4                          | 7.2                       |                        |                        |                                 |
| Nodejs     | -                            | 10.19                     |                        |                        |                                 |
| Nginx      | -                            | 1.14                      |                        |                        |                                 |
| httpd      | 2.4.6                        | 2.4.37                    |                        |                        |                                 |

## 配置 Bash

### .inputrc

```bash
// 设置 bash 大小写不敏感
$ echo "set completion-ignore-case on">>~/.inputrc
```

## 常用包

yum groups install Development Tools（内含 gcc, git, cmake, perl）
net-tools.x86_64（内含 netstat, ifconfig, route）
yum-cron
bash-completion
mlocate.x86_64
java-1.8.0-openjdk-devel.x86_64（内含 Java 诊断工具）
epel-release（EPEL仓库有Python3）
yum install python-pip（默认安装pyhon-pip2）
pip install --upgrade pip（更新pip）
[open-vm-tools](https://github.com/vmware/open-vm-tools).x86_64（VMware 虚拟机包）

## 配置EPEL镜像

```bash
###备份(如有配置其他epel源)

$ mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
$ mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup

###下载新repo 到/etc/yum.repos.d/
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

### 安装 docker-ce

```bash
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
$ sudo yum install -y docker-ce docker-ce-cli containerd.io

// 启动服务设为开机启动
$ sudo systemctl enable --now docker

// Verify
$ sudo docker run hello-world
```

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
| [阿里云](https://links.jianshu.com/go?to=https%3A%2F%2Fcr.console.aliyun.com) | `https://xxx.mirror.aliyuncs.com`    | 需登录，系统分配  | Docker Hub                                                   |
| [七牛云](https://links.jianshu.com/go?to=https%3A%2F%2Fkirk-enterprise.github.io%2Fhub-docs%2F%23%2Fuser-guide%2Fmirror) | `https://reg-mirror.qiniu.com`       |                   | Docker Hub、GCR、Quay                                        |
| [网易云](https://links.jianshu.com/go?to=https%3A%2F%2Fc.163yun.com%2Fhub) | `https://hub-mirror.c.163.com`       |                   | Docker Hub                                                   |
| [腾讯云](https://links.jianshu.com/go?to=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F457%2F9113) | `https://mirror.ccs.tencentyun.com`  |                   | Docker Hub                                                   |

参考：[Docker Hub 镜像加速器](https://www.jianshu.com/p/5a911f20d93e)



### 开启远程访问

```bash
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

### 设置容器开机自启

```bash
// 在运行docker容器时可以加如下参数来保证每次docker服务重启后容器也自动重启：
$ docker run --restart always <CONTAINER ID>

// 如果已经启动了则可以使用如下命令：
$ docker update --restart always <CONTAINER ID>
```

--restart 具体参数值详细信息：

- no - 容器退出时，不重启容器；

- on-failure - 只有在非0状态退出时才从新启动容器，可设置失败次数，如 on-failure:3 失败重试3次；

- always - 无论退出状态是如何，都重启容器；

### 安装 Docker Compose

```bash
// 下载可执行文件
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
// 下载 docker-compose（这里使用 daocloud 镜像下载）
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

// 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose
// 创建软连接
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
// 安装命令补全
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
// 验证
docker-compose --version
// 使用 docker-compose 启动容器
docker-compose -f docker-compose.yml up -d
```

## Docker Compose 

使用 docker-compose 安装服务需要配置好以下内容：

- 镜像名
- 容器名
- 开机自启动
- 端口映射
- 用户名和密码
- 数据卷映射：数据、日志、配置
- 允许外网访问

### 部署常用服务

```yaml
version: '3'
services:
  portainer:
    image: portainer/portainer
    container_name: portainer
    command: -H unix:///var/run/docker.sock
    restart: always
    ports:
      - 9000:9000
      - 8000:8000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
  mysql:
    image: mysql/mysql-server:8.0
    container_name: mysql
    restart: always
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: toor
      MYSQL_ROOT_HOST: '%'
    volumes:
      - /var/lib/mysql:/var/lib/mysql #数据目录挂载
      - /var/log/mysql:/var/log/mysql #日志目录挂载
      - /etc/mysql/conf.d:/etc/mysql/conf.d #配置目录挂载
  redis:
    image: redis:5.0
    container_name: redis
    command: redis-server --appendonly yes
    restart: always
    ports:
      - 6379:6379
    volumes:
      - /var/lib/redis:/data #数据目录挂载
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    volumes:
      - /var/lib/rabbitmq:/var/lib/rabbitmq #数据目录挂载
      - /var/log/rabbitmq:/var/log/rabbitmq #日志目录挂载
    environment:
      RABBITMQ_DEFAULT_USER: rabbitmq  #默认 guest
      RABBITMQ_DEFAULT_PASS: rabbitmq  #默认 guest
      # RABBITMQ_DEFAULT_VHOST: rabbitmq
    ports:
      - 5672:5672
      - 15672:15672
    restart: always
volumes:
  portainer_data:
```

### 部署 MongoDB

mongo.yaml

```yaml
version: '3'
services:
  mongo:
    image: mongo:4.2
    container_name: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongo
      MONGO_INITDB_ROOT_PASSWORD: mongo
    volumes:
      - /var/lib/mongo:/data/db #数据目录挂载（启动前需要清空已有目录或使用旧密码，否则启动或登陆失败）
    ports:
      - 27017:27017
  mongo-express:
    image: mongo-express:0.49
    container_name: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: mongo
      ME_CONFIG_MONGODB_ADMINPASSWORD: mongo
    depends_on:
      - mongo
  # nginx:
  #   image: nginx:1.17
  #   container_name: nginx
  #   restart: always
  #   ports:
  #   - 8080:80
  #   # - 443:443
  #   volumes:
  #   - /usr/share/nginx/html:/usr/share/nginx/html #静态资源根目录挂载
  #   - /var/log/nginx:/var/log/nginx #日志目录挂载
  #   - /etc/nginx/nginx.conf:/etc/nginx/nginx.conf #配置目录挂载

```



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

## Portainer

Portainer 是 Docker 的 Web 管理界面。

```bash
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer:latest

```

访问 http://localhost:9000，首次访问会提示设置用户名密码，分别设置为 admin 和 portainer.io。

**开启 IPv4 forward**

如果不开启 IPv4 forward，Portainer 容器无法通过外网访问，会提示“WARNING: IPv4 forwarding is disabled. Networking will not work”。

```bash
// 开启
$ vim /etc/sysctl.conf:
net.ipv4.ip_forward = 1
// 重新加载
$ sysctl -p /etc/sysctl.conf
// 检查
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
// 或者通过以下命令检查
$ cat /proc/sys/net/ipv4/ip_forward
1
```



参考：

1. https://www.portainer.io/installation/
2. http://www.senra.me/docker-management-panel-series-portainer/
3. https://docs.kvasirsg.com/centos-7/prefilight-configuration/how-to-enable-ip-forwarding

## Cockpit

Cockpit 是 Linux 的 Web 控制台。Centos 8 在安装时可以选择安装该服务。

1. 安装 cockpit:

   ```bash
   $ yum install cockpit
   ```

2. 启动 cockpit 服务:

   ```bash
   $ systemctl enable --now cockpit.socket
   ```

3. 打开防火墙:

   ```bash
   $ firewall-cmd --permanent --zone=public --add-service=cockpit
   $ firewall-cmd --reload
   ```
   
4. 访问  https://ip-address:9090 

## Rancher

Rancher 开源的企业级 Kubernetes 管理平台。

### 通过 docker 安装

```bash
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --name=rancher rancher/rancher
```

用户名默认为 admin，安装后会提示设置密码，这里设置为 admin


## MySQL

### 通过 docker 安装

mysql-server 版（也可以使用 docker hub 官方 mysql 镜像，不过镜像更大）

```bash
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

#### Centos 8

Centos 8 的 AppStream 仓库下有 mysql-server.x86_64 的包

```bash
$ yum install mysql-server.x86_64
$ systemctl enable --now mysqld.service # 启动并设置开机启动
$ mysql -uroot # 登录
```

#### Centos 7

Centos 7 默认不带 MySQL，但是有 MariaDB 替代。如果要安装 MySQL，需要添加仓库。

1. 从 https://dev.mysql.com/downloads/repo/yum/ 下载 rpm 仓库包

2. 安装 rpm 仓库包：`yum install mysql80-community-release-el7-{version-number}.noarch.rpm` 

3. 检查仓库是否已添加并开启：`yum repolist enabled | grep "mysql.*-community.*"`

4. 安装并登录 MySQL 服务器：
    ```bash
    $ yum install -y mysql-community-server.x86_64
    $ grep 'temporary password' /var/log/mysqld.log # 临时密码
    $ mysql -uroot -p #登录
    ```
    
    

#### 安装后配置

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
$ docker run --name redis -d redis
// or start with persistent storage
$ docker run --name redis -p 6379:6379 -d redis redis-server --appendonly yes
```

### 通过 tar 包安装

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
-v /var/lib/rabbitmq:/var/lib/rabbitmq -v /var/log/rabbitmq:/var/log/rabbitmq \
rabbitmq:3-management

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

## MongoDB

### 通过 docker 安装

```bash
$ sudo docker run --name mongo -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=mongo -e MONGO_INITDB_ROOT_PASSWORD=mongo \
-v /var/lib/mongo:/data/db \
mongo:4.2
```

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
# or
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
// 安装 kibana
$ docker run --name kibana -p 5601:5601 --net elasticsearch -d kibana:7.4.1
// 安装 logstash
$ docker run --rm -it -v ~/pipeline/:/usr/share/logstash/pipeline/ --name logstash -d logstash:7.4.2 #方法一，提供 logstash.conf 文件
$ docker run --rm -it -v ~/settings/:/usr/share/logstash/config/ --name logstash -d logstash:7.4.2 #方法二，提供 logstash.yml 文件
$ docker run --rm -it -v ~/settings/logstash.yml:/usr/share/logstash/config/ --name logstash -d logstash.yml logstash:7.4.2

// ###安装监控工具###
// 这个监控工具需要保证ES在同一Docker Network才能使用 “http://elasticsearch:9200” 连接
$ docker run -p 9800:9800 --net elasticsearch --name elastichd -d containerize/elastichd
// 这两款监控工具只能通过 http://centos-vm://9200 打开（centos-vm是ES服务在宿主机的域名，即虚拟机在宿主机的域名，因为ES服务部署在虚拟机上）
$ docker run -p 9100:9100 --net elasticsearch --name elasticsearch-head -d mobz/elasticsearch-head:5
$ docker run -p 1358:1358 --net elasticsearch --name dejavu -d appbaseio/dejavu
```

参考：

https://hub.docker.com/_/elasticsearch

https://hub.docker.com/_/kibana

https://hub.docker.com/r/mobz/elasticsearch-head

https://github.com/360EntSecGroup-Skylar/ElasticHD 

https://hub.docker.com/r/appbaseio/dejavu



logstash.conf 配置文件

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



## Nacos

### 通过 docker 安装

```bash
$ docker run --name nacos-standalone -e MODE=standalone -p 8848:8848 -d nacos/nacos-server:latest
```

或

```bash
$ git clone https://github.com/nacos-group/nacos-docker.git
$ cd nacos-docker
$ docker-compose -f example/standalone-derby.yaml up -d
```

访问 http://127.0.0.1:8848/nacos/ 

用户名和密码：nacos

参考：https://github.com/nacos-group/nacos-docker/blob/master/README_ZH.md

## Seata

### 通过 docker 安装

```bash
$ docker run --name seata-server -p 8091:8091 seataio/seata-server:latest -d
```

参考：https://hub.docker.com/r/seataio/seata-server

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

## *Kafka

参考：https://hub.docker.com/r/wurstmeister/kafka

## *FastDFS

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
   > p #打印分区表
   > d #删除分区（分区也行）
   > n #新增分区
   > p #打印分区表
   > t #修改分区类型为8e(8e表示LVM，默认是不创建LVM分区)
   > w #写分区表
   > q #退出
   partprobe # 或 重启机器
   ```

   

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

   Centos 6 的文件系统是 Ext4FS，使用

   ```bash
   resize2fs /dev/mapper/centos-root
   ```

   

### 设置网卡自动启动

Centos 安装后本地网卡不会自动启动并获取IP地址，需要使用 `nmtui` 来配置

参考：https://wiki.centos.org/FAQ/CentOS7#Why_does_my_Ethernet_not_work_unless_I_log_in_and_explicitly_enable_it.3F

### NetworkManager 导致网卡无法获取 IP

NetworkManager 异常导致网卡设备处以 unmanaged 状态，且无法恢复为 managed 状态，可以尝试以下操作：

```bash
systemctl stop network-manager
// 清除 NetworkManager 异常状态
rm /var/lib/NetworkManager/NetworkManager.state
systemctl start network-manager
```

### 防火墙开放端口

```bash
// 开放端口
firewall-cmd --add-port=80/tcp --zone=public --permanent
// 重新加载配置
firewall-cmd --reload
// 查询是否开放
firewall-cmd --query-port=80/tcp
// 查询所有开放端口
firewall-cmd --list-port
```

### 为什么 Centos7 会不带 ipconfig 和 netstat 命令？

Centos 7 默认不安装 `net-tools` 工具包，推荐使用 `ip` 和 `ss` 命令替代。

参考：[What have you done with ifconfig/netstat?](https://wiki.centos.org/FAQ/CentOS7#What_have_you_done_with_ifconfig.2Fnetstat.3F)

