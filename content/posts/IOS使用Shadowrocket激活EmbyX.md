---
title: "IOS使用Shadowrocket激活Emby"
date: 2022-11-16T10:44:11+08:00
draft: false
---

前提需要手机安装ShadowRocket。没有可以自己去找共享ID。
![aMKuwbEe1Y3BVgF](https://s2.loli.net/2022/11/16/aMKuwbEe1Y3BVgF.png)
长按后弹出选项：
![1HKa2Vywfq9uU8L](https://s2.loli.net/2022/11/16/1HKa2Vywfq9uU8L.png)
打开配置，例如在default上添加规则。点击default，编辑纯文本。
![aMNLtnGY6TxwuHS](https://s2.loli.net/2022/11/16/aMNLtnGY6TxwuHS.png)
在文本最下面加入内容：
```conf
[Script]
EmbyPremiere = type=http-response,script-path=https://gitlab.com/iptv-org/embypublic/-/raw/master/Script/EmbyPremiere.js,pattern=^https?:\/\/mb3admin.com\/admin\/service\/registration\/validateDevice,max-size=131072,requires-body=true,timeout=10,enable=true

[MITM]
hostname = mb3admin.com
```
![S3tpX5xZlwqKCYe](https://s2.loli.net/2022/11/16/S3tpX5xZlwqKCYe.png)
点击保存后，点右侧!进入。点击HTTPS解密。
![oP4D65aX2MKbwxO](https://s2.loli.net/2022/11/16/oP4D65aX2MKbwxO.png)
![5pQf9oy73ISxi1M](https://s2.loli.net/2022/11/16/5pQf9oy73ISxi1M.png)
开启HTTPS加密，会生成证书。同意安装描述文件。右上角点击勾。
![gnRNIqTBvfZVsMd](https://s2.loli.net/2022/11/16/gnRNIqTBvfZVsMd.png)
![3gsU81y6Av9OoFT](https://s2.loli.net/2022/11/16/3gsU81y6Av9OoFT.png)
去手机设置中，通用->描述文件与设备管理->点击描述文件(名称大概有Shadowrocket那个)->安装。
![wVqbC231FH945Zn](https://s2.loli.net/2022/11/16/wVqbC231FH945Zn.png)
注意在Shadowrocket的首页上，全局路由选择配置。

最后进入Emby软件。可能会提示证书点OK。如果没有提示，点一个影片后也会提示，点OK。如果Shadowrocket开启通知可以看到提示认证成功。
![TgfoqKyI2pFNrt8](https://s2.loli.net/2022/11/16/TgfoqKyI2pFNrt8.png)
后面就可以畅快看影视啦。

下次进入Emby时不需要Shadowrocket的配置了。当出现付费提示再开启那个配置就可以激活了。

完结～
祝好！
