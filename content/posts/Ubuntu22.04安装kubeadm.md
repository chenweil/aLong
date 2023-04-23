---
title: "Ubuntu22.04安装kubeadm"
date: 2023-01-11T16:30:47+08:00
tags: ["Ubutun","Liunx","Kubernetes"]
---

# 学习k8s
学习k8s做笔记，通过kubeadm搭建1master、2node。网络插件：flannel。系统Ubuntu22.04


## 系统安装docker
[Docker官方手册](https://docs.docker.com/engine/install/ubuntu/)

master、node设备各安装docker、kubelet、 kubeadm、 kubectl。

## 为kubeadm准备
master、node进行准备工作：

1. 将cgroup处理改成systemd

nano /etc/docker/daemon.json
```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```

`sudo systemctl enable docker`
`sudo systemctl daemon-reload`
`sudo systemctl restart docker`

2. 为了让 Kubernetes 能够检查、转发网络流量，你需要修改 iptables 的配置，启用“br_netfilter”模块

`nano /etc/modules-load.d/k8s.conf`

```ini
br_netfilter
```


`nano /etc/sysctl.d/k8s.conf`

```ini
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
```
`sudo sysctl --system`

3. 修改“/etc/fstab”，关闭 Linux 的 swap 分区，提升 Kubernetes 的性能
```ini
sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
```


### 安装kubeadm
[Kuneadm官方手册](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
**在master、node上都需要安装**

* 更新 `apt` 包索引并安装使用 Kubernetes `apt` 仓库所需要的包：
```shell
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl
```
* (有条件)下载 Google Cloud 公开签名秘钥：
```shell
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
``` 
如果上面访问不到，可选择其他,例如[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/kubernetes/)提供：
导入gpg key
```shell
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

创建`/etc/apt/sources.list.d/kubernetes.list`，内容：
```shell
deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main
```

之后 `sudo apt update` 更新一下。

* 安装所需的1.23.x 版本：
`sudo apt install -y kubeadm=1.23.3-00 kubelet=1.23.3-00 kubectl=1.23.3-00`

* 锁定这三个软件的版本:
`sudo apt-mark hold kubelet kubeadm kubectl`

## Master配置
```shell
kubeadm init \
--apiserver-advertise-address=172.20.20.12 \
--kubernetes-version v1.23.3 \
--pod-network-cidr=10.20.0.0/16
```


安装完成后按提示执行：
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

同时还有一个提示：
*Then you can join any number of worker nodes by running the following on each as root:*
下面那行命令是其他节点就爱如集群的命令。

Master的token是有时效性的，默认超过24H后失效，要再创建新的token。在master上执行`kubeadm token create --print-join-command`

* 安装网络插件flannel
[kube-flannel.yml](https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml "kube-flannel.yml")
修改 `net-conf.json`： NetWrok地址，与Kubernetes的`--pod-network-cidr` 网段地址。

修改完成，`kubectl apply -f kube-flannel.yml`。 

## Node节点配置
Node安装好kubeadm之后，通过join指令加入。
例如： 
```shell
sudo kubeadm join 172.20.20.12:6443 --token 80duqn.gzxz6rv5gkgosxkb \
--discovery-token-ca-cert-hash sha256:84d55c1168e9830b4f66e1c0816a218da55f96cce50f750456186fcc726a79ab 
```

## 验证
在Master上`kubectl get node` 查看。

完结
祝好