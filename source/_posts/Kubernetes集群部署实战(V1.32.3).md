---
title: Kubernetes集群部署实战(基于v1.32.3版本)
date: 2025-03-23 10:10:00
categories: "Kubernetes" #文章分类目录 可以省略
tags: #Java 
     - Kubernetes
     - Ubuntu
     - containerd
     - Calico
     - Helm
     - KubernetesDashboard
     - 云原生
     - K8S
     - PVE
description: Kubernetes集群部署实战，基于Kubernetes v1.32.3、Ubuntu 24.04.2 LTS、Containerd v1.7.29版本搭建  #你對本頁的描述 可以省略
---

# 前言

![components-of-kubernetes](https://kubernetes.io/images/docs/components-of-kubernetes.svg)


[Kubernetes集群](https://kubernetes.io/zh-cn/docs/concepts/overview/components/)部署实战，基于Kubernetes v1.32.3、Ubuntu 24.04.2 LTS、Containerd v1.7.27版本搭建。



## 1.1 前置准备

### 1.1.1 前置依赖软件及物料准备

- 服务器系统：PVE 8.1 
- 虚拟机操作系统 Ubuntu 24.04.2 LTS ISO
- Kubernetes v1.32.3 （kubeadm、kubelet、kubectl、kube-apiserver、kube-controller-manager、kube-scheduler、kube-proxy）
- Cri-Containerd v1.7.27    [releases](https://github.com/containerd/containerd/releases)       crictl
- CoreDNS v1.11.3
- Pause v3.10
- Etcd v3.5.16-0
- Calico v3.29.2
- Nginx镜像
- Helm v3.17.2
- Kubernetes Dashboard v7.11.1


### 1.1.2 K8S集群网络IP规划

| 节点名称      | 规划IP       | 当前验证实际使用IP  |          备注           |
| :----------: | :----------: | :---------------: | :---------------------: |
| K8S-Master01 | 10.200.10.11 | 192.168.11.161    | ...                     |
| K8S-Master02 | 10.200.10.12 |                   | ...                     |
| K8S-Master03 | 10.200.10.13 |                   | ...                     |
| k8S-Worker01 | 10.200.10.21 | 192.168.11.165    | ...                     |
| K8S-Worker02 | 10.200.10.22 | 192.168.11.166    | ...                     |
| K8S-Worker03 | 10.200.10.23 |                   | ...                     |
| K8S-Worker04 | 10.200.10.24 |                   | ...                     |
| K8S-Worker05 | 10.200.10.25 |                   | ...                     |


### 1.1.3 主机硬件配置说明 

| 节点名称      | 角色     | CPU  |  内存   | 硬盘  | 备注   |
| :----------: | :--- --: | :--: | :----: | :---: | :-------: |
| K8S-Master01 | master   |  4C  |  8G    | 200GB | 最低4C8G   |
| k8S-Worker01 | worker   |  8C  |  16G   | 200GB | 最低8C16G  |
| K8S-Worker02 | worker   |  8C  |  16G   | 200GB | 最低8C16G  |

                

## 1.2 主机系统基础环境准备 

### 1.2.1 正式安装前，系统需要检查及准备以下操作

- 更新基础软件版本  
- 安装 net-tools、iputils-ping 工具，查看IP配置、网络状况
- 安装必要的编辑器
- 启用SSH连接 OpenSSH  开启SSH服务
- 支持cron定时任务
- 远程时间同步服务支持 
- 初始化系统 systemd 支持程序开机自启动
- 网络下载工具：wget、curl
- 签名校验工具gpg

安装必要的软件及工具

```bash
~# sudo apt-get update
~# sudo apt install net-tools iputils-ping systemd nano openssh-server cron ntpdate ipset ipvsadm debianutils tar which apt-transport-https ca-certificates curl wget gpg -y
~# sudo apt-get upgrade -y
```



### 1.2.2 开启SSH远程登录服务

确认你的系统使用的是哪种初始化系统：

```bash
~# ps -p 1 -o comm=
```

```bash
~$ sudo systemctl enable ssh

~$ sudo systemctl start ssh

~$ sudo system status ssh      # 查看ssh状态 
```


## 1.3 集群主机配置

### 1.3.1 主机名配置 

由于本次使用3台主机完成kubernetes集群部署，其中1台为master节点,名称为k8s-master01;其中2台为worker节点，名称分别为：k8s-worker01及k8s-worker02

master01节点

```bash
~# hostnamectl set-hostname k8s-master01
```

worker01节点

```bash
~# hostnamectl set-hostname k8s-worker01
```


worker02节点

```bash
~# hostnamectl set-hostname k8s-worker02
```


### 1.3.2 主机IP地址配置 

k8s-master01节点IP地址为：192.168.11.161/24

```bash
 root@k8s-master01:~# nano /etc/netplan/50-cloud-init.yaml
 root@k8s-master01:~# cat /etc/netplan/50-cloud-init.yaml
```

50-cloud-init.yaml文件中添加或修改IP地址（注意网卡名称需要与系统中的以太网卡一致）

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.11.161/24
      routes:
        - to: default
          via: 192.168.11.1
      nameservers:
        addresses: [119.29.29.29,114.114.114.114,8.8.8.8]
```

保存k8s-master01的网卡配置

```bash
root@k8s-master01:~# sudo netplan apply
```

重新按新ip连接主机，然后使用以下命名查看ip地址信息


```bash
ip a s
```

k8s-worker01节点IP地址为：192.168.11.165/24

```bash
root@k8s-worker01:~# nano /etc/netplan/50-cloud-init.yaml
root@k8s-worker01:~# cat /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp6s18:
      dhcp4: no
      addresses:
        - 192.168.11.165/24
      routes:
        - to: default
          via: 192.168.11.1
      nameservers:
        addresses: [119.29.29.29,114.114.114.114,8.8.8.8]
```

保存k8s-worker01的网卡配置

```bash
root@k8s-worker01:~# sudo netplan apply
```


k8s-worker02节点IP地址为：192.168.11.166/24

```bash
root@k8s-worker02:~# nano /etc/netplan/50-cloud-init.yaml
root@k8s-worker02:~# cat /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp6s18:
      dhcp4: no
      addresses:
        - 192.168.11.166/24
      routes:
        - to: default
          via: 192.168.11.1
      nameservers:
        addresses: [119.29.29.29,114.114.114.114,8.8.8.8]
```

保存k8s-worker02的网卡配置

```bash
root@k8s-worker02:~# sudo netplan apply
```




### 1.3.3 主机名与IP地址解析 
 

所有集群主机均需要进行配置。


在没有DNS服务器的环境中需要先能解析到各个节点的IP地址



```bash
~# sudo nano /etc/hosts
```

- 在文件后面追加以下内容

```bash
192.168.11.161 k8s-master01
192.168.11.165 k8s-worker01
192.168.11.166 k8s-worker02
```


修改后Ctrl+O,然后Enter键，再Ctrl+X保存文件修改并退出

查看IP解析配置是否生效命令

```bash
~$ ping -c 4 k8s-master01
~$ ping -c 4 k8s-worker01
~$ ping -c 4 k8s-worker02
```

### 1.3.4 时间同步配置 

查看时间

```bash
~# date
 Thu Sep  7 05:39:21 AM UTC 2024
```

更换时区

```bash
~# timedatectl set-timezone Asia/Shanghai
```
 
再次查看时间
```bash
~# date
 Thu Sep  7 01:39:51 PM CST 2024
```

安装ntpdate命令

```bash
~# apt install ntpdate -y
```

使用ntpdate命令同步时间

```bash
~# ntpdate time1.aliyun.com
```


通过计划任务实现时间同步

```bash
~# apt install cron -y
~# crontab -e
 no crontab for root - using an empty one
 Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed
 Choose 1-4 [1]: 2
 ......
0 */1 * * * ntpdate time1.aliyun.com           // 写入配置文件
~# crontab -l   // 查看crontab配置
 ......
0 */1 * * * ntpdate time1.aliyun.com        
```


### 1.3.5 配置内核转发及网桥过滤 

所有主机均需要操作。


Way1：手动执行，手动加载内核转发及网桥过滤模块

```bash
~# modprobe overlay
~# modprobe br_netfilter
```


Way2：创建加载内核模块文件

```bash
~# sudo cat << EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

- 查看已加载的模块

```bash
~# lsmod | egrep "overlay"
overlay               151552  0
```


```bash
~# lsmod | egrep "br_netfilter"
br_netfilter    32768   0       
bridge           307200  1 br_netfilter
```


- 添加网桥过滤及内核转发配置文件

```bash
~# cat << EOF| tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

查看配置文件

```bash
~# nano /etc/modules-load.d/k8s.conf
~# cat /etc/modules-load.d/k8s.conf
```

加载内核参数,使之生效

```bash
~# sysctl --system
```

检查输出的结果中是否包含以下三项：

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```



### 1.3.6 安装ipset及ipvsadm 

所有主机均需要操作。


安装ipset及ipvsadm命令

```bash
 ~# apt install ipset ipvsadm -y
```

配置ipvsadm模块加载

添加系统启动需要加载的模块文件

```bash
~# cat << EOF | tee /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF
```

创建加载模块脚本文件

```bash
~# cat << EOF | tee ipvs.sh
#!/bin/sh
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF
```

上述命令执行完成后，当前目录下会生成一个ipvs.sh脚本文件，接下来使用bash命令执行它

执行ipvs.sh脚本文件，以加载模块

```bash
~# sh ipvs.sh
```

查看ipvs配置是否已生效

```bash
root@k8s-master01:~# lsmod | grep ip_vs
ip_vs_sh               12288  0
ip_vs_wrr              12288  0
ip_vs_rr               12288  0
ip_vs                 221184  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          196608  1 ip_vs
nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
libcrc32c              12288  4 nf_conntrack,btrfs,raid456,ip_vs
```

### 1.3.7 关闭SWAP分区 

修改完成后需要重启操作系统，

Way1：临时关闭（不推荐），若不重启，命令为

```bash
~# swapoff -a
```

Way2：永久关闭swap分区（推荐，操作需要重启操作系统）

```bash
~# nano /etc/fstab    // 进入分区配置文件
 ......

 #/swap.img   none    swap   sw      0       0    // 在此行首添加“#”以注释swap分区的配置
```

修改完成后保存文件并退出


# 二、K8S集群容器运行时 Containerd 准备 

- 安装docker-ce方法：[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

- 本文实际采用containerd作为集群容器镜像的运行平台

## 2.1 Containerd部署文件获取

wget下载1.7.27版本containerd

```bash
~# wget https://github.com/containerd/containerd/releases/download/v1.7.27/cri-containerd-1.7.27-linux-amd64.tar.gz
```

解压安装

```bash
~# sudo apt install debianutils
~# sudo apt install --reinstall libc6
~# sudo apt install --reinstall which


~# sudo tar xf /home/acanx/cri-containerd-1.7.27-linux-amd64.tar.gz  -C /

~# which containerd

/usr/local/bin/containerd
~# which runc

/usr/local/sbin/runc
```

验证containerd、runc是否安装成功

```bash
~# which containerd
~# which runc
~# whereis containerd
```

## 2.2 Containerd配置文件生成并修改 

创建containerd配置文件目录/etc/containerd

```bash
~# mkdir /etc/containerd
~# sudo chmod -R 777 /etc/containerd/       // 然后可以上传配置文件
```

生成配置文件config.toml及默认配置

```bash
~# containerd config default > /etc/containerd/config.toml
```

修改配置文件的第67行

```bash
~# nano /etc/containerd/config.toml
```

```txt
 sandbox_image = "registry.k8s.io/pause:3.10"  // 由3.9修改为3.10
```

- 如果使用阿里云容器镜像仓库，也可以修改为：

```txt
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"  // 由3.9修改为3.10
```

修改配置文件的第139行


```bash
~# vim /etc/containerd/config.toml
 SystemdCgroup = true 由false修改为true
```


## 2.3 Containerd启动及开机自启动 

设置containerd开机自启并立即启动

```bash
~# sudo tee /etc/systemd/system/containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target
 
[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
 
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
 
# 限制日志大小
LimitNOFILE=infinity
 
[Install]
WantedBy=multi-user.target
EOF
```


```bash
~# ls /etc/systemd/system/containerd.service
~# systemctl enable --now containerd
```

验证containerd版本

```bash
~# containerd --version
```



# 三、K8S集群部署 

## 3.1 K8S集群软件apt源准备 

本次使用kubernetes社区软件源仓库或阿里云软件源仓库，建议网络不畅通的选择阿里云软件源仓库

下载用于 Kubernetes 软件包仓库的公共签名密钥,所有仓库都使用相同的签名密钥，因此你可以忽略URL中的版本：

- K8S社区软件源仓库签名文件下载

```bash

~# curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
~# ls /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

- 阿里云软件源仓库签名文件下载

```bash
~# curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 添加 Kubernetes apt 仓库

请注意，此仓库仅包含适用于 Kubernetes 1.32.3 的软件包； 对于其他 Kubernetes 次要版本，则需要更改 URL 中的 Kubernetes 次要版本以匹配你所需的次要版本 。

此操作会覆盖 /etc/apt/sources.list.d/kubernetes.list 中现存的所有配置，如果有的情况下。


#### K8S社区

```bash
~# echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### 阿里云

```bash
~# echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list
```


更新 apt 包索引

```bash
~# sudo apt-get update
```




## 3.2 K8S集群软件安装及kubelet配置 

所有节点均可安装

### 3.2.1 k8s集群软件安装 
 
查看软件列表


```bash
root@k8s-master01:~# apt-cache policy kubeadm

kubeadm:
  Installed: (none)
  Candidate: 1.32.3-1.1
  Version table:
     1.32.3-1.1 500
        500 https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
     1.32.2-1.1 500
        500 https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
     1.32.1-1.1 500
        500 https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
     1.32.0-1.1 500
        500 https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
```


查看软件列表及其依赖关系

```bash
root@k8s-master01:~# apt-cache showpkg kubeadm

Package: kubeadm
Versions: 
1.32.3-1.1 (/var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages)
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 

1.32.2-1.1 (/var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages)
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 

1.32.1-1.1 (/var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages)
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 

1.32.0-1.1 (/var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages)
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 
 Description Language: 
                 File: /var/lib/apt/lists/mirrors.aliyun.com_kubernetes-new_core_stable_v1.32_deb_Packages
                  MD5: 


Reverse Depends: 
  kubeadm:ppc64el,kubeadm
  kubeadm:s390x,kubeadm
  kubeadm:arm64,kubeadm
  kubeadm:ppc64el,kubeadm
  kubeadm:s390x,kubeadm
  kubeadm:arm64,kubeadm
  kubeadm:ppc64el,kubeadm
  kubeadm:s390x,kubeadm
  kubeadm:arm64,kubeadm
  kubeadm:ppc64el,kubeadm
  kubeadm:s390x,kubeadm
  kubeadm:arm64,kubeadm
Dependencies: 
1.32.3-1.1 - cri-tools (2 1.30.0) kubeadm:ppc64el (32 (null)) kubeadm:arm64 (32 (null)) kubeadm:s390x (32 (null)) 
1.32.2-1.1 - cri-tools (2 1.30.0) kubeadm:ppc64el (32 (null)) kubeadm:arm64 (32 (null)) kubeadm:s390x (32 (null)) 
1.32.1-1.1 - cri-tools (2 1.30.0) kubeadm:ppc64el (32 (null)) kubeadm:arm64 (32 (null)) kubeadm:s390x (32 (null)) 
1.32.0-1.1 - cri-tools (2 1.30.0) kubeadm:ppc64el (32 (null)) kubeadm:arm64 (32 (null)) kubeadm:s390x (32 (null)) 
Provides: 
1.32.3-1.1 - 
1.32.2-1.1 - 
1.32.1-1.1 - 
1.32.0-1.1 - 
Reverse Provides: 
```

查看可用软件列表

```bash
root@k8s-master01:~# apt-cache madison kubeadm

   kubeadm | 1.32.3-1.1 | https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
   kubeadm | 1.32.2-1.1 | https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
   kubeadm | 1.32.1-1.1 | https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
   kubeadm | 1.32.0-1.1 | https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.32/deb  Packages
```

Way1：采用默认安装

```bash
~# sudo apt-get install kubelet kubeadm kubectl -y
```


Way2：安装指定版本(v1.32.3-1.1)

```bash
~# sudo apt-get install -y kubelet=1.32.3-1.1 kubeadm=1.32.3-1.1 kubectl=1.32.3-1.1
```

- 如有报错：

```bash
E: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 5005 
(unattended-upgr)
 N: Be aware that removing the lock file is not a solution and may break your 
system.
 E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is 
another process using it?
```
可稍等等

锁定版本，防止后期自动更新

```bash
~# sudo apt-mark hold kubelet kubeadm kubectl
```


解锁版本，则可以执行更新

```bash
~# sudo apt-mark unhold kubelet kubeadm kubectl
```


### 3.2.2 配置kubelet 
 
为了实现容器运行时使用的cgroupdriver与kubelet使用的cgroup的一致性，建议修改如下文件内容。

```bash
~# nano /etc/default/kubelet
 KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
```

- 设置kubelet为开机自启动，由于没有生成配置文件，集群初始化后自动启动

```bash
~# systemctl enable kubelet
```


## 3.3 K8S集群初始化 

### 3.3.1 查看版本 

```bash
root@k8s-master01:~# kubeadm version

kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.3", GitCommit:"32cc146f75aad04beaaa245a7157eb35063a9f99", GitTreeState:"clean", BuildDate:"2025-03-11T19:57:38Z", GoVersion:"go1.23.6", Compiler:"gc", Platform:"linux/amd64"}

```



### 3.3.2 生成部署配置文件 


生成kubeadm部署默认配置文件

```bash
root@k8s-master01:~# kubeadm config print init-defaults > /home/acanx/kubeadm-config.yaml
```

#### 使用kubernetes社区版容器镜像仓库

```bash
root@k8s-master01:~# nano /home/acanx/kubeadm-config.yaml
root@k8s-master01:~# cat kubeadm-config.yaml
```

kubeadm-config.yaml文件中 的配置

```yaml
 apiVersion: kubeadm.k8s.io/v1beta3
 bootstrapTokens:- groups:- system:bootstrappers:kubeadm:default-node-token
 token: abcdef.0123456789abcdef
 ttl: 24h0m0s
 usages:- signing- authentication
 kind: InitConfiguration
 localAPIEndpoint:
 advertiseAddress: 192.168.10.140
 bindPort: 6443
 nodeRegistration:
 criSocket: unix:///var/run/containerd/containerd.sock
 imagePullPolicy: IfNotPresent
 name: k8s-master01
 taints: null--
apiServer:
 timeoutForControlPlane: 4m0s
 apiVersion: kubeadm.k8s.io/v1beta3
 certificatesDir: /etc/kubernetes/pki
 clusterName: kubernetes
 controllerManager: {}
 dns: {}
 etcd:
 local:
 dataDir: /var/lib/etcd
 imageRepository: registry.k8s.io
 kind: ClusterConfiguration
 kubernetesVersion: 1.30.0
 networking:
 dnsDomain: cluster.local
 serviceSubnet: 10.96.0.0/12
 podSubnet: 10.244.0.0/16
scheduler: {}
--
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```



#### 使用阿里云容器镜像仓库

编辑kubeadm配置文件（使用阿里云容器镜像仓库）

```bash
root@k8s-master01:~# nano kubeadm-config.yaml
```

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.11.161   # 设置节点IP地址
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  imagePullSerial: true
  name: k8s-master01      # 修改节点名称
  taints: null
timeouts:
  controlPlaneComponentHealthCheck: 4m0s
  discovery: 5m0s
  etcdAPICall: 2m0s
  kubeletHealthCheck: 4m0s
  kubernetesAPICall: 1m0s
  tlsBootstrap: 5m0s
  upgradeManifests: 5m0s
---
apiServer: {}
apiVersion: kubeadm.k8s.io/v1beta4
caCertificateValidityPeriod: 87600h0m0s
certificateValidityPeriod: 8760h0m0s
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
encryptionAlgorithm: RSA-2048
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers   # 修改镜像地址
kind: ClusterConfiguration
kubernetesVersion: 1.32.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16    # 可按需修改，原则：不与已有的节点网段重合
proxy: {}
scheduler: {}
---
kind: KubeletConfiguration                   # 最后四行是补加的
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

### 3.3.3 查看并下载镜像 


#### 查看及使用Kubernetes社区镜像仓库下载镜像

查看镜像列表（K8S社区镜像仓库）

```bash
root@k8s-master01:~# kubeadm config images list   # 查看kubernetes镜像默认版本

root@k8s-master01:~# kubeadm config images list --kubernetes-version=v1.32.3    # 指定kubernetes镜像版本

registry.k8s.io/kube-apiserver:v1.32.3
registry.k8s.io/kube-controller-manager:v1.32.3
registry.k8s.io/kube-scheduler:v1.32.3
registry.k8s.io/kube-proxy:v1.32.3
registry.k8s.io/coredns/coredns:v1.11.3
registry.k8s.io/pause:3.10
registry.k8s.io/etcd:3.5.16-0
```


```bash
 root@k8s-master01:~# kubeadm config images pull           # 默认从K8S社区镜像仓库下载镜像

 [config/images] Pulled registry.k8s.io/kube-apiserver:v1.32.3
 [config/images] Pulled registry.k8s.io/kube-controller-manager:v1.32.3
 [config/images] Pulled registry.k8s.io/kube-scheduler:v1.32.3
 [config/images] Pulled registry.k8s.io/kube-proxy:v1.32.3
 [config/images] Pulled registry.k8s.io/coredns/coredns:v1.11.3
 [config/images] Pulled registry.k8s.io/pause:3.10
 [config/images] Pulled registry.k8s.io/etcd:3.5.16-0
```


#### 查看及使用阿里云容器镜像仓库下载镜像


查看镜像列表（阿里云容器镜像仓库）

```bash
root@k8s-master01:~# kubeadm config images list --image-repository registry.aliyuncs.com/google_containers

registry.aliyuncs.com/google_containers/kube-apiserver:v1.32.3
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.32.3
registry.aliyuncs.com/google_containers/kube-scheduler:v1.32.3
registry.aliyuncs.com/google_containers/kube-proxy:v1.32.3
registry.aliyuncs.com/google_containers/coredns:v1.11.3
registry.aliyuncs.com/google_containers/pause:3.10
registry.aliyuncs.com/google_containers/etcd:3.5.16-0
```

使用阿里云容器镜像仓库下载

```bash
root@k8s-master01:~# kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers --cri-socket=unix:///run/containerd/containerd.sock

[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.32.3
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.32.3
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.32.3
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.32.3
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.11.3
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.10
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.16-0
```


#### 查看下载的镜像

查看本地下载的containerd镜像

```bash
root@k8s-master01:~# crictl images

WARN[0000] Config "/etc/crictl.yaml" does not exist, trying next: "/usr/bin/crictl.yaml" 
WARN[0000] Image connect using default endpoints: [unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
IMAGE                                                             TAG                 IMAGE ID            SIZE
registry.aliyuncs.com/google_containers/coredns                   v1.11.3             c69fa2e9cbf5f       18.6MB
registry.aliyuncs.com/google_containers/etcd                      3.5.16-0            a9e7e6b294baf       57.7MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.32.0             c2e17b8d0f4a3       28.7MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.32.3             f8bdc4cfa0651       28.7MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.32.0             8cab3d2a8bd0f       26.3MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.32.3             085818208a521       26.3MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.32.0             040f9f8aac8cd       30.9MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.32.3             a1ae78fd2f9d8       30.9MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.32.0             a389e107f4ff1       20.7MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.32.3             b4260bf5078ab       20.7MB
registry.aliyuncs.com/google_containers/pause                     3.10                873ed75102791       320kB
```




### 3.3.4 使用部署配置文件初始化K8S集群 


使用命令初始化Kubeadm（只在Master节点操作）

```bash
root@k8s-master01:~# kubeadm init --config /home/acanx/kubeadm-config.yaml  --upload-certs --v=9  --cri-socket=unix:///run/containerd/containerd.sock
```
输出内容如下：

```bash
I0322 20:07:35.623621    4886 initconfiguration.go:261] loading configuration from "/home/acanx/kubeadm-config.yaml"
[init] Using Kubernetes version: v1.32.0
[preflight] Running pre-flight checks
I0322 20:07:35.633576    4886 checks.go:561] validating Kubernetes and kubeadm version
I0322 20:07:35.633610    4886 checks.go:166] validating if the firewall is enabled and active
I0322 20:07:35.651826    4886 checks.go:201] validating availability of port 6443
I0322 20:07:35.652309    4886 checks.go:201] validating availability of port 10259
I0322 20:07:35.652373    4886 checks.go:201] validating availability of port 10257
I0322 20:07:35.652434    4886 checks.go:278] validating the existence of file /etc/kubernetes/manifests/kube-apiserver.yaml
I0322 20:07:35.652491    4886 checks.go:278] validating the existence of file /etc/kubernetes/manifests/kube-controller-manager.yaml
I0322 20:07:35.652520    4886 checks.go:278] validating the existence of file /etc/kubernetes/manifests/kube-scheduler.yaml
I0322 20:07:35.652560    4886 checks.go:278] validating the existence of file /etc/kubernetes/manifests/etcd.yaml
I0322 20:07:35.652582    4886 checks.go:428] validating if the connectivity type is via proxy or direct
I0322 20:07:35.652631    4886 checks.go:467] validating http connectivity to first IP address in the CIDR
I0322 20:07:35.652672    4886 checks.go:467] validating http connectivity to first IP address in the CIDR
I0322 20:07:35.652699    4886 checks.go:102] validating the container runtime
I0322 20:07:35.653795    4886 checks.go:637] validating whether swap is enabled or not
I0322 20:07:35.653855    4886 checks.go:368] validating the presence of executable ip
I0322 20:07:35.653947    4886 checks.go:368] validating the presence of executable iptables
I0322 20:07:35.654055    4886 checks.go:368] validating the presence of executable mount
I0322 20:07:35.654080    4886 checks.go:368] validating the presence of executable nsenter
I0322 20:07:35.654117    4886 checks.go:368] validating the presence of executable ethtool
I0322 20:07:35.654145    4886 checks.go:368] validating the presence of executable tc
I0322 20:07:35.654182    4886 checks.go:368] validating the presence of executable touch
I0322 20:07:35.654210    4886 checks.go:514] running all checks
I0322 20:07:35.680418    4886 checks.go:399] checking whether the given node name is valid and reachable using net.LookupHost
I0322 20:07:35.680635    4886 checks.go:603] validating kubelet version
I0322 20:07:35.753081    4886 checks.go:128] validating if the "kubelet" service is enabled and active
I0322 20:07:35.774195    4886 checks.go:201] validating availability of port 10250
I0322 20:07:35.774355    4886 checks.go:327] validating the contents of file /proc/sys/net/ipv4/ip_forward
I0322 20:07:35.774477    4886 checks.go:201] validating availability of port 2379
I0322 20:07:35.774568    4886 checks.go:201] validating availability of port 2380
I0322 20:07:35.774631    4886 checks.go:241] validating the existence and emptiness of directory /var/lib/etcd
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
I0322 20:07:35.777659    4886 checks.go:832] using image pull policy: IfNotPresent
I0322 20:07:35.779807    4886 checks.go:868] pulling: registry.aliyuncs.com/google_containers/kube-apiserver:v1.32.0
I0322 20:08:11.358938    4886 checks.go:868] pulling: registry.aliyuncs.com/google_containers/kube-controller-manager:v1.32.0
I0322 20:08:38.140994    4886 checks.go:868] pulling: registry.aliyuncs.com/google_containers/kube-scheduler:v1.32.0
I0322 20:09:00.270475    4886 checks.go:868] pulling: registry.aliyuncs.com/google_containers/kube-proxy:v1.32.0
I0322 20:09:26.947685    4886 checks.go:863] image exists: registry.aliyuncs.com/google_containers/coredns:v1.11.3
I0322 20:09:26.948207    4886 checks.go:863] image exists: registry.aliyuncs.com/google_containers/pause:3.10
I0322 20:09:26.948594    4886 checks.go:863] image exists: registry.aliyuncs.com/google_containers/etcd:3.5.16-0
[certs] Using certificateDir folder "/etc/kubernetes/pki"
I0322 20:09:26.948692    4886 certs.go:112] creating a new certificate authority for ca
[certs] Generating "ca" certificate and key
I0322 20:09:27.066597    4886 certs.go:473] validating certificate period for ca certificate
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.11.161]
[certs] Generating "apiserver-kubelet-client" certificate and key
I0322 20:09:27.313725    4886 certs.go:112] creating a new certificate authority for front-proxy-ca
[certs] Generating "front-proxy-ca" certificate and key
I0322 20:09:27.439421    4886 certs.go:473] validating certificate period for front-proxy-ca certificate
[certs] Generating "front-proxy-client" certificate and key
I0322 20:09:27.532545    4886 certs.go:112] creating a new certificate authority for etcd-ca
[certs] Generating "etcd/ca" certificate and key
I0322 20:09:27.631261    4886 certs.go:473] validating certificate period for etcd/ca certificate
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.11.161 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.11.161 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
I0322 20:09:28.312889    4886 certs.go:78] creating new public/private key files for signing service account users
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
I0322 20:09:28.495281    4886 kubeconfig.go:111] creating kubeconfig file for admin.conf
[kubeconfig] Writing "admin.conf" kubeconfig file
I0322 20:09:28.848054    4886 kubeconfig.go:111] creating kubeconfig file for super-admin.conf
[kubeconfig] Writing "super-admin.conf" kubeconfig file
I0322 20:09:29.129094    4886 kubeconfig.go:111] creating kubeconfig file for kubelet.conf
[kubeconfig] Writing "kubelet.conf" kubeconfig file
I0322 20:09:29.386206    4886 kubeconfig.go:111] creating kubeconfig file for controller-manager.conf
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
I0322 20:09:29.483874    4886 kubeconfig.go:111] creating kubeconfig file for scheduler.conf
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
I0322 20:09:29.607843    4886 local.go:66] [etcd] wrote Static Pod manifest for a local etcd member to "/etc/kubernetes/manifests/etcd.yaml"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
I0322 20:09:29.607888    4886 manifests.go:104] [control-plane] getting StaticPodSpecs
I0322 20:09:29.608085    4886 certs.go:473] validating certificate period for CA certificate
I0322 20:09:29.608178    4886 manifests.go:130] [control-plane] adding volume "ca-certs" for component "kube-apiserver"
I0322 20:09:29.608192    4886 manifests.go:130] [control-plane] adding volume "etc-ca-certificates" for component "kube-apiserver"
I0322 20:09:29.608198    4886 manifests.go:130] [control-plane] adding volume "k8s-certs" for component "kube-apiserver"
I0322 20:09:29.608206    4886 manifests.go:130] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-apiserver"
I0322 20:09:29.608214    4886 manifests.go:130] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-apiserver"
I0322 20:09:29.609567    4886 manifests.go:159] [control-plane] wrote static Pod manifest for component "kube-apiserver" to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
I0322 20:09:29.609599    4886 manifests.go:104] [control-plane] getting StaticPodSpecs
I0322 20:09:29.609807    4886 manifests.go:130] [control-plane] adding volume "ca-certs" for component "kube-controller-manager"
I0322 20:09:29.609819    4886 manifests.go:130] [control-plane] adding volume "etc-ca-certificates" for component "kube-controller-manager"
I0322 20:09:29.609824    4886 manifests.go:130] [control-plane] adding volume "flexvolume-dir" for component "kube-controller-manager"
I0322 20:09:29.609835    4886 manifests.go:130] [control-plane] adding volume "k8s-certs" for component "kube-controller-manager"
I0322 20:09:29.609841    4886 manifests.go:130] [control-plane] adding volume "kubeconfig" for component "kube-controller-manager"
I0322 20:09:29.609846    4886 manifests.go:130] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-controller-manager"
I0322 20:09:29.609854    4886 manifests.go:130] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-controller-manager"
I0322 20:09:29.610610    4886 manifests.go:159] [control-plane] wrote static Pod manifest for component "kube-controller-manager" to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[control-plane] Creating static Pod manifest for "kube-scheduler"
I0322 20:09:29.610640    4886 manifests.go:104] [control-plane] getting StaticPodSpecs
I0322 20:09:29.610848    4886 manifests.go:130] [control-plane] adding volume "kubeconfig" for component "kube-scheduler"
I0322 20:09:29.611322    4886 manifests.go:159] [control-plane] wrote static Pod manifest for component "kube-scheduler" to "/etc/kubernetes/manifests/kube-scheduler.yaml"
I0322 20:09:29.611339    4886 kubelet.go:70] Stopping the kubelet
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
I0322 20:09:29.948568    4886 loader.go:402] Config loaded from file:  /etc/kubernetes/admin.conf
I0322 20:09:29.949088    4886 envvar.go:172] "Feature gate default state" feature="ClientsAllowCBOR" enabled=false
I0322 20:09:29.949115    4886 envvar.go:172] "Feature gate default state" feature="ClientsPreferCBOR" enabled=false
I0322 20:09:29.949124    4886 envvar.go:172] "Feature gate default state" feature="InformerResourceVersion" enabled=false
I0322 20:09:29.949144    4886 envvar.go:172] "Feature gate default state" feature="WatchListClient" enabled=false
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 502.456949ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
I0322 20:09:30.452200    4886 wait.go:220] "Request Body" body=""
I0322 20:09:30.452302    4886 round_trippers.go:473] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubeadm/v1.32.3 (linux/amd64) kubernetes/32cc146" 'https://192.168.11.161:6443/healthz?timeout=10s'
I0322 20:09:30.452515    4886 round_trippers.go:515] HTTP Trace: Dial to tcp:192.168.11.161:6443 failed: dial tcp 192.168.11.161:6443: connect: connection refused
I0322 20:09:30.452547    4886 round_trippers.go:560] GET https://192.168.11.161:6443/healthz?timeout=10s  in 0 milliseconds
I0322 20:09:30.452664    4886 round_trippers.go:577] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 0 ms Duration 0 ms
I0322 20:09:30.452746    4886 round_trippers.go:584] Response Headers:
I0322 20:09:30.952488    4886 wait.go:220] "Request Body" body=""
I0322 20:09:30.952695    4886 round_trippers.go:473] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubeadm/v1.32.3 (linux/amd64) kubernetes/32cc146" 'https://192.168.11.161:6443/healthz?timeout=10s'
I0322 20:09:30.953055    4886 round_trippers.go:515] HTTP Trace: Dial to tcp:192.168.11.161:6443 failed: dial tcp 192.168.11.161:6443: connect: connection refused
I0322 20:09:30.953141    4886 round_trippers.go:560] GET https://192.168.11.161:6443/healthz?timeout=10s  in 0 milliseconds
I0322 20:09:30.953178    4886 round_trippers.go:577] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 0 ms Duration 0 ms
I0322 20:09:30.953196    4886 round_trippers.go:584] Response Headers:
I0322 20:09:31.453072    4886 wait.go:220] "Request Body" body=""
I0322 20:09:31.453425    4886 round_trippers.go:473] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubeadm/v1.32.3 (linux/amd64) kubernetes/32cc146" 'https://192.168.11.161:6443/healthz?timeout=10s'
I0322 20:09:31.453892    4886 round_trippers.go:515] HTTP Trace: Dial to tcp:192.168.11.161:6443 failed: dial tcp 192.168.11.161:6443: connect: connection refused
I0322 20:09:31.453935    4886 round_trippers.go:560] GET https://192.168.11.161:6443/healthz?timeout=10s  in 0 milliseconds
I0322 20:09:31.453947    4886 round_trippers.go:577] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 0 ms Duration 0 ms
I0322 20:09:31.453955    4886 round_trippers.go:584] Response Headers:
I0322 20:09:31.952836    4886 wait.go:220] "Request Body" body=""
I0322 20:09:31.953094    4886 round_trippers.go:473] curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: kubeadm/v1.32.3 (linux/amd64) kubernetes/32cc146" 'https://192.168.11.161:6443/healthz?timeout=10s'
I0322 20:09:31.953553    4886 round_trippers.go:517] HTTP Trace: Dial to tcp:192.168.11.161:6443 succeed
I0322 20:09:33.315605    4886 round_trippers.go:560] GET https://192.168.11.161:6443/healthz?timeout=10s 403 Forbidden in 1362 milliseconds
I0322 20:09:33.315644    4886 round_trippers.go:577] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 1336 ms ServerProcessing 25 ms Duration 1362 ms
I0322 20:09:33.315657    4886 round_trippers.go:584] Response Headers:
I0322 20:09:33.315670    4886 round_trippers.go:587]     X-Content-Type-Options: nosniff
I0322 20:09:33.315701    4886 round_trippers.go:587]     X-Kubernetes-Pf-Flowschema-Uid: 
I0322 20:09:33.315710    4886 round_trippers.go:587]     X-Kubernetes-Pf-Prioritylevel-Uid: 
I0322 20:09:33.315717    4886 round_trippers.go:587]     Content-Length: 192
I0322 20:09:33.315723    4886 round_trippers.go:587]     Date: Sat, 22 Mar 2025 12:09:33 GMT
I0322 20:09:33.315731    4886 round_trippers.go:587]     Audit-Id: 989b38d0-29d4-4450-b564-12e6142c52b7
I0322 20:09:33.315737    4886 round_trippers.go:587]     Cache-Control: no-cache, private
I0322 20:09:33.315741    4886 round_trippers.go:587]     Content-Type: application/json
I0322 20:09:33.316997    4886 wait.go:220] "Response Body" body=<
	{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User \"kubernetes-admin\" cannot get path \"/healthz\"","reason":"Forbidden","details":{},"code":403}
 >

.......   中间冗长的日志省略

I0322 20:09:37.294961    4886 round_trippers.go:473] curl -v -XPOST  -H "User-Agent: kubeadm/v1.32.3 (linux/amd64) kubernetes/32cc146" -H "Accept: application/vnd.kubernetes.protobuf,application/json" -H "Content-Type: application/vnd.kubernetes.protobuf" 'https://192.168.11.161:6443/apis/rbac.authorization.k8s.io/v1/namespaces/kube-system/rolebindings?timeout=10s'
I0322 20:09:37.298583    4886 round_trippers.go:560] POST https://192.168.11.161:6443/apis/rbac.authorization.k8s.io/v1/namespaces/kube-system/rolebindings?timeout=10s 201 Created in 3 milliseconds
I0322 20:09:37.298609    4886 round_trippers.go:577] HTTP Statistics: GetConnection 0 ms ServerProcessing 3 ms Duration 3 ms
I0322 20:09:37.298634    4886 round_trippers.go:584] Response Headers:
I0322 20:09:37.298649    4886 round_trippers.go:587]     X-Kubernetes-Pf-Flowschema-Uid: 9a8a526f-0630-4d03-b851-7ace48e8dc05
I0322 20:09:37.298661    4886 round_trippers.go:587]     X-Kubernetes-Pf-Prioritylevel-Uid: d044472a-2a80-46df-8bc8-7d94366f6e34
I0322 20:09:37.298667    4886 round_trippers.go:587]     Content-Length: 385
I0322 20:09:37.298675    4886 round_trippers.go:587]     Date: Sat, 22 Mar 2025 12:09:37 GMT
I0322 20:09:37.298680    4886 round_trippers.go:587]     Audit-Id: ece06481-523b-4fe4-9d2c-2d369eceecc1
I0322 20:09:37.298690    4886 round_trippers.go:587]     Cache-Control: no-cache, private
I0322 20:09:37.298695    4886 round_trippers.go:587]     Content-Type: application/vnd.kubernetes.protobuf
I0322 20:09:37.298744    4886 type.go:252] "Response Body" body=<
	00000000  6b 38 73 00 0a 2b 0a 1c  72 62 61 63 2e 61 75 74  |k8s..+..rbac.aut|
	00000010  68 6f 72 69 7a 61 74 69  6f 6e 2e 6b 38 73 2e 69  |horization.k8s.i|
	00000020  6f 2f 76 31 12 0b 52 6f  6c 65 42 69 6e 64 69 6e  |o/v1..RoleBindin|
	00000030  67 12 c9 02 0a c0 01 0a  0a 6b 75 62 65 2d 70 72  |g........kube-pr|
	00000040  6f 78 79 12 00 1a 0b 6b  75 62 65 2d 73 79 73 74  |oxy....kube-syst|
	00000050  65 6d 22 00 2a 24 63 38  62 62 36 66 63 64 2d 38  |em".*$c8bb6fcd-8|
	00000060  66 35 38 2d 34 35 35 34  2d 39 66 36 30 2d 38 39  |f58-4554-9f60-89|
	00000070  39 32 61 39 65 36 64 36  64 39 32 03 33 30 33 38  |92a9e6d6d92.3038|
	00000080  00 42 08 08 81 d1 fa be  06 10 00 8a 01 69 0a 07  |.B...........i..|
	00000090  6b 75 62 65 61 64 6d 12  06 55 70 64 61 74 65 1a  |kubeadm..Update.|
	000000a0  1c 72 62 61 63 2e 61 75  74 68 6f 72 69 7a 61 74  |.rbac.authorizat|
	000000b0  69 6f 6e 2e 6b 38 73 2e  69 6f 2f 76 31 22 08 08  |ion.k8s.io/v1"..|
	000000c0  81 d1 fa be 06 10 00 32  08 46 69 65 6c 64 73 56  |.......2.FieldsV|
	000000d0  31 3a 22 0a 20 7b 22 66  3a 72 6f 6c 65 52 65 66  |1:". {"f:roleRef|
	000000e0  22 3a 7b 7d 2c 22 66 3a  73 75 62 6a 65 63 74 73  |":{},"f:subjects|
	000000f0  22 3a 7b 7d 7d 42 00 12  55 0a 05 47 72 6f 75 70  |":{}}B..U..Group|
	00000100  12 19 72 62 61 63 2e 61  75 74 68 6f 72 69 7a 61  |..rbac.authoriza|
	00000110  74 69 6f 6e 2e 6b 38 73  2e 69 6f 1a 2f 73 79 73  |tion.k8s.io./sys|
	00000120  74 65 6d 3a 62 6f 6f 74  73 74 72 61 70 70 65 72  |tem:bootstrapper|
	00000130  73 3a 6b 75 62 65 61 64  6d 3a 64 65 66 61 75 6c  |s:kubeadm:defaul|
	00000140  74 2d 6e 6f 64 65 2d 74  6f 6b 65 6e 22 00 1a 2d  |t-node-token"..-|
	00000150  0a 19 72 62 61 63 2e 61  75 74 68 6f 72 69 7a 61  |..rbac.authoriza|
	00000160  74 69 6f 6e 2e 6b 38 73  2e 69 6f 12 04 52 6f 6c  |tion.k8s.io..Rol|
	00000170  65 1a 0a 6b 75 62 65 2d  70 72 6f 78 79 1a 00 22  |e..kube-proxy.."|
	00000180  00                                                |.|
 >
[addons] Applied essential addon: kube-proxy
I0322 20:09:37.299409    4886 loader.go:402] Config loaded from file:  /etc/kubernetes/admin.conf
I0322 20:09:37.300120    4886 loader.go:402] Config loaded from file:  /etc/kubernetes/admin.conf

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.11.161:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:c7a1d928a82c30bec6dec880092339e64309b24cc01782d8a9d9c6c0ecfde58f 

```


## 3.4 准备kubectl配置文件 

以下操作仅在Master节点上进行。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 3.5 工作节点加入集群 

### k8s-worker01节点加入集群 

以下操作仅在Worker节点操作

```bash
root@k8s-worker01:~# kubeadm join 192.168.11.161:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:c7a1d928a82c30bec6dec880092339e64309b24cc01782d8a9d9c6c0ecfde58f 
```

### k8s-worker02节点加入集群 

以下操作仅在Worker节点操作

```bash
root@k8s-worker02:~# kubeadm join 192.168.11.161:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:c7a1d928a82c30bec6dec880092339e64309b24cc01782d8a9d9c6c0ecfde58f
```


## 3.6 验证K8S集群节点是否可用 

在Master节点检查集群节点是否可用 

```bash
root@k8s-master01:~# kubectl get nodes

NAME           STATUS     ROLES           AGE     VERSION
k8s-master01   NotReady   control-plane   103m    v1.32.3
k8s-worker01   NotReady   <none>          4m12s   v1.32.3
k8s-worker02   NotReady   <none>          4m12s   v1.32.3
```

在Master节点检查集群Pod状态（按命名空间筛选） 

```bash
root@k8s-master01:~# kubectl get pods -n kube-system

NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6766b7b6bb-6dtl7               0/1     Pending   0          104m
coredns-6766b7b6bb-lrpw7               0/1     Pending   0          104m
etcd-k8s-master01                      1/1     Running   0          104m      // 静态pod
kube-apiserver-k8s-master01            1/1     Running   0          104m      // 静态pod
kube-controller-manager-k8s-master01   1/1     Running   0          104m      // 静态pod    
kube-proxy-6g8dn                       1/1     Running   0          104m      
kube-proxy-kmzv6                       1/1     Running   0          5m1s
kube-proxy-pj2t5                       1/1     Running   0          5m1s
kube-scheduler-k8s-master01            1/1     Running   0          104m      // 静态pod 
```

查看命名空间

```bash
root@k8s-master01:~# kubectl get cs

Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE   ERROR
etcd-0               Healthy   ok        
scheduler            Healthy   ok        
controller-manager   Healthy   ok  
```


# 四、K8S集群网络插件calico部署 

[calico网络插件](https://projectcalico.docs.tigera.io/about/about-calico)



## 4.1 第一步：安装tigera-operator

```bash
root@k8s-master01:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
```

### 4.1.1 输出内容：

``` bash
root@k8s-master01:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml

namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/tiers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/adminnetworkpolicies.policy.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
```
 


```bash
root@k8s-master01:~# kubectl get ns

NAME              STATUS   AGE
default           Active   116m
kube-node-lease   Active   116m
kube-public       Active   116m
kube-system       Active   116m
tigera-operator   Active   106s      // 新增的pod
```


```bash
root@k8s-master01:~# kubectl get pods -n tigera-operator

NAME                              READY   STATUS             RESTARTS   AGE
tigera-operator-ccfc44587-vswlt   0/1     ImagePullBackOff   0          3m44s
```


## 4.2 第二步：安装calico-system


### 4.2.1 下载custom-resources.yaml

```bash
~# wget https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml
```


### 4.2.2 修改custom-resources.yaml中的网段cidr配置

```bash 
~# nano custom-resources.yaml
```

```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.Installation
 apiVersion: operator.tigera.io/v1
 kind: Installation
 metadata:
  name: default
 spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16                # 修改此行内容为初始化时定义的pod network cidr
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
      
---
# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
 spec: {}
```


- 再次检查修改后的配置文件

```bash
~# cat custom-resources.yaml
```


### 4.2.3 应用部署配置文件，进行部署

```bash
root@k8s-master01:~# kubectl create -f custom-resources.yaml

installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
```


### 4.2.4 检查calico安装结果

```bash
root@k8s-master01:~# watch kubectl get pods -n calico-system
```

经过一段时间的下载安装，全部署完成后大致如下输出所示：

```bash
root@k8s-master01:~# kubectl get pods -n calico-system

NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-cf49d8944-6zv2z   1/1     Running   0          53m
calico-node-2nfvf                         1/1     Running   0          53m
calico-node-nxhvj                         1/1     Running   0          53m
calico-node-v9lp9                         1/1     Running   0          53m
calico-typha-7b6f98bc78-49bf5             1/1     Running   0          53m
calico-typha-7b6f98bc78-7vhd6             1/1     Running   0          53m
csi-node-driver-6nbvr                     2/2     Running   0          53m
csi-node-driver-9cqq5                     2/2     Running   0          53m
csi-node-driver-9pm4t                     2/2     Running   0          53m   
```


查看节点状态变化：

```bash
root@k8s-master01:~# kubectl get nodes

NAME           STATUS   ROLES           AGE     VERSION
k8s-master01   Ready    control-plane   3h40m   v1.32.3
k8s-worker01   Ready    <none>          121m    v1.32.3
k8s-worker02   Ready    <none>          121m    v1.32.3
```

查看pod状态

```bash
root@k8s-master01:~# kubectl get pods -n kube-system

NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6766b7b6bb-6dtl7               1/1     Running   0          3h42m
coredns-6766b7b6bb-lrpw7               1/1     Running   0          3h42m
etcd-k8s-master01                      1/1     Running   0          3h42m
kube-apiserver-k8s-master01            1/1     Running   0          3h42m
kube-controller-manager-k8s-master01   1/1     Running   0          3h42m
kube-proxy-6g8dn                       1/1     Running   0          3h42m
kube-proxy-kmzv6                       1/1     Running   0          122m
kube-proxy-pj2t5                       1/1     Running   0          122m
kube-scheduler-k8s-master01            1/1     Running   0          3h42m
```

查看pod的IP及网段

```bash
root@k8s-master01:~# kubectl get pods -n kube-system -o wide

NAME                                   READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
coredns-6766b7b6bb-6dtl7               1/1     Running   0          3h43m   10.244.69.198    k8s-worker02   <none>           <none>
coredns-6766b7b6bb-lrpw7               1/1     Running   0          3h43m   10.244.69.197    k8s-worker02   <none>           <none>
etcd-k8s-master01                      1/1     Running   0          3h44m   192.168.11.161   k8s-master01   <none>           <none>
kube-apiserver-k8s-master01            1/1     Running   0          3h44m   192.168.11.161   k8s-master01   <none>           <none>
kube-controller-manager-k8s-master01   1/1     Running   0          3h44m   192.168.11.161   k8s-master01   <none>           <none>
kube-proxy-6g8dn                       1/1     Running   0          3h43m   192.168.11.161   k8s-master01   <none>           <none>
kube-proxy-kmzv6                       1/1     Running   0          124m    192.168.11.165   k8s-worker01   <none>           <none>
kube-proxy-pj2t5                       1/1     Running   0          124m    192.168.11.166   k8s-worker02   <none>           <none>
kube-scheduler-k8s-master01            1/1     Running   0          3h44m   192.168.11.161   k8s-master01   <none>           <none>
```



# 五、部署Nginx应用验证K8S集群可用性 

本章以Nginx为例，先添加nginx应用配置文件nginx.yaml，再添加相关应用配置


## 5.1 配置NginxWeb部署配置文件nginx-website.yaml

```bash
root@k8s-master01:~# nano /home/acanx/nginx-website.yaml
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxweb
spec:
  selector:
    matchLabels:
      app: nginxwebsite
  replicas: 3
  template:
    metadata:
      labels:
        app: nginxwebsite
    spec:
      containers:
      - name: nginxwebsite
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginxweb-service
spec:
  externalTrafficPolicy: Cluster
  selector:
    app: nginxwebsite
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```


## 5.2 执行命令部署nginx服务

```bash
root@k8s-master01:~# kubectl apply -f /home/acanx/nginx-website.yaml
```

```
deployment.apps/nginxweb created
service/nginxweb-service created
```


## 5.3 检查部署结果

```bash
root@k8s-master01:~# kubectl get deployment

NAME       READY   UP-TO-DATE   AVAILABLE   AGE
nginxweb   0/3     3            0           2m35s
```

```bash
root@k8s-master01:~# kubectl get pods

NAME                       READY   STATUS              RESTARTS   AGE
nginxweb-7c88f6c77-pck4f   0/1     ContainerCreating   0          3m43s
nginxweb-7c88f6c77-rv76n   0/1     ContainerCreating   0          3m43s
nginxweb-7c88f6c77-vb8xj   0/1     ContainerCreating   0          3m43s
```


### 5.3.1 测试K8S集群内访问

```bash
root@k8s-master01:~# kubectl get service

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        4h11m
nginxweb-service   NodePort    10.106.65.218   <none>        80:30080/TCP   10m
root@k8s-master01:~# curl http://10.106.65.218               // 验证在K8S集群内布访问

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 5.3.2 测试集群外访问

打开集群外的局域网内内主机，正常情况下集群内任意一个节点IP加对应端口号都可以访问，实际分别测试访问 http://192.168.11.161:30080/ http://192.168.11.165:30080/  http://192.168.11.166:30080/ 页面，以上三个url链接均可以正常打开。

效果如下图所示：

![Master01](/Node/imgs/03_02/Nginx-K8SMaster01.png)
![Worker01](/Node/imgs/03_02/Nginx-K8SWorker01.png)
![Worker02](/Node/imgs/03_02/Nginx-K8SWorker02.png)

上述页面可以正常显示则说明nginx演示应用部署成功！



# 六、 部署Kubernetes Dashboard应用

- [部署和访问 Kubernetes 仪表板（Dashboard）](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/)

## 6.1 安装Helm 

### 6.1.1 Helm介绍及下载

Helm 是查找、分享和使用软件构建 Kubernetes 的最优方式。

- 官网 https://helm.sh/
- 项目地址 https://github.com/helm/helm
- Helm安装文档 https://helm.sh/zh/docs/intro/install/

### 6.1.2 Helm安装


```bash
root@k8s-master01:~# wget https://get.helm.sh/helm-v3.17.2-linux-amd64.tar.gz   # 下载文件

root@k8s-master01:~# tar -zxvf /home/acanx/helm-v3.0.0-linux-amd64.tar.gz         # 解压文件

root@k8s-master01:~# sudo mv linux-amd64/helm /usr/local/bin/helm                    # 移动到指定目录
root@k8s-master01:~# sudo mv /home/acanx/helm /usr/local/bin/helm                    # 移动到指定目录

root@k8s-master01:~# ls -l /usr/local/bin/helm

root@k8s-master01:~# chmod 755 /usr/local/bin/helm

root@k8s-master01:~# helm 
```

```bash
root@k8s-master01:~# helm

The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                                                                                |
|------------------------------------|------------------------------------------------------------------------------------------------------------|
| $HELM_CACHE_HOME                   | set an alternative location for storing cached files.                                                      |
| $HELM_CONFIG_HOME                  | set an alternative location for storing Helm configuration.                                                |
| $HELM_DATA_HOME                    | set an alternative location for storing Helm data.                                                         |
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode                                                      |
| $HELM_DRIVER                       | set the backend storage driver. Values are: configmap, secret, memory, sql.                                |
| $HELM_DRIVER_SQL_CONNECTION_STRING | set the connection string the SQL storage driver should use.                                               |
| $HELM_MAX_HISTORY                  | set the maximum number of helm release history.                                                            |
| $HELM_NAMESPACE                    | set the namespace used for the helm operations.                                                            |
| $HELM_NO_PLUGINS                   | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                                                 |
| $HELM_PLUGINS                      | set the path to the plugins directory                                                                      |
| $HELM_REGISTRY_CONFIG              | set the path to the registry config file.                                                                  |
| $HELM_REPOSITORY_CACHE             | set the path to the repository cache directory                                                             |
| $HELM_REPOSITORY_CONFIG            | set the path to the repositories file.                                                                     |
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config")                                |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                                                  |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                                             |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.                                      |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                                         |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                                                    |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                                          |
| $HELM_KUBEINSECURE_SKIP_TLS_VERIFY | indicate if the Kubernetes API server's certificate validation should be skipped (insecure)                |
| $HELM_KUBETLS_SERVER_NAME          | set the server name used to validate the Kubernetes API server certificate                                 |
| $HELM_BURST_LIMIT                  | set the default burst limit in the case the server contains many CRDs (default 100, -1 to disable)         |
| $HELM_QPS                          | set the Queries Per Second in cases where a high number of calls exceed the option for higher burst values |

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --burst-limit int                 client-side default throttling limit (default 100)
      --debug                           enable verbose output
  -h, --help                            help for helm
      --kube-apiserver string           the address and the port for the Kubernetes API server
      --kube-as-group stringArray       group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string             username to impersonate for the operation
      --kube-ca-file string             the certificate authority file for the Kubernetes API server connection
      --kube-context string             name of the kubeconfig context to use
      --kube-insecure-skip-tls-verify   if true, the Kubernetes API server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kube-tls-server-name string     server name to use for Kubernetes API server certificate validation. If it is not provided, the hostname used to contact the server is used
      --kube-token string               bearer token used for authentication
      --kubeconfig string               path to the kubeconfig file
  -n, --namespace string                namespace scope for this request
      --qps float32                     queries per second used when communicating with the Kubernetes API, not including bursting
      --registry-config string          path to the registry config file (default "/root/.config/helm/registry/config.json")
      --repository-cache string         path to the directory containing cached repository indexes (default "/root/.cache/helm/repository")
      --repository-config string        path to the file containing repository names and URLs (default "/root/.config/helm/repositories.yaml")

Use "helm [command] --help" for more information about a command.
root@k8s-master01:~# 
```

若输出上述内容，则说明安装成功



## 6.2 安装kubernetes-dashboard


### 6.2.1 添加 kubernetes-dashboard 仓库

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

- /root/.cache/helm/repository/kubernetes-dashboard-charts.txt
- /root/.cache/helm/repository/kubernetes-dashboard-index.yaml


```bash
root@k8s-master01:~/.cache/helm/repository# helm repo list

NAME                	URL                                    
kubernetes-dashboard	https://kubernetes.github.io/dashboard/
root@k8s-master01:~/.cache/helm/repository# 
```



### 6.2.2 使用 kubernetes-dashboard Chart 部署名为“kubernetes-dashboard” 的 Helm Release

```bash
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

```bash
root@k8s-master01:~# helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard

Release "kubernetes-dashboard" does not exist. Installing it now.
NAME: kubernetes-dashboard
LAST DEPLOYED: Sun Mar 23 17:44:31 2025
NAMESPACE: kubernetes-dashboard
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*************************************************************************************************
*** PLEASE BE PATIENT: Kubernetes Dashboard may need a few minutes to get up and become ready ***
*************************************************************************************************

Congratulations! You have just installed Kubernetes Dashboard in your cluster.

To access Dashboard run:
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

NOTE: In case port-forward command does not work, make sure that kong service name is correct.
      Check the services in Kubernetes Dashboard namespace using:
        kubectl -n kubernetes-dashboard get svc

Dashboard will be available at:
  https://localhost:8443
root@k8s-master01:~# 



# 注意：以下命令未做验证，后续可以测试确认
# 我的集群使用默认参数安装 kubernetes-dashboard-kong 出现异常 8444 端口占用
# 使用下面的命令进行安装，在安装时关闭kong.tls功能
root@k8s-master01:~# helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --namespace kubernetes-dashboard --set kong.admin.tls.enabled=false   # 原文链接：https://blog.csdn.net/qq_33921750/article/details/136668014
```

#### 查看kubernetes-dashboard相关的pod的安装情况

```bash
root@k8s-master01:~# watch kubectl get pod -A

NAMESPACE              NAME                                                    READY   STATUS              RESTARTS   AGE
calico-apiserver       calico-apiserver-7b7bbf4d75-b77ph                       1/1     Running             0          18h
calico-apiserver       calico-apiserver-7b7bbf4d75-gqq5w                       1/1     Running             0          18h
calico-system          calico-kube-controllers-cf49d8944-6zv2z                 1/1     Running             0          18h
calico-system          calico-node-2nfvf                                       1/1     Running             0          18h
calico-system          calico-node-nxhvj                                       1/1     Running             0          18h
calico-system          calico-node-v9lp9                                       1/1     Running             0          18h
calico-system          calico-typha-7b6f98bc78-49bf5                           1/1     Running             0          18h
calico-system          calico-typha-7b6f98bc78-7vhd6                           1/1     Running             0          18h
calico-system          csi-node-driver-6nbvr                                   2/2     Running             0          18h
calico-system          csi-node-driver-9cqq5                                   2/2     Running             0          18h
calico-system          csi-node-driver-9pm4t                                   2/2     Running             0          18h
default                nginxweb-7c88f6c77-pck4f                                1/1     Running             0          17h
default                nginxweb-7c88f6c77-rv76n                                1/1     Running             0          17h
default                nginxweb-7c88f6c77-vb8xj                                1/1     Running             0          17h
kube-system            coredns-6766b7b6bb-6dtl7                                1/1     Running             0          21h
kube-system            coredns-6766b7b6bb-lrpw7                                1/1     Running             0          21h
kube-system            etcd-k8s-master01                                       1/1     Running             0          21h
kube-system            kube-apiserver-k8s-master01                             1/1     Running             0          21h
kube-system            kube-controller-manager-k8s-master01                    1/1     Running             0          21h
kube-system            kube-proxy-6g8dn                                        1/1     Running             0          21h
kube-system            kube-proxy-kmzv6                                        1/1     Running             0          19h
kube-system            kube-proxy-pj2t5                                        1/1     Running             0          19h
kube-system            kube-scheduler-k8s-master01                             1/1     Running             0          21h
kubernetes-dashboard   kubernetes-dashboard-api-c787c9d97-5rqvp                0/1     ContainerCreating   0          2m35s
kubernetes-dashboard   kubernetes-dashboard-auth-7ff7855689-vmltw              0/1     ContainerCreating   0          2m35s
kubernetes-dashboard   kubernetes-dashboard-kong-79867c9c48-txfql              0/1     Init:0/1            0          2m35s
kubernetes-dashboard   kubernetes-dashboard-metrics-scraper-84655b9bd8-mq8h7   0/1     ContainerCreating   0          2m35s
kubernetes-dashboard   kubernetes-dashboard-web-658946f7f9-crj69               0/1     ContainerCreating   0          2m35s
tigera-operator        tigera-operator-ccfc44587-vswlt                         1/1     Running             0          19h



root@k8s-master01:~/.cache/helm/repository# kubectl get pod -A

NAMESPACE              NAME                                                    READY   STATUS    RESTARTS   AGE
calico-apiserver       calico-apiserver-7b7bbf4d75-b77ph                       1/1     Running   0          19h
calico-apiserver       calico-apiserver-7b7bbf4d75-gqq5w                       1/1     Running   0          19h
calico-system          calico-kube-controllers-cf49d8944-6zv2z                 1/1     Running   0          19h
calico-system          calico-node-2nfvf                                       1/1     Running   0          19h
calico-system          calico-node-nxhvj                                       1/1     Running   0          19h
calico-system          calico-node-v9lp9                                       1/1     Running   0          19h
calico-system          calico-typha-7b6f98bc78-49bf5                           1/1     Running   0          19h
calico-system          calico-typha-7b6f98bc78-7vhd6                           1/1     Running   0          19h
calico-system          csi-node-driver-6nbvr                                   2/2     Running   0          19h
calico-system          csi-node-driver-9cqq5                                   2/2     Running   0          19h
calico-system          csi-node-driver-9pm4t                                   2/2     Running   0          19h
default                nginxweb-7c88f6c77-pck4f                                1/1     Running   0          18h
default                nginxweb-7c88f6c77-rv76n                                1/1     Running   0          18h
default                nginxweb-7c88f6c77-vb8xj                                1/1     Running   0          18h
kube-system            coredns-6766b7b6bb-6dtl7                                1/1     Running   0          22h
kube-system            coredns-6766b7b6bb-lrpw7                                1/1     Running   0          22h
kube-system            etcd-k8s-master01                                       1/1     Running   0          22h
kube-system            kube-apiserver-k8s-master01                             1/1     Running   0          22h
kube-system            kube-controller-manager-k8s-master01                    1/1     Running   0          22h
kube-system            kube-proxy-6g8dn                                        1/1     Running   0          22h
kube-system            kube-proxy-kmzv6                                        1/1     Running   0          20h
kube-system            kube-proxy-pj2t5                                        1/1     Running   0          20h
kube-system            kube-scheduler-k8s-master01                             1/1     Running   0          22h
kubernetes-dashboard   kubernetes-dashboard-api-c787c9d97-5rqvp                1/1     Running   0          41m
kubernetes-dashboard   kubernetes-dashboard-auth-7ff7855689-vmltw              1/1     Running   0          41m
kubernetes-dashboard   kubernetes-dashboard-kong-79867c9c48-txfql              1/1     Running   0          41m
kubernetes-dashboard   kubernetes-dashboard-metrics-scraper-84655b9bd8-mq8h7   1/1     Running   0          41m
kubernetes-dashboard   kubernetes-dashboard-web-658946f7f9-crj69               1/1     Running   0          41m
tigera-operator        tigera-operator-ccfc44587-vswlt                         1/1     Running   0          20h
```


## 6.3 访问Dashboard前的准备

本节参考自官方文档 

- [创建示例用户](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)


### 6.3.1 创建示例用户 service-account

创建dashboard-service-account.yaml文件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

创建示例用户service-account命令

```bash
root@k8s-master01:~# kubectl create -f dashboard-service-account.yaml
```


### 6.3.2 创建 ClusterRoleBinding

创建dashboard-cluster-role-binding.yaml文件

```yaml
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

创建 ClusterRoleBinding

```bash
# 创建 ClusterRoleBinding
root@k8s-master01:~# kubectl create -f dashboard-cluster-role-binding.yaml
```



创建dashboard-admin-user.yaml 文件

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

创建 ServiceAccount、ClusterRoleBinding

```bash
root@k8s-master01:~/.cache/helm/repository# kubectl create -f /home/acanx/Dashboard/dashboard-admin-user.yaml

serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

#### 6.3.3 获取 ServiceAccount 的 Bearer Token（短期访问的Token令牌） 

```bash
# 生成toke命令
root@k8s-master01:~/.cache/helm/repository# kubectl -n kubernetes-dashboard create token admin-user

eyJhbGciOiJSUzI1NiIsImtpZCI6IjZPMURRc2RoVjl0YWJnUGU2RDVyLTRKblhybnhBaG1oS3JoVGNBTjh6UkkifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQyNzMxMjUwLCJpYXQiOjE3NDI3Mjc2NTAsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiYWM2NWZiNTItNjViMi00ZmQ5LWI4OTctMzQ4NjI4N2UzYTlmIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiNTBhNzgwZTYtODgzOC00ODI1LTk5ODktMzdhYTE0ZjM5ZjlkIn19LCJuYmYiOjE3NDI3Mjc2NTAsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.9j0DPg2sKuVfODjWBxxXN3-r1GmXCGV58LULx81fiCwE5IdCp0LK4to0T35c55yiZjkW2fc7l01k6V7uKFTCGiJ3TDJn_1JJe5lZpWBWXSWoO_q-1nkMgniGPowieHEny3ZU1wm-Kk-nrr2CYZKhXShGDbRxpnjAPf1X0aB6lAzPt59GKVYpPaMYhcEBMMT_ASZrGLYrNR31UpbEXf2UP9Cgyh9m5vpA48FOJKZNt8FRq20egHWlWCw4a4PPb261w81vcWWB1bwTHIw-8CyNZjptrPVX06ch4DT-eX5No5jr8kh2gCiq--bARESxjgDjCebBS8O96NpKRWtQMQbFAQ
```

获取到的toke是类似上述的字符串



#### 6.3.4 获取 ServiceAccount 的长期 Bearer Token （长期Token） 

- 创建dashboard-secret.yaml文件

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
```

- 获取 ServiceAccount 的长期 Bearer Token 

```bash
# 创建
root@k8s-master01:~# kubectl create -f /home/acanx/Dashboard/dashboard-secret.yaml

# 获取对应的token令牌
root@k8s-master01:~# kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 -d

eyJhbGciOiJSUzI1NiIsImtpZCI6IjZPMURRc2RoVjl0YWJnUGU2RDVyLTRKblhybnhBaG1oS3JoVGNBTjh6UkkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1MGE3ODBlNi04ODM4LTQ4MjUtOTk4OS0zN2FhMTRmMzlmOWQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.UG-iAoiCRrMeUePUi5ryn4tUOhFC94eTv_THHkrT_XZhUwC-IHqy9zkmi20tfQ-jnSnzy9_H9Z_auNw6Y4VtGB6pUs7srk6X9AIr4JDrScEVSMLbLXyCsdwGKbAaQJ8n2KA0J_pXibAiaQ77vCIDs9A4HXjn-T7GhoWEeswAcZF1N1SYbDyMxC-paMKb2I5CxL6pxtgWXuPCFkHxVnzeAT9Pswj-q0_Htr46U0Ieyo3dEBVGoOE4FA56UCYEnIfIv-OHXHytcxdaETrKl22EWleAYYENQy82qrdn8eCMK9ahOQyvCFORUbltJWP0hRkIRoMSGx1gD1DMHWMd6T9IRw
```



### 6.4 正式访问Dashboard  

参考资料 [Kubernetes 集群中安装和配置 Kubernetes Dashboard](https://blog.csdn.net/qq719779232/article/details/145212142)


### 6.4.1 方式一：使用 Port Forwarding进行命令行代理

Port Forwarding 是一种临时性的解决方案，它允许你通过本地机器上的端口直接访问集群内部的服务。这对于开发人员来说非常方便，因为他们不需要对外暴露服务即可进行调试或测试。


#### 6.4.1.1 开启访问

通过 Port Forwarding 访问 Dashboard，需要先执行如下命令：

```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 8443:443
```

这将Dashboard在 [https://localhost:8443](https://localhost:8443) 上可用。


#### 6.4.1.2 优点与缺点

- 这种方式的优点在于方法简单易用，无需更改任何集群配置，适合小规模测试或个人使用。但仅限于执行命令的本地机器，它是临时的，关闭终端后连接即断开，无法在其他机器上访问，并且每次使用时都需要手动开启端口转发。

- 也就是说，无法直接在集群外部访问，此方案不适合本次（集群外访问）实际访问场景，因此需要调整采用其他方案


### 6.4.2 方式二：修改服务类型为 NodePort

#### 6.4.2.1 方法概述

- NodePort 允许 Kubernetes 服务在各个集群节点的特定端口上对外暴露。这使得用户能够通过任意节点的 IP 地址和指定 NodePort 进行访问 Dashboard。（范围通常是 30000-32767），从而允许外部流量通过该端口进入集群并访问指定的服务。

- 也就是支持集群外部访问


#### 6.4.2.2 开启访问

编辑 Dashboard 的服务配置，将其类型更改为 NodePort：

```bash
root@k8s-master01:~# kubectl patch svc kubernetes-dashboard-kong-proxy -n kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
# TODO 若需要指定端口，可以指定一个固定的 nodePort：
root@k8s-master01:~# kubectl patch svc kubernetes-dashboard-kong-proxy -n kubernetes-dashboard --type='json' -p '[{"op":"add","path":"/spec/ports/0/nodePort","value":30000}]'
```

以下是实际操作及结果：

```bash
root@k8s-master01:~# kubectl patch svc kubernetes-dashboard-kong-proxy -n kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

service/kubernetes-dashboard-kong-proxy patched
root@k8s-master01:~# kubectl get svc -A

NAMESPACE              NAME                                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
calico-apiserver       calico-api                             ClusterIP   10.109.219.68    <none>        443/TCP                  21h
calico-system          calico-kube-controllers-metrics        ClusterIP   None             <none>        9094/TCP                 20h
calico-system          calico-typha                           ClusterIP   10.104.149.150   <none>        5473/TCP                 21h
default                kubernetes                             ClusterIP   10.96.0.1        <none>        443/TCP                  23h
default                nginxweb-service                       NodePort    10.106.65.218    <none>        80:30080/TCP             19h
kube-system            kube-dns                               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   23h
kubernetes-dashboard   kubernetes-dashboard-api               ClusterIP   10.104.42.174    <none>        8000/TCP                 128m
kubernetes-dashboard   kubernetes-dashboard-auth              ClusterIP   10.98.125.185    <none>        8000/TCP                 128m
kubernetes-dashboard   kubernetes-dashboard-external          NodePort    10.101.246.117   <none>        443:30011/TCP            7m8s
kubernetes-dashboard   kubernetes-dashboard-kong-proxy        NodePort    10.110.23.250    <none>        443:31094/TCP            128m
kubernetes-dashboard   kubernetes-dashboard-metrics-scraper   ClusterIP   10.102.211.164   <none>        8000/TCP                 128m
kubernetes-dashboard   kubernetes-dashboard-web               ClusterIP   10.107.236.126   <none>        8000/TCP                 128m

# 相关的命令1
root@k8s-master01:~# kubectl get svc -A | grep dash

kubernetes-dashboard   kubernetes-dashboard-api               ClusterIP   10.104.42.174    <none>        8000/TCP                 166m
kubernetes-dashboard   kubernetes-dashboard-auth              ClusterIP   10.98.125.185    <none>        8000/TCP                 166m
kubernetes-dashboard   kubernetes-dashboard-external          NodePort    10.101.246.117   <none>        443:30011/TCP            44m
kubernetes-dashboard   kubernetes-dashboard-kong-proxy        NodePort    10.110.23.250    <none>        443:31094/TCP            166m
kubernetes-dashboard   kubernetes-dashboard-metrics-scraper   ClusterIP   10.102.211.164   <none>        8000/TCP                 166m
kubernetes-dashboard   kubernetes-dashboard-web               ClusterIP   10.107.236.126   <none>        8000/TCP                 166m

# 相关的命令2
root@k8s-master01:~# kubectl get svc -A -n kubernetes-dashboard

NAMESPACE              NAME                                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
calico-apiserver       calico-api                             ClusterIP   10.109.219.68    <none>        443/TCP                  21h
calico-system          calico-kube-controllers-metrics        ClusterIP   None             <none>        9094/TCP                 21h
calico-system          calico-typha                           ClusterIP   10.104.149.150   <none>        5473/TCP                 21h
default                kubernetes                             ClusterIP   10.96.0.1        <none>        443/TCP                  24h
default                nginxweb-service                       NodePort    10.106.65.218    <none>        80:30080/TCP             20h
kube-system            kube-dns                               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   24h
kubernetes-dashboard   kubernetes-dashboard-api               ClusterIP   10.104.42.174    <none>        8000/TCP                 165m
kubernetes-dashboard   kubernetes-dashboard-auth              ClusterIP   10.98.125.185    <none>        8000/TCP                 165m
kubernetes-dashboard   kubernetes-dashboard-external          NodePort    10.101.246.117   <none>        443:30011/TCP            44m
kubernetes-dashboard   kubernetes-dashboard-kong-proxy        NodePort    10.110.23.250    <none>        443:31094/TCP            165m
kubernetes-dashboard   kubernetes-dashboard-metrics-scraper   ClusterIP   10.102.211.164   <none>        8000/TCP                 165m
kubernetes-dashboard   kubernetes-dashboard-web               ClusterIP   10.107.236.126   <none>        8000/TCP                 165m

```



如果显示您的连接不是私密连接，在当前页面用键盘输入thisisunsafe,不是在地址栏输入，就直接输入，页面即会自动刷新进入网页。不行就多输入几次，但是要确保输入是正确的。具体请查看这篇[博客](https://blog.csdn.net/qq719779232/article/details/144852360)


#### 6.4.2.4 访问Kubernetes Dashboard Web页面

可以看到随机开放了31094端口访问，浏览器中打开 https://192.168.11.161:31094 测试（忽略不是私密连接的提示后）可以正常打开，输入token后能够正常登录。

![Kubernetes-Dashboard-HomePage](/Node/imgs/03_02/Dashboard-Load.png)

![Kubernetes-Dashboard-HomePage](/Node/imgs/03_02/Dashboard.png)



#### 6.4.2.5 优点与缺点

- 这种方法一旦配置完成，只要集群存在就可以持续访问，提供了长期稳定的访问途径；同时可以直接通过任意节点 IP 加上指定端口号来访问服务，易于访问。

- 但是，它也带来了较高的安全风险，如果未正确配置防火墙规则，可能会导致服务被未经授权的人访问；

- 此外，还需要选择一个未被占用的端口，因此有一定的端口限制。


#### 6.4.2.6 参考资料

- [kubernetes-dashboard-7.11.1/README.md](https://github.com/kubernetes/dashboard/blob/kubernetes-dashboard-7.11.1/README.md)
- [访问仪表板](https://github.com/kubernetes/dashboard/tree/master/docs/user/accessing-dashboard)
- [69.Kubernetes(k8s)手把手教程—部署Dashboard以图形化方式管理Kubernetes](https://www.bilibili.com/video/BV15UsSeDEZA/)
- [Kubernetes(k8s) Web-UI界面(一)：部署和访问仪表板(Dashboard)](https://www.cnblogs.com/renshengdezheli/p/17513879.html)
- [二进制安装Kubernetes（k8s）v1.32.0](https://www.oiox.cn/index.php/archives/428/)

### 6.5 清理和后续步骤

删除管理员ServiceAccount和ClusterRoleBinding。

```bash
root@k8s-master01:~# kubectl -n kubernetes-dashboard delete serviceaccount admin-user
root@k8s-master01:~# kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```

了解有关如何在 Kubernetes 中授予/拒绝权限的更多信息，阅读官方[身份验证](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)和[授权](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)文档。






# 七、 实际部署过程中遇到的问题


### 7.1.1操作系统重启后无法使用原账户登录


### 7.1.2 kubeadm初始化失败，抛出错误


### 7.1.3 安装如遇到错误，需要修改配置，关闭kubelet 

```bash
~# sudo rm -rf /etc/kubernetes/manifests/kube-apiserver.yaml
~# sudo rm -rf /etc/kubernetes/manifests/kube-controller-manager.yaml
~# sudo rm -rf /etc/kubernetes/manifests/kube-scheduler.yaml
~# sudo rm -rf /etc/kubernetes/manifests/etcd.yaml

~# sudo rm -rf /etc/kubernetes/manifests/kube-apiserver.yaml && sudo rm -rf /etc/kubernetes/manifests/kube-controller-manager.yaml && sudo rm -rf /etc/kubernetes/manifests/kube-scheduler.yaml && sudo rm -rf /etc/kubernetes/manifests/etcd.yaml

sudo netstat -tulnp | grep :10250
sudo lsof -i :10250

sudo kill 2832
sudo kill -9 1234

~# systemctl status kubelet
~# systemctl stop kubelet
```

#### 7.1.3.2 重新初始化 Kubernetes 集群

- 步骤 1：重置 kubeadm\2：清理残留文件

```bash
~# sudo kubeadm reset -f && sudo rm -rf /etc/kubernetes /var/lib/etcd /var/lib/kubelet
```


-  https://blog.csdn.net/m0_51510236/article/details/136329885

```bash
root@k8s-master01:~# sudo kubeadm init \
--apiserver-advertise-address=192.168.11.161 \
--apiserver-cert-extra-sans=k8s-master01 \
--image-repository=registry.aliyuncs.com/google_containers \
--kubernetes-version=v1.32.3 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--cri-socket=unix:///run/containerd/containerd.sock \
--cgroup-driver=systemd \
--upload-certs \
--v=9
```




```bash
Mar 22 13:26:21 k8s-master01 kubelet[9760]: E0322 13:26:21.545465    9760 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-controller-manager-k8s-master01_kube-system(19dfb154dd163834eef61b7252703fed)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"kube-controller-manager-k8s-master01_kube-system(19dfb154dd163834eef61b7252703fed)\\\": rpc error: code = Unknown desc = failed to start sandbox \\\"623e76469b34076613d313938932728b1c6e63d0951c4c1a1a7563332c35b2e2\\\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/623e76469b34076613d313938932728b1c6e63d0951c4c1a1a7563332c35b2e2/log.json: no such file or directory): exec: \\\"runc\\\": executable file not found in $PATH\"" pod="kube-system/kube-controller-manager-k8s-master01" podUID="19dfb154dd163834eef61b7252703fed"
Mar 22 13:26:23 k8s-master01 kubelet[9760]: E0322 13:26:23.120378    9760 controller.go:145] "Failed to ensure lease exists, will retry" err="Get \"https://192.168.11.161:6443/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/k8s-master01?timeout=10s\": dial tcp 192.168.11.161:6443: connect: connection refused" interval="7s"
Mar 22 13:26:23 k8s-master01 kubelet[9760]: I0322 13:26:23.327532    9760 kubelet_node_status.go:75] "Attempting to register node" node="k8s-master01"
Mar 22 13:26:23 k8s-master01 kubelet[9760]: E0322 13:26:23.328029    9760 kubelet_node_status.go:107] "Unable to register node with API server" err="Post \"https://192.168.11.161:6443/api/v1/nodes\": dial tcp 192.168.11.161:6443: connect: connection refused" node="k8s-master01"
Mar 22 13:26:25 k8s-master01 kubelet[9760]: W0322 13:26:25.039914    9760 reflector.go:569] k8s.io/client-go/informers/factory.go:160: failed to list *v1.CSIDriver: Get "https://192.168.11.161:6443/apis/storage.k8s.io/v1/csidrivers?limit=500&resourceVersion=0": dial tcp 192.168.11.161:6443: connect: connection refused
Mar 22 13:26:25 k8s-master01 kubelet[9760]: E0322 13:26:25.040030    9760 reflector.go:166] "Unhandled Error" err="k8s.io/client-go/informers/factory.go:160: Failed to watch *v1.CSIDriver: failed to list *v1.CSIDriver: Get \"https://192.168.11.161:6443/apis/storage.k8s.io/v1/csidrivers?limit=500&resourceVersion=0\": dial tcp 192.168.11.161:6443: connect: connection refused" logger="UnhandledError"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: W0322 13:26:26.039053    9760 reflector.go:569] k8s.io/client-go/informers/factory.go:160: failed to list *v1.Service: Get "https://192.168.11.161:6443/api/v1/services?fieldSelector=spec.clusterIP%21%3DNone&limit=500&resourceVersion=0": dial tcp 192.168.11.161:6443: connect: connection refused
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.039171    9760 reflector.go:166] "Unhandled Error" err="k8s.io/client-go/informers/factory.go:160: Failed to watch *v1.Service: failed to list *v1.Service: Get \"https://192.168.11.161:6443/api/v1/services?fieldSelector=spec.clusterIP%21%3DNone&limit=500&resourceVersion=0\": dial tcp 192.168.11.161:6443: connect: connection refused" logger="UnhandledError"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.450839    9760 kubelet.go:3190] "No need to create a mirror pod, since failed to get node info from the cluster" err="node \"k8s-master01\" not found" node="k8s-master01"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.451717    9760 dns.go:153] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 119.29.29.29 114.114.114.114 8.8.8.8"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.547499    9760 log.go:32] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to start sandbox \"c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.547575    9760 kuberuntime_sandbox.go:72] "Failed to create sandbox for pod" err="rpc error: code = Unknown desc = failed to start sandbox \"c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH" pod="kube-system/kube-apiserver-k8s-master01"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.547611    9760 kuberuntime_manager.go:1237] "CreatePodSandbox for pod failed" err="rpc error: code = Unknown desc = failed to start sandbox \"c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH" pod="kube-system/kube-apiserver-k8s-master01"
Mar 22 13:26:26 k8s-master01 kubelet[9760]: E0322 13:26:26.547680    9760 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-apiserver-k8s-master01_kube-system(843638c9eef6e3d63d22d53c9d990887)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"kube-apiserver-k8s-master01_kube-system(843638c9eef6e3d63d22d53c9d990887)\\\": rpc error: code = Unknown desc = failed to start sandbox \\\"c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54\\\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/c54802a9d762ba695f68d28021ea9339eb8e9845d41161d6fa1f9bfb4d811c54/log.json: no such file or directory): exec: \\\"runc\\\": executable file not found in $PATH\"" pod="kube-system/kube-apiserver-k8s-master01" podUID="843638c9eef6e3d63d22d53c9d990887"
Mar 22 13:26:29 k8s-master01 kubelet[9760]: E0322 13:26:29.498553    9760 eviction_manager.go:292] "Eviction manager: failed to get summary stats" err="failed to get node info: node \"k8s-master01\" not found"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.122192    9760 controller.go:145] "Failed to ensure lease exists, will retry" err="Get \"https://192.168.11.161:6443/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/k8s-master01?timeout=10s\": dial tcp 192.168.11.161:6443: connect: connection refused" interval="7s"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: I0322 13:26:30.329597    9760 kubelet_node_status.go:75] "Attempting to register node" node="k8s-master01"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.330070    9760 kubelet_node_status.go:107] "Unable to register node with API server" err="Post \"https://192.168.11.161:6443/api/v1/nodes\": dial tcp 192.168.11.161:6443: connect: connection refused" node="k8s-master01"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.450085    9760 kubelet.go:3190] "No need to create a mirror pod, since failed to get node info from the cluster" err="node \"k8s-master01\" not found" node="k8s-master01"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.450964    9760 dns.go:153] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 119.29.29.29 114.114.114.114 8.8.8.8"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.548988    9760 log.go:32] "RunPodSandbox from runtime service failed" err="rpc error: code = Unknown desc = failed to start sandbox \"f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.549096    9760 kuberuntime_sandbox.go:72] "Failed to create sandbox for pod" err="rpc error: code = Unknown desc = failed to start sandbox \"f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH" pod="kube-system/kube-scheduler-k8s-master01"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.549145    9760 kuberuntime_manager.go:1237] "CreatePodSandbox for pod failed" err="rpc error: code = Unknown desc = failed to start sandbox \"f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b/log.json: no such file or directory): exec: \"runc\": executable file not found in $PATH" pod="kube-system/kube-scheduler-k8s-master01"
Mar 22 13:26:30 k8s-master01 kubelet[9760]: E0322 13:26:30.549256    9760 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-scheduler-k8s-master01_kube-system(c7ec3024ac27f675ae60ec998a90ce85)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"kube-scheduler-k8s-master01_kube-system(c7ec3024ac27f675ae60ec998a90ce85)\\\": rpc error: code = Unknown desc = failed to start sandbox \\\"f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b\\\": failed to create containerd task: failed to create shim task: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/containerd/io.containerd.runtime.v2.task/k8s.io/f8b4188ebe49ae964cfbbb1837c3899194dede9e323889d10636e4002993581b/log.json: no such file or directory): exec: \\\"runc\\\": executable file not found in $PATH\"" pod="kube-system/kube-scheduler-k8s-master01" podUID="c7ec3024ac27f675ae60ec998a90ce85"
Mar 22 13:26:31 k8s-master01 kubelet[9760]: E0322 13:26:31.073491    9760 event.go:368] "Unable to write event (may retry after sleeping)" err="Post \"https://192.168.11.161:6443/api/v1/namespaces/default/events\": dial tcp 192.168.11.161:6443: connect: connection refused" event="&Event{ObjectMeta:{k8s-master01.182f0852158da106  default    0 0001-01-01 00:00:00 +0000 UTC <nil> <nil> map[] map[] [] [] []},InvolvedObject:ObjectReference{Kind:Node,Namespace:,Name:k8s-master01,UID:k8s-master01,APIVersion:,ResourceVersion:,FieldPath:,},Reason:NodeHasSufficientPID,Message:Node k8s-master01 status is now: NodeHasSufficientPID,Source:EventSource{Component:kubelet,Host:k8s-master01,},FirstTimestamp:2025-03-22 13:18:49.45761511 +0800 CST m=+0.101712151,LastTimestamp:2025-03-22 13:18:49.45761511 +0800 CST m=+0.101712151,Count:1,Type:Normal,EventTime:0001-01-01 00:00:00 +0000 UTC,Series:nil,Action:,Related:nil,ReportingController:kubelet,ReportingInstance:k8s-master01,}"

```



### 7.1.4 Kubernets集群中部署calico网络插件时kubectl get pods -n tigera-operator命令一直显示pod为ImagePullBackOff状态

```bash
~# kubectl describe pod tigera-operator-ccfc44587-vswlt -n tigera-operator
```

```bash
~# kubectl get pods -n tigera-operator -o wide
```

查到“Failed to pull image "quay.io/tigera/operator:v1.36.5": failed to pull and unpack image "quay.io/tigera/operator:v1.36.5": failed to copy: read tcp 192.168.11.166:53828->23.60.96.137:443: read: connection reset by peer
”
需要手动拉取镜像或者增加镜像源（参考https://blog.csdn.net/nxg810251/article/details/145170594中的config.toml文件的Line206~Line219中的配置） 


```bash
~# sudo systemctl daemon-reload && sudo systemctl restart containerd && sudo systemctl restart kubelet
```

```bash
~# systemctl daemon-reload && systemctl restart containerd
```


#### 7.1.4.3 参考：

- [K8s的Pod出现Init:ImagePullBackOff问题的解决(以calico为例)](https://blog.csdn.net/weixin_40444270/article/details/132543441)
- [Kubernetes1.32版本搭建【超详细】](https://blog.csdn.net/nxg810251/article/details/145170594)
- [Kubernetes1.32版本搭建【极简版】](https://blog.csdn.net/weixin_42434261/article/details/144539712)



# 八、 总结

## 8.1 可以继续优化的流程

### 8.1.1 虚拟机镜像准备

### 8.1.2 containerd版本选择

### 8.1.3 镜像源加速

### 8.1.4 网络配置



