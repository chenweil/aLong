---
title: "Colima MacOS最小的设置运行容器运行时,替换Docker Desktop方案"
date: 2023-04-07T19:33:25+08:00
tags: [Docker, Container, MacOS]
---

# Colima
[GitHub](https://github.com/abiosoft/colima)

Colima 是一个轻量级的容器运行时，专门针对开发者在本地环境中运行容器应用。它的目标是提供一种比 Docker Desktop 更简单、更快速、更安全的容器运行时解决方案。

Colima 基于 Moby 和 LinuxKit 构建，具有以下特点：

由于它只是一个容器运行时，因此相对于 Docker Desktop，它的安装和启动速度更快，所需的资源更少。
它使用了轻量级的虚拟化技术，例如 HyperKit 和 VPNKit，以提高容器的性能和安全性。
Colima 提供了一组简单的 CLI 命令，使得用户可以轻松地启动、停止、删除容器，以及执行其他常见操作。
它还提供了一些有用的功能，例如在本地浏览器中打开容器中运行的应用程序、自动重启容器等。
总的来说，Colima 是一个轻量级、易于使用的容器运行时，旨在为开发者提供一种更快、更安全的容器环境，使得开发和测试容器应用程序变得更加轻松。

## 安装
在 macOS 上安装 Colima 很简单，只需要执行以下几个步骤：

* 首先，您需要打开终端并使用 Homebrew 包管理器安装 Colima。如果您没有安装 Homebrew，请先安装 Homebrew：
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

* 安装 Colima：
`brew install colima`

  安装完成后，您可以使用 colima 命令来启动 Colima：
`colima start` 

如果您想停止 Colima，可以使用以下命令：
`colima stop`

* 要卸载 Colima，请运行以下命令：

`brew uninstall colima`

安装完成后，您就可以使用 Colima 运行和管理容器应用程序了。

## docker交互
启动 Colima 后，您可以使用 Docker CLI 与容器交互。在 Colima 中，Docker CLI 已经预先安装了，所以您可以直接使用 `docker` 命令来管理容器。

以下是一些基本的 Docker CLI 命令：

*   `docker run` - 运行容器
*   `docker ps` - 查看正在运行的容器
*   `docker stop` - 停止容器
*   `docker rm` - 删除容器
*   `docker images` - 查看本地镜像
*   `docker pull` - 下载镜像
*   `docker push` - 上传镜像

请注意，当您在 Colima 中运行容器时，它们将运行在 Colima 虚拟机中，而不是您的本地计算机上。因此，在使用 Docker CLI 命令时，请确保将其指向 Colima 虚拟机，而不是本地 Docker 安装。

您可以使用以下命令来配置 Docker CLI，使其指向 Colima 虚拟机：

bash

```bash
eval "$(colima env)"
```

此命令将设置环境变量，使得 `docker` 命令指向 Colima 虚拟机中的 Docker 服务。在执行此命令后，您就可以直接使用 `docker` 命令来管理 Colima 中的容器了。

## Colima 部分操作命令说明
以下是一些常见的 Colima CLI 命令及其说明：

*   `colima start` - 启动 Colima 容器运行时。
*   `colima stop` - 停止 Colima 容器运行时。
*   `colima restart` - 重启 Colima 容器运行时。
*   `colima status` - 显示 Colima 容器运行时的状态信息。
*   `colima ssh` - 通过 SSH 连接到 Colima 容器运行时。
*   `colima ip` - 显示 Colima 容器运行时的 IP 地址。
*   `colima info` - 显示有关 Colima 容器运行时的详细信息，包括版本、磁盘使用情况和安装路径等。
*   `colima doctor` - 运行诊断程序以检查 Colima 容器运行时的配置和设置是否正确。
*   `colima web` - 在本地浏览器中打开容器中运行的应用程序。

![isS4jKa](https://i.imgur.com/isS4jKa.png)

比如你想修改docker源的地址，可以通过`colima ssh` 进去之后，编辑 Docker 的配置文件 `/etc/docker/daemon.json`。

除了以上列出的常见命令外，Colima 还提供了许多其他有用的命令和选项。您可以使用 `colima --help` 命令来查看完整的命令列表和选项说明。

## Colima自定义虚拟机
Colima 使用虚拟机技术来提供容器运行时环境。默认情况下，Colima 会自动创建和配置虚拟机。但是，如果您需要更改虚拟机的配置或行为，可以使用以下命令来配置 Colima 的虚拟机：

*   `colima config set` - 设置指定的 Colima 配置项。
*   `colima config get` - 获取指定的 Colima 配置项的值。
*   `colima config unset` - 删除指定的 Colima 配置项。
*   `colima config inspect` - 显示所有 Colima 配置项的当前值。

在初始启动时，Colima 使用用户指定的运行时，默认为 Docker。其他还有 Containerd、Kubernetes。

以下是一些常用的 Colima 配置项：

*   `vm-cpus` - 虚拟机的 CPU 核心数量。
*   `vm-memory` - 虚拟机的内存大小。
*   `vm-disk-size` - 虚拟机磁盘的大小。
*   `vm-network` - 虚拟机的网络配置，如 IP 地址、网关和 DNS 服务器等。
*   `docker-version` - Colima 容器运行时中 Docker 的版本号。

例如，要将虚拟机的 CPU 核心数量设置为 4，可以使用以下命令：

```arduino
colima config set vm-cpus 4
```

完成以上命令后，重新启动 Colima 容器运行时即可使设置生效：

`colima restart`

您可以使用 `colima config get` 命令来查看当前的 Colima 配置项值。如果需要删除某个配置项，可以使用 `colima config unset` 命令。如果需要查看所有 Colima 配置项的当前值，可以使用 `colima config inspect` 命令。

## 重启电脑后
重启电脑后，需要先启动colima的之前创建的虚拟机。 默认启动上次关闭的虚拟机。 `colima staer` 启动之后，就可以进行docker的操作了。

完结
祝好～
