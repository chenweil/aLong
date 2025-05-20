---
date: 2025-05-20
lastmod: 2025-05-20
showTableOfContents: false
title: Gemini制作AppleScritp一次过
type: post
tags:
  - AI
  - Arc
---
# 前提
我喜欢使用Arc浏览器，自从同厂出的Dia出来后体验AI部分功能不错。但用它当作主力还是有点Chrome的感觉。我还是喜欢使用Arc，我想能否使用插件或者脚本来实现这个功能呢。没写过插件、脚本我问了大模型告诉我 `open -a "Dia" "https://example.com"` 能实现我的想法💡。 哈哈，没错可以的。 

## 处理过程
但当我问怎么通过Arc浏览器去实现时，大模型就说法不一了。我测试了qwen3、claude3.7还有gemini2.5 pro。国内qwen3、扣子空间都是拒绝回答这个，提示违规😓。

结果是： 

一开始都说插件能搞定，但通过cursor没实现成功。 

zed我也试了试效果也不对。 他们确实制作了一个浏览器插件，我从网上生成能了icon本地加载一看，我靠他们那个插件有个按钮，点击下载了一个sh文件，打开一看 是我上面的那条命令。 🤔 这么忽悠我啊？ 

再去问问，她们说权限不行，浏览器不支持执行脚本等等等。我没开发过不懂，继续告诉他我使用的笔记本是macOS系统，能否通过系统实现这个功能。模型说可以使用Apple Script。 嗯，我没用过，那请开始你的表演。 由于其他模型回答都是写了个脚本，没说清楚具体怎么使用。我最终选择了一个不错的答案，Gemini 3.5 pro的结果。 

👉[对话分享链接](https://aistudio.google.com/app/prompts?state=%7B%22ids%22:%5B%221V9-dOuaNeFmDoqJJIe7NUeOaxDBJrFTr%22%5D,%22action%22:%22open%22,%22userId%22:%22112999297973691289741%22,%22resourceKeys%22:%7B%7D%7D&usp=sharing)

如果链接看不了，下面插入图片：
问题：
![](https://s2.loli.net/2025/05/20/gcnzFGdf3OUXVob.png)
思考内容：
![](https://s2.loli.net/2025/05/20/PbYlIsUd4VfJXRG.png)

返回的内容：

![](https://s2.loli.net/2025/05/20/xSNXMKgJbyh5ZD9.png)

实验结果： 按照模型的回答，按照方法二一次成功。方法二中对于我完全没用过的小白来说，按照文字就能操作了，很强👍。那其他模型由于没有这么细致我就待过不提了😟。

最后在Arc上加入快捷键：

![](https://s2.loli.net/2025/05/20/58EMKd1Gua4OBqW.png)

配置快捷键的具体过程是：
1. 在Arc 点`Services`， `Services Seetings` 。
![](https://s2.loli.net/2025/05/20/6yRTsEQvBrWY3H2.png)

2.  在Services 中找到 你的脚本，在右侧双击录入快捷键。
	![](https://s2.loli.net/2025/05/20/8IGzlHOQtcU2Em6.png)

我从来没用过Apple Script，也没开发过浏览器插件。 功能看起来在熟练的大佬眼里这就是个小功能，手到擒来。对于我就是作大山，我的心智是抵抗学习这些东西的，越不会越不想学。

在模型的能力下，让我通过想法到实现目的只需要1个小时左右。前期我一直考虑是否能通过插件实现，试了三次插件都没实现，其他模型会提示没办法执行脚本。也有模型提示使用`dia://xxx.com` 方式，我在dia浏览器测试不可以❌。实现后同样结果。

最后我选择在Google Al Studio的`Gemini 2.5 Pro Preview 05-06`实现此功能。让我对开发一些小玩意儿增强了自信。

同时在国内的体验中，都是拒绝回答这话题，我不知道哪句话是敏感性的😓。
![](https://s2.loli.net/2025/05/20/phoFP3KxR2ZgyLt.png)
![](https://s2.loli.net/2025/05/20/4yKYGNmlougqkhe.png)


完结
祝好
～