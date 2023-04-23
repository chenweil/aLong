---
title: "通过minikube部署K8S学习环境"
date: 2023-01-06T15:35:43+08:00
tags: ["Linux","Kubernetes"]
---

Minikube 是本地 Kubernetes，专注于让 Kubernetes 易于学习和开发。

# MacOS部署方式:
[官方教程](https://minikube.sigs.k8s.io/docs/start/)

本次使用的驱动是 parallels驱动。所以本机需要安装此软件。
安装kubectl：`brew install kubectl`
安装minikube：
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64

sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### 通过minikube启动集群
```shell
minikube start \
--kubernetes-version=v1.23.9 \
--image-mirror-country='cn' \
--registry-mirror=https://？？？？？.mirror.aliyuncs.com
```

因为是国内，所以`--image-mirror-country='cn'`。通过`--registry-mirror`为 Docker daemon 配置镜像加速。例如上面是阿里云镜像服务地址。
因为我学习的版本是1.23.x，所以制定了版本`--kubernetes-version=v1.23.9`。
默认他读去了驱动配置，识别是`parallels`,自己定义方式`--driver=hyperkit`。

安装结果：
![C3AFxdiPbq8aIuR](https://s2.loli.net/2023/01/06/C3AFxdiPbq8aIuR.jpg)

*错误*：
如果存在错误请看具体上面的问题，很多情况都能直观看到提示。其中图标是❗️提示一些问题、💡给你一些建议等等。

####  驱动选择：
使用docker驱动需要安装docker desktop。其他驱动：virtualbox、vmware、ssh、Hyperkit等。具体详见[drivers page](https://minikube.sigs.k8s.io/docs/drivers/)

### 验证结果

* 查询minikube状态
![otU8LYBjuFeDa31](https://s2.loli.net/2023/01/06/otU8LYBjuFeDa31.png)
* 进入面板
![FmrBXOTJ1N2AZgU](https://s2.loli.net/2023/01/06/FmrBXOTJ1N2AZgU.png)
![IPBCwUSx7Fb28Ee](https://s2.loli.net/2023/01/06/IPBCwUSx7Fb28Ee.png)

* kubectl控制
`minikube kubectl -- get pods -A`
![EziUHcpln27ufeB](https://s2.loli.net/2023/01/06/EziUHcpln27ufeB.png)

kubectl在minikube中使用，需要写 `nimikune kubectl --`后面加入执行的命令。
macos默认使用的zsh，在zsh加入alias就可以敲的少的了。
```zsh
vi ~/.zshrc

#加入一行
alias kubectl="minikube kubectl --"

#退出
source ~/.zshrc
```

* 再创建一个Pod
创建：
`kubectl run nginx --image=nginx:alpine`
查询：
`kubectl get pods nginx`
详情：
`kubectl describe pods nginx`
![pYPqIgt84LzBcEU](https://s2.loli.net/2023/01/06/pYPqIgt84LzBcEU.png)

到这里k8s可以使用了。

# Ubuntu22.04部署方式
我在续集上安装了一个Ubuntu的操作系统。在上面安装一遍，这次驱动是docker。
装完系统后，安装docker。
[官方手册](https://docs.docker.com/engine/install/ubuntu/)
1. 准备源操作
```shell


sudo apt-get remove docker docker-engine docker.io containerd runc

sudo apt-get update

sudo apt-get -y install \
ca-certificates \
curl \
gnupg \
lsb-release
  
sudo mkdir -p /etc/apt/keyrings && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

2. 安装Docker Engine
```shell
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

#给当前用户加入docker组，加入完成注销再登陆
sudo usermod -aG docker $USER

# 自启
sudo systemctl enable docker.service

```

3. 安装minikube
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
![b8ZMAqLeOviN19D](https://s2.loli.net/2023/01/06/b8ZMAqLeOviN19D.jpg)

4.  部署k8s
```shell
minikube start \
--kubernetes-version=v1.23.9 \
--image-mirror-country='cn' \
--registry-mirror=https://????????.mirror.aliyuncs.com
```
![CkRH2nNBodxzh6F](https://s2.loli.net/2023/01/06/CkRH2nNBodxzh6F.jpg)

可以看到装完他还提示我 kubectl没有。
 5. 安装kubectl
`curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl`

`sudo chmod +x kubectl && sudo  mv ./kubectl /usr/local/bin/kubectl `

 ![rOksbmSFMdhXipu](https://s2.loli.net/2023/01/06/rOksbmSFMdhXipu.jpg)

到此再Ubuntu2204也部署完成。

后面继续学习

祝好～

