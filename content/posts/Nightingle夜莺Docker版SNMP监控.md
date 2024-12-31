---
date: 2023-06-23
# description: ""
# image: ""
lastmod: 2023-06-23
showTableOfContents: false
tags: [prometheus,categraf]
title: "Nightingle夜莺Docker版SNMP监控"
type: "post"
---

## 起因
对夜莺很感兴趣，想使用一下。我看官方提供了v6版本的docker-compose。而且我之前有使用过promtheus和grafana，虽然很好但是总觉得还是得二开。总有一天有人去搞一个不错的玩意儿出来。[官方文档地址](https://flashcat.cloud/docs/)

## 安装与配置
直接运行docker版本的demo，启动后，我发现有prometheus和categraf。但我想根据官方文档使用VictoriaMetrics单机版本。
### 使用VictoriaMetrics
我在compose中去掉了prometheus，然后在主机安装了VictoriaMetrics。我没用在compose中添加，我懒改了。直接看官当文档多香。

后面启动后修改数据源：
![ZviA7j7](https://img-blog.csdnimg.cn/img_convert/13af579a7f90bfb9a5d197de818b08ab.png)

添加好数据源后，看下是否正常：
时序指标随便敲敲看有没指标
![WxPg5Rs](https://img-blog.csdnimg.cn/img_convert/3cb79c47b991fef817056f3dc5bab48c.png)
看到有指标我很自信的说成了。

后面我加了两个Ping，发现可以没问题的。

![JoY5v64](https://img-blog.csdnimg.cn/img_convert/60fd758e67e9dc0fd712bae376735e00.png)
使用内置的展示仪表盘美滋滋。

### SNMP部分
之后我想测试SNMP部分，这是我工作需要的内容。

首先默认的官方demo没有提供相关categraf插件配置。他的插件都是在github上面的。下载一份放到demo的categraf/conf目录下。
我的路径： `home/xxx/nightingate/docker/categraf/conf`

我这里用到snmp， 就拷贝input.snmp 这个目录到上面路径内。

进入这个`input.snmp`,里面有两个文件,一个 `snmp.toml`另一个是`snmp.toml.example` 。实际使用的是第一个，在里面编辑内容：
```toml
[[instances]]
  agents = ["udp://10.10.10.2:161"]
  timeout = "5s"
  version = 2
  path = ["/usr/share/snmp/mibs"]
  community = "public"
  agent_host_tag = "switch"
  retries = 1

[[instances.field]]
  oid = ".1.3.6.1.2.1.1.3.0"
  name = "uptime"

[[instances.field]]
  oid = ".1.3.6.1.2.1.1.5.0"
  name = "source"
  is_tag = true

[[instances.table]]
  oid = "IF-MIB::ifTable"
  name = "interface"
  inherit_tags = ["source"]
  index_as_tag = true
  include_filter = ["ifIndex:2","ifIndex:4"]

[[instances.table.field]]
  oid = "IF-MIB::ifDescr"
  name = "ifDescr"
  is_tag = true
[[instances.table.field]]
  oid = "IF-MIB::ifPhysAddress"
  name = "ifPhysAddress"
  is_tag = true
```

### 遇到问题
保存重启categraf后，我只看到了一个指标`snmp_uptime`。我单独取`uptime`这个是有的，就好像没办法执行`snmpwalk` 只能搞`snmpget`。

通过docker log我看到categraf中关于snmp部分报错：
*metrics_agent.go:255: E! failed to init input: local.snmp error: initializing table interface ins: translating: exec: "snmptranslate": executable file not found in $PATH*

我开始以为是没有mibs文件导致的，所以我在docker-compose.yaml中挂在了宿主机的mibs。结果是重启categraf无效。

我问了下GPT这句话怎么肥事，回答内容：
*这个错误信息表明 Nightingale 监控软件在尝试初始化 SNMP 输入时遇到了问题。具体来说，它无法找到名为 "snmptranslate" 的可执行文件。*

看这个名字大概就知道，他主要功能是翻译转换吧。大概就是MIB和OID的关系了。这部分内容建议看看SNMP的一些内容。n9e官方提供了一些相关内容：[SNMP插件](https://flashcat.cloud/docs/content/flashcat-monitor/categraf/plugin/snmp/)

好，既然我知道是缺少这个，我就进到容器里去看看，果然没有，算你蒙对了。我就自己安装一个吧，在容器内安装命令：
`sudo apt update`，`sudo apt install snmp`，这个容器进来就是root，所以直接装就好了。装好之后，重启categraf果然错误变了:
*metrics_agent.go:255: E! failed to init input: local.snmp error: initializing table interface ins: translating: Created directory: /var/lib/snmp/cert_indexes /usr/share/snmp/mibs/iana: No such file or directory /usr/share/snmp/mibs/ietf: No such file or directory MIB search path: /root/.snmp/mibs:/usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf Cannot find module (IF-MIB): At line 1 in (none) IF-MIB::ifTable: Unknown Object Identifier: exit status 2*

这个意思大概是他没找到MIB文件。我可能挂载有问题吧，既然这样我就直接在容器折腾了，`apt-get install snmp-mibs-downloader`, 装完之后需要创建一个目录`~/.snmp`,并在这目录中创建一个`snmp.conf`，文件内容：`mibs +ALL`。

之后我手动执行了categraf test参数，来直接测试snmp部分内容。 容器内部执行 `/usr/bin/categraf --configs /etc/categraf/conf --test --inputs snmp`如果执行没出现任何error，应该是ok的。

然后在界面查询指标：
![[Pasted image 20230627093410.png]]
都有了，那么也就是我的折腾是有效的。

#### 进行调整
整理上面的操作，我们需要对原来的镜像进行调整，所以我拉取他的代码，在上面进行魔改。

进入源码目录：
![RQgBBdl](https://img-blog.csdnimg.cn/img_convert/5088afb060eda1ebd614c8a5e43cdfa8.png)
可以看到他的DockerFile是在docker目录下。

```Dockerfile
FROM ubuntu:22.10

RUN echo 'hosts: files dns' >> /etc/nsswitch.conf

RUN set -ex && \
    mkdir -p /usr/bin /etc/categraf

COPY categraf  /usr/bin/categraf

COPY conf /etc/categraf/conf

COPY entrypoint.sh /entrypoint.sh

CMD ["/entrypoint.sh"]
```

简单明了，把程序搞过来，配置搞过来完事了，那我加上相关snmp部分配置就好啦。

先把go程序编译放到此目录，然后我看copy一个conf目录，但是我在使用官方demo都是挂在宿主机上的conf，所以这里我给他一个空conf就好啦。docker目录下创建一个conf。在`RUN`命令加入安装SNMP命令。

```Dockerfile
#最终魔改后
FROM ubuntu:22.10

RUN echo 'hosts: files dns' >> /etc/nsswitch.conf \
    && apt-get update \
    && apt-get install -y snmp snmp-mibs-downloader \
    && mkdir -p ~/.snmp \
    && echo "mibs +ALL" > ~/.snmp/snmp.conf \
    && apt-get clean

RUN set -ex && \
    mkdir -p /usr/bin /etc/categraf

COPY categraf  /usr/bin/categraf

COPY conf /etc/categraf/conf

COPY entrypoint.sh /entrypoint.sh

CMD ["/entrypoint.sh"]
```

最后docker build 一个镜像。使用魔改后的就不需要再折腾容器了。

如果你还是想体验夜莺SNMP相关，又不想自己build呢？
没问题用我的这个[镜像地址](chenwl2016/categraf:0.3.16)来拉取替换官方版本体验SNMP。

在夜莺demo中修改docker-compose.yaml
```yaml
...
services:
  ...
  categraf:
    image: "chenwl2016/categraf:0.3.16"
    ulimits:
      nofile:
        soft: 8192
        hard: 8192
    cap_add:
      - NET_RAW
    cap_drop:
      - NET_RAW
    container_name: "categraf"
    hostname: "categraf01"
    restart: always
    environment:
      TZ: Asia/Shanghai
      HOST_PROC: /hostfs/proc
      HOST_SYS: /hostfs/sys
      HOST_MOUNT_PREFIX: /hostfs
    volumes:
      - ./categraf/conf:/etc/categraf/conf
      - /:/hostfs
      - /var/run/docker.sock:/var/run/docker.sock
    network_mode: host
    depends_on:
      - n9e
      - ibex
...
```

最终docker版本的SNMP监控可以使用了。具体SNMP配置还是需要看官方文档了。

完结

祝好～
