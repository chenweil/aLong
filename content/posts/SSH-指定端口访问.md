---
title: SSH 指定端口访问
date: 2021-03-13 18:21:50
tags: ["SSH"]
type: "post"
---
## 很尴尬
今天测试，一个通过隧道远程到设备的功能。

隧道创建完成，然后我就要ssh到哪台设备。反复连接一直不通，心里万马奔腾啊！～

查看配置没问题啊，怎么就！@#¥%！！！

后来发现 自己的锅， `ssh user@ip：prot`  Hahaha～～

## SSH连接

SSH 默认端口22，通常我们ssh 时候指令是这样的 `ssh user@ip`

### 指定端口

指定端口 我很少用，即便是改端口的我也大部分终端软件去连的。

正确方式： `ssh -p port user@ip`

记忆深刻，这下很尴尬。 特此记录。
