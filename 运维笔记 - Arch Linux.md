[toc]

# 安装

参考：[Installation guide (简体中文)](https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

安装镜像为 archlinux-2020.08.01-x86_64.iso

检查网络是否连通（需要联网下载必要的包）

```bash
ping www.baidu.com
```

更新系统时间

```bash
// 开启网络时间同步
# timedatectl set-ntp true
// 设置时区
# timedatectl set-timezone Asia/Shanghai
```

磁盘分区

```bash
// 验证启动模式
ls /sys/firmware/efi/efivars
// 如果没有任何输出，则是以 BIOS 模式启动（虚拟机默认），否则以 EFI 模式启动

// 查看硬盘
fdisk -l
// 给硬盘分区（我的硬盘是/dev/sda）
fdisk /dev/sda
// 给硬盘分出一个大于512MB的SWAP分区/dev/sda1，以及一个根分区/dev/sda2
// 格式化分区
mkfs.ext4 /dev/sda2
mkswap /dev/sda1
swapon /dev/sda1

// 挂载文件系统
mount /dev/sda2 /mnt
```

安装必要的包

```bash
// 如果在虚拟机安装，则 linux-firmware 可以省略
pacstrap /mnt base linux linux-firmware

//也可以在系统安装重启后通过 pacman 安装
// 文本编辑器
pacstrap /mnt vim
// 命令手册 man 和 info
pacstrap /mnt man-db man-pages textinfo
// 命令补全
pacstrap /mnt bash-completion
// 开发工具包（推荐）
pacstrap /mnt base-devel
```

生成 fstab 文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

chroot 进新系统

```bash
arch-chroot /mnt
// 注意执行 timedatectl、localectl、hostnamectl 等命令
// 不会对 chroot 后的系统起作用
// 如果想要设置，只能通过上面命令的配置文件来设置
// 也可以将上面命令的配置文件复制到挂载盘
// 另外，chroot 后安装的软件包都是安装在Live系统中，而不是 chroot 的系统
```

设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
// 生成 /etc/adjtime
hwclock --systohc
```

本地化（配置 locale.gen 和 locale.conf，如果是 en_US 则不用配置，不推荐设置为中文，会导致 TTY 乱码）

```bash
// 编辑 /etc/locale.gen 文件（由于没有文本编辑器，使用 sed 来处理，当然可以
// 执行 pacman -S vim 来安装）
sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
// 生成区域信息
locale-gen
```

执行 `passwd` 命令设置 root 密码，这里设置为 toor

安装 grub 引导

```bash
// 安装 grub 包
pacman -S grub
// 安装 grub 引导到磁盘
grub-install /dev/sda
// 生成 grub 配置文件
grub-mkconfig -o /boot/grub/grub.cfg
```

退出 chroot

```bash
exit
// 或 Ctrl+D
```

然后执行 `reboot` 命令重启

# 配置

上面的步骤只是安装 Arch 到本地硬盘并重启，重启之后还需要一定的配置才能使用。

参考：[General recommendations (简体中文)](https://wiki.archlinux.org/index.php/General_recommendations_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

## 配置网络

上面安装重启之后是没有网络的，因为网络还没有配置。网络配置放在在重启之前或之后都可以。

Arch 的 base 包中已经包含 systemd-networkd（systemd-networkd.service 和 systemd-resolved.service）网络管理器，无需安装其它网络管理器。所以这里就用 networkd 来配置网络。

```bash
// 查看网络适配器，这里是 ens33
ip addr
// 添加网络配置文件
cat > /etc/systemd/network/20-wired.network <<EOF
```

动态IP地址网络配置文件：

```bash
[Match]
Name=ens33

[Network]
DHCP=yes
EOF
```

或静态IP地址配置文件：

```bash
[Match]
Name=ens33

[Network]
Address=192.168.17.133/24
Gateway=192.168.17.2
DNS=114.114.114.114
```

最后启动网络相关服务：

```bash
// 启动网络管理服务
systemctl enable systemd-networkd.service
systemctl start systemd-networkd.service
// 启动域名解析服务
systemctl enable systemd-resolved.service
systemctl start systemd-resolved.service
```



安装并配置 openssh

```bash
// 安装 openssh
# pacman -S openssh
// 允许192.168.17.*网段的所有电脑使用SSH来访问这台机器。
# vim /etc/hosts.allow
shd:192.168.17.*  

// OpenSSH默认也是不允许root帐户直接登录的。
# vim /etc/ssh/sshd_config 
// 将 "PermitRootLogin yes" 前的 "#" 号去掉就OK。

// 设置开机启动
# systemctl enable sshd
// 启动 ssh 服务
# systemctl start sshd
```



设置主机名（主机名会写入到 /etc/hostname 文件中）（选配）

```bash
hostnamectl set-hostname archos
```

配置域名解析（选配）

```bash
// cat > /etc/hosts <<EOF
127.0.0.1	localhost
::1		localhost
127.0.1.1	archos.localdomain	archos
```

## 配置时间

```bash
// 开启网络时间同步
# timedatectl set-ntp true
// 设置时区
# timedatectl set-timezone Asia/Shanghai
```


## Pacman

```bash
pacman -Sy //更新数据源
pacman -Syy //强制更新数据源
pacman -Syu //升级安装的包。也就是系统升级
```


## *配置源

在安装 Arch 的时候默认会通过以下命令生成镜像排列列表

```bash
// 排列并选择源
reflector --protocol https --latest 70 --sort rate --save /etc/pacman.d/mirrorlist
```

默认的镜像排序列表还会包含其它国家的，所以可以指定为中国

```bash
reflector --verbose --country China --protocol https --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

上面的命令可以在安装系统时 chroot 前执行，或在系统安装重启后安装 reflector 包然后执行。




配置 ArchlinuxCN 源

```bash
# vi /etc/pacman.conf
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

// 更新数据源
# pacman -Sy
// 安装 keyring
# pacman -S archlinuxcn-keyring
```



配置 AUR (Arch User Repository) 源

```bash
pacman -S yay //安装第三方包管理
```

执行以下命令修改 aururl :

```
yay --aururl "https://aur.tuna.tsinghua.edu.cn" --save
```

修改的配置文件位于 `~/.config/yay/config.json` ，可以通过以下命令查看该配置文件：

```
yay -P -g
```

参考：

https://mirrors.tuna.tsinghua.edu.cn/help/archlinuxcn/

https://mirrors.ustc.edu.cn/help/archlinuxcn.html

https://mirrors.tuna.tsinghua.edu.cn/help/AUR/

## *配置防火墙

## 配置 Bash

### 设置命令别名

```bash
cat > ~/.bash_aliases <<EOF
alias ll='ls -alFh --color'
EOF
```

ls 命令的 -F 选项可以让目录在后面加 “/”的形式显示，方便区分

## 配置 Zsh

```bash
// 查看本地有哪几种shell 
cat /etc/shells
// 切换到zsh 
chsh -s /bin/zsh

// 安装 oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 安装其它包

```bash
pacman -S cockpit
systemctl enable cockpit && systemctl start cockpit
```

# 维护

## 用户名和密码

- 用户名：root
- 密码：toor

## 发行日志

https://www.archlinux.org/releng/releases/

## Docker

```bash
// 安装
# pacman -S docker docker-compose
// 查看版本
# docker -v
Docker version 19.03.12-ce, build 48a66213fe

// 设置 Docker Hub 镜像加速器
# tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://1nj0zren.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://registry.docker-cn.com"
    ]
}
EOF
# systemctl restart docker

// 验证
# docker info
# docker run hello-world
```



