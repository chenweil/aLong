---
date: 2024-11-01
title: "Obsidian插件使用"
type: "post"
tags: ["Obsidian"]
---

# 场景
一直使用Obsidian进行记录内容,网页上很多内容觉得不错想摘录一下. 之前使用过的内容是简悦和一个Safari的脚本.

前阵子看到有人分享Obsidian的插件,装上一直没用,本周体验完记录一下.

本次记录时演示的浏览器是 `zen browser` .

# 安装插件
下面是支持浏览器的地址:

[hrome Web Store](https://chromewebstore.google.com/detail/obsidian-web-clipper/cnjifjpddelmedmihgijeibhnjfabmlf) for Chrome, Brave, Edge, Arc, Orion, and other Chromium-based browsers.

[Firefox Add-Ons](https://addons.mozilla.org/en-US/firefox/addon/web-clipper-obsidian/) for Firefox and Firefox Mobile.

[Safari Extensions](https://apps.apple.com/us/app/obsidian-web-clipper/id6720708363) for macOS, iOS, and iPadOS.


# 使用

安装完再浏览器上可以看到插件图标.

![](https://s2.loli.net/2024/11/01/NUjCu9ZJ2rwm7LT.png)

在使用之前我们先进行一些配置,点击图标后点击设置按钮:

## 基本设置

![](https://s2.loli.net/2024/11/01/BTVvyZdahUwWQSF.png)

基本设置中,最需要设置的是 Vaults 目录,可以配置一个或多个,当在插件上保存时可以切换不同的 Vault 进行保存.

输入Vault名字,回车添加.

## 属性配置

![](https://s2.loli.net/2024/11/01/69vpVCsSYlARt1q.png)

属性我没有调整,使用的默认值.等需要进阶或者修改时候再考虑.

## 荧光笔配置

![](https://s2.loli.net/2024/11/01/1GdZfazujbJRBCD.png)

荧光笔目前用起来不是很好用. 他能给网页进行高亮,通知在配置时是否显示,已经再导出内容是,是否用高亮的内容替换原文.

白话就是仅导出高亮内容.看你选择,我选择默认替换原文.

## 模版配置

![](https://s2.loli.net/2024/11/01/X5tsYpaM3GK7WLV.png)

模版我修改的地方不多,我仅把Vault设置了默认的名字,没有选择LastUsed. 如果多个库可以用lastUsed更合适.

模版有一处配置会影响导出结果:

![](https://s2.loli.net/2024/11/01/kfPMwS8hQOURaXJ.png)

默认是创建新文件. 如果需要进行同名称追加和日记可以自行选择测试.


# 导出测试

插件图标上也有些按钮,下面进行简单介绍:

![](https://s2.loli.net/2024/11/01/93d1LTeYuZv5mjQ.png)

在Add to obsidian 按钮上面是选择输出的`vault`,后面的输入是指定输出目录, 默认是**Clippings** .
## 原文导出

在浏览的网页导出原文是,点击插件图标后选择 `Add to Obsidian` .

导出的结果如下:

![](https://s2.loli.net/2024/11/01/auPQv4B96i8s2Sm.png)

如果在 `Add to Obsidian`按钮上面输入的是`testdir`,就像图片内容一下, 有一个testdir目录,里面导出了网页内容.

### 导出荧光笔内容

![](https://s2.loli.net/2024/11/01/N1fGRD6CIyLTghS.png)

插件支持导出荧光笔内容,需要现在网页上开启荧光笔,并且在设置中选择导出荧光笔内容是替换原文.

这时候在网页使用荧光笔高亮部分可以导出到Obsidian中.

导出荧光笔结果:

![](https://s2.loli.net/2024/11/01/AGw6HXK5cMTbL4v.png)


以上就是插件的使用方法.

完结

祝好
