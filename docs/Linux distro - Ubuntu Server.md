## 用户名和密码

- 用户名：root
- 密码：toor

## 发行日志

Ubuntu 20.04 LTS：https://wiki.ubuntu.com/FocalFossa/ReleaseNotes

Ubuntu 18.04 LTS：https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes

## Bash 配置

### 设置命令别名

```bash
cat > ~/.bash_aliases <<EOF
alias ll='ls -alFh --color'
EOF
```

ls 命令的 -F 选项可以让目录在后面加 “/”的形式显示，方便区分

## 常用包

cockpit（Web控制台）
build-essential

## 设置静态IP地址

配置 `/etc/netplan/` 下的配置文件

```bash
network:
  version: 2
  renderer: NetworkManager
  
  ethernets:
    ens33:
      dhcp4: no
      dhcp6: no
      addresses: 
        - 192.168.17.132/24
      gateway4: 192.168.17.2
      nameservers:
        addresses: [192.168.17.2]
```

最后 `sudo netplan apply` 使配置生效。

如果没安装 NetworkManager，使用自带的 networkd 也行

## 防火墙配置

Ubuntu 使用 ufw 来简化防火墙的配置，默认 ufw 是不开启的，需要打开：

```bash
sudo ufw enable
```

ufw 的常见使用

```bash
sudo ufw allow 22 # 开启22端口
sudo ufw app list # 列出已添加进防火墙的所有服务
sudo ufw allow OpenSSH #允许 OpenSSH 服务的端口
sudo ufw app info OpenSSH #查看 OpenSSH 服务允许的端口
sudo ufw status # ufw 状态
sudo ufw disable # 关闭 ufw
```

## Docker

Ubuntu 下 docker 命令补全不全，谨慎考虑安装

### 安装 docker-ce

```bash
# step 0: 卸载旧版本
sudo apt-get remove docker docker-engine docker.io containerd runc
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

```

### 安装 docker.io

docker.io 是 Ubuntu 软件源中自带的包

安装

```bash
sudo apt install docker.io
sudo apt install docker-compose
```

查看版本

```bash
$ docker -v
Docker version 19.03.8, build afacb8b7f0
```

参考：https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.3e221b11PpVOLC

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
sudo systemctl restart docker
// 验证
docker info
```

### Docker Swarm

在 manager 节点初始化 swarm：

```bash
ubuntu@ubuntu:~$ sudo docker swarm init
[sudo] password for ubuntu:
Swarm initialized: current node (q671h15fa7obh291ejf2mois4) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4jbnqb7fbux4ptts0ob7vr3zteznxr6qdyrwvxatae9boyzzjy-2zf0natmzb4zp5nowra31e9y8 192.168.17.131:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

在 worker 节点执行上面输出的命令，加入 swarm 节点：

```bash
docker swarm join --token SWMTKN-1-4jbnqb7fbux4ptts0ob7vr3zteznxr6qdyrwvxatae9boyzzjy-2zf0natmzb4zp5nowra31e9y8 192.168.17.131:2377
```

## RabbitMQ

Centos 的仓库不带 RabbitMQ 的包，所以用 Ubuntu 来安装。

```bash
sudo apt install rabbitmq-server
sudo rabbitmq-plugins enable rabbitmq_management #开启RabbitMQ的web管理界面
```

默认 guest 账户不允许远程访问，需要配置

```bash
$ sudo vi /etc/rabbitmq/rabbitmq.conf
loopback_users.guest = false
```

如果要添加用户：

```bash
sudo rabbitmqctl add_user rabbitmq rabbitmq
sudo rabbitmqctl set_user_tags rabbitmq administrator
sudo rabbitmqctl set_permissions -p / rabbitmq ".*" ".*" ".*" 
```

如果要删除用户：

```bash
sudo rabbitmqctl list_users # 先列出所有用户
sudo rabbitmqctl delete_user guest # 删除guest用户
```

添加或删除 Virtual Hosts：

```bash
sudo rabbitmqctl add_vhost <vhost>
sudo rabbitmqctl delete_vhost <vhost>
```



安装延迟消息交换机插件

1. 到这里下载插件：https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases
2. 上传并复制到 `/usr/lib/rabbitmq/lib/rabbitmq_server-3.8.2/plugins` 目录
3. 执行命令：`sudo rabbitmq-plugins enable rabbitmq_delayed_message_exchange`



参考：

1. https://blog.csdn.net/kk185800961/article/details/55214474
2. https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example

## MongoDB

Ubuntu 官方支持的包版本是 3.6，而当前最新版本是 4.2

## Fluentd

### 提高文件描述符数量上限

```bash
# 查询文件描述符数量上限
ulimit -n
# 设置成 65535
cat > /etc/security/limits.d/fluentd.conf <<EOF
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
EOF
# 重启系统
```

### 配置内核参数

```bash
# 配置 Elasticsearch 所需内核参数
echo "vm.max_map_count = 262144" > /etc/sysctl.d/99-elasticsearch.conf
# 重新加载内核参数
sudo sysctl --system
# 查看是否生效
sudo sysctl --values vm.max_map_count

# 配置 Fluentd 所需内核参数
cat > /etc/sysctl.d/99-fluentd.conf <<EOF
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 5000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_max_syn_backlog = 8096
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535
EOF
# 重新加载内核参数
sudo sysctl --system
```

### 开始安装

```bash
# 确保 Elasticsearch 依赖的 Java 版本为 1.8 以上
java -version

# 安装 Elasticsearch
curl -O https://mirrors.huaweicloud.com/kibana/6.1.0/kibana-6.1.0-linux-x86_64.tar.gz
tar -xf elasticsearch-6.1.0.tar.gz
# 开启外网访问
cat >> elasticsearch-6.1.0/config/elasticsearch.yml <<EOF
network.host: 0.0.0.0
EOF
# 启动 Elasticsearch
(cd elasticsearch-6.1.0 && ./bin/elasticsearch)

# 安装 Kibana
curl -O https://mirrors.huaweicloud.com/elasticsearch/6.1.0/elasticsearch-6.1.0.tar.gz
tar -xf kibana-6.1.0-linux-x86_64.tar.gz
# 开启外网访问
cat >> kibana-6.1.0-linux-x86_64/config/kibana.yml <<EOF
server.host: "0.0.0.0"
EOF
# 启动 Kibana
(cd kibana-6.1.0-linux-x86_64 && ./bin/kibana)

# 安装 Fluentd (td-agent)
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-td-agent4.sh | sh
# 安装 Fluentd 的 Elasticsearch 插件
sudo /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch --no-document
```



配置 rsyslogd 以发送日志到 fluentd。在 /etc/rsyslog.d/99-fluentd.conf (没有则新建) 添加以下一行：

```
*.* @127.0.0.1:42185
```

重启 rsyslog 服务：

```
sudo systemctl restart rsyslog
```



配置 Fluentd (td-agent) 以接收来自 rsyslog 的日志，并发送日志到 Elasticsearch。备份并编辑 `/etc/td-agent/td-agent.conf` 为以下内容：

```conf
# source 标记定义数据的来源
# @type 指定使用的插件，port 表示监听的端口，tag 表示给接收的事件打上标签
<source>
  @type syslog
  port 42185
  tag syslog
</source>

# get logs from fluent-logger, fluent-cat or other fluentd instances
# forward 插件接收TCP请求事件，而 http 插件接收http请求事件。
<source>
  @type forward
</source>

# match 标记定义数据的处理方式，需要指定匹配的 tag，这里匹配打上 syslog.*.* 标签的事件。
<match syslog.**>
  @type elasticsearch
  logstash_format true
  <buffer>
    flush_interval 10s # for testing
  </buffer>
</match>
```

重启并测试 Fluentd (td-agent)：

```bash
sudo systemctl restart td-agent.service

# 测试
curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
tail /var/log/td-agent/td-agent.log
```



打开 Kibana 的控制台 http://localhost:5601 查看数据。



参考：

https://docs.fluentd.org/how-to-guides/free-alternative-to-splunk-by-fluentd

https://docs.fluentd.org/installation/install-by-deb

https://docs.fluentd.org/installation/before-install

https://docs.fluentd.org/configuration/config-file

https://www.fluentd.org/datasources

https://www.fluentd.org/guides/recipes/rsyslogd-aggregation

## *ZooKeeper

版本 3.4

## *etcd



# DevStack

DevStack 是一套快速部署 OpenStack 的脚本和工具。



安装前先配置全局 PyPi 镜像地址，配置文件为：/etc/pip.conf

```conf
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```



以下为在 Ubuntu 22.04.1 LTS 系统上安装的命令：

```bash
git clone https://ghproxy.com/https://github.com/openstack/devstack.git --branch stable/zed
cd devstack

# create a local.conf
ADMIN_PASSWORD=openstack
cat > local.conf <<EOF
[[local|localrc]]
ADMIN_PASSWORD=$ADMIN_PASSWORD
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
### download mirror
GIT_BASE=https://ghproxy.com/https://github.com
NOVNC_REPO=https://ghproxy.com/https://github.com/novnc/novnc.git
ETCD_DOWNLOAD_URL=https://ghproxy.com/https://github.com/etcd-io/etcd/releases/download
### swift
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
SWIFT_REPLICAS=1
EOF

# create folder
DEST=${DEST:-/opt/stack}
STACK_USER=$(whoami)
sudo mkdir -p $DEST
sudo chown -R $STACK_USER $DEST
sudo chmod 0755 $DEST

# download images
curl -o /home/ubuntu/devstack/files/cirros-0.5.2-x86_64-disk.img https://ghproxy.com/https://github.com/cirros-dev/cirros/releases/download/0.5.2/cirros-0.5.2-x86_64-disk.img

# fix start_ovn problem（only exists in branch stable/zed）
sed -i '715,727s/OVS_RUNDIR/OVN_RUNDIR/' lib/neutron_plugins/ovn_agent

# start the install
./stack.sh

# uninstall
# ./unstack.sh
```

最后终端输出以下信息表示安装成功

```bash
This is your host IP address: 192.168.204.141
This is your host IPv6 address: ::1
Horizon is now available at http://192.168.204.141/dashboard
Keystone is serving at http://192.168.204.141/identity/
The default users are: admin and demo
The password: openstack

Services are running under systemd unit files.
For more information see:
https://docs.openstack.org/devstack/latest/systemd.html

DevStack Version: zed
Change: 30a7d790b6bf45bbcc6333008621b093c84055d1 Fix setting the tempest virtual env constraints env var 2023-01-26 22:38:37 -0600
OS Version: Ubuntu 22.04 jammy

2023-02-12 12:37:32.260 | stack.sh completed in 1030 seconds.
```





参考：

https://opendev.org/openstack/devstack

https://docs.openstack.org/devstack/latest/configuration.htm
