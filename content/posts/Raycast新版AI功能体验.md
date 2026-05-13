---
date: 2025-07-31
lastmod: 2025-07-31
showTableOfContents: false
title: Raycast新版AI功能Custom Providers体验
type: post
tags: ["Raycast", "AI"]
---
之前使用第三方的转发体验了三方API的接入。现在最新版支持**Custom Providers** 功能了。
# 通过Custom Providers 接入API

![](https://cdn.img2ipfs.com/ipfs/QmTxSPrSUnnUTvjxssZeEfbvwCycqtBJwvNAPLuq9rMKRg?filename=Pasted%20image%2020251220165549.png)

在 Raycast 中，添加自定义模型（Custom Providers）的具体位置如下：

1. 打开 **Raycast**。
2. 进入 **Settings**（设置）。
3. 点击左侧的 **AI** 选项卡。
4. 第一次需要在**AI**选项最底下的**Experiments**中开启**Custom Providers**。
5. 在 **AI** 选项卡页面，找到 **Custom Providers**（自定义提供商）部分。
6. 点击 **Reveal Providers Config**（显示提供商配置）按钮，这会在 Finder 中打开配置文件目录。
![](https://cdn.img2ipfs.com/ipfs/QmdKVpFQKKb7WHNiam7aVz4cooLQhhY3LVJMvT9FRxTbBk?filename=Pasted%20image%2020251220165604.png)
点击后会跳转到具体目录中。默认是有一个**providers.template.yaml**名字的文件。这个文件是一个模板，根据模板创建一个名称为**providers.yaml** 。
![](https://cdn.img2ipfs.com/ipfs/QmP89KBF9H5HhKdpsz5tS5WBh37qa1W4asodZx9G6Hv1U2?filename=Pasted%20image%2020251220165613.png)
通过dia 我询问了一下，可能 **raycast仅加载此文件。** 其他名称的我没做测试。

![Pasted%20image%2020251220165649.png](https://cdn.img2ipfs.com/ipfs/QmR3zWHdBNbu88pE6LFDcA1zL4aLLhqjQZKDvJLufC6GMy?filename=Pasted%20image%2020251220165649.png)

## 配置providers
我这里偷个懒，直接使用claude code进行配置，告诉他一些参数。 

![](https://cdn.img2ipfs.com/ipfs/QmXFs1NUdAYnHQ85qTUHXDScXFa6okpEVd21ngH7dsLM2U?filename=Pasted%20image%2020251220165850.png)

他呱唧呱唧的写完了大致的配置，我在base_url 加入了/v1，不加实际测试不正常。
api_key 我也改写成真实的(之前告诉他的是一个假的为了隐私)。 

通过测试没问题，可以使用。但有些能力限制。

# custum providers 体验
接入后，对话是没问题的。gemini 2.5 支持联网搜索，但实际体验下来不可以。
可以打开本地的一些程序，比如浏览器，apple music等等。 
MCP支持有问题，我不知道是模型的事还是什么。比如通过exa搜索会返回error。 这是我在其他工具中没遇到的。

![Pasted%20image%2020251220165919.png](https://cdn.img2ipfs.com/ipfs/Qmad4KXvCeu5g5KALnvt3niNCrt12rkWy8d56CQzKmtMGP?filename=Pasted%20image%2020251220165919.png)

使用raycast支持的模型，比如填写gemini key 的模型支持联网，调用MCP都OK。
如图：

![](https://cdn.img2ipfs.com/ipfs/QmaG4shM62KQDMCjZuxkF7jkagfpPatQEHLp7drP6sGqE1?filename=Pasted%20image%2020251220165932.png)

在raycast可以判断你接入的模型支持什么：

![](https://cdn.img2ipfs.com/ipfs/Qmf3ARKSB1DEG8eurY9HKTo9NfEX69ZxFhB7m8syMSahTX?filename=Pasted%20image%2020251220165940.png)

我在dia询问大致也是类似的回答吧：

![](https://cdn.img2ipfs.com/ipfs/QmbLyCukN7pgbWdq7A52Vyd98Lmbe23edHzGrC3JVFHShj?filename=Pasted%20image%2020251220165947.png)

以上就是大致的体验。Custom Providers给接入第三方接入提供了便利，让我不需要在本机开启其他proxy。 直接使用自定义的API，期待这方面后续的发展。

完结
祝好
🍀