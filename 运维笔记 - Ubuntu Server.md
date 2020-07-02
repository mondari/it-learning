[toc]

## 用户名和密码

- 用户名：root
- 密码：toor

## 发行日志

Ubuntu 20.04 LTS：https://wiki.ubuntu.com/FocalFossa/ReleaseNotes

Ubuntu 18.04 LTS：https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes

## Bash 配置

```bash
cat > ~/.bash_aliases <<EOF
alias ll='ls -alFh'
EOF
```



## 常用包

cockpit（Web控制台）

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

## Docker

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

## *ZooKeeper

版本 3.4

## etcd

