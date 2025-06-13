---
date: 2025-06-13
lastmod: 2025-06-13
title: "Dia浏览器体验记录"
type: "post"
---

早在论坛拿到邀请🐎，下载打开一看，这啥啊这事？ 这么简约？

# **当个AI提问**

最开始就用它来当个AI，做一些简单事情。 比如打开一个文章、帖子，问他讲了什么。 根据内容再提出其他问题。

在浏览的内容点击 `chat` 按钮或者快捷键 `command(ctrl) + e`。

也可以在地址栏输入 选择chat。

对话支持文件上传。

例子：

![](https://s2.loli.net/2025/06/13/8CAzPBVKm2Hvxl3.png)
*演示图片瑕疵，我后面又看了其他主题。截图时没注意。 :shushing_face:*

# **生成内容**

好看完了帖子，想表达却又不知道怎么说呢？ 让他搞吧。

此时输出内容UI不同，这个insert按钮需要你在网页定位到能输入的地方。

另一个例子：

![](https://s2.loli.net/2025/06/13/RCmoUHdLqVfFDxI.png)


# **个性化设置**

![](https://s2.loli.net/2025/06/13/aH5LtABcV8P47f1.png)

## Personalize 四个选项我的理解

* 第一项看起来是品味。 我不太理解，大概是你喜欢什么品牌、作家等的风格吧。比如鲁迅分、川普风。🐒

* 第二项那里，如果你常看见他时不时出来英文，也可以在这里提醒他 speak chinese。 具体描述你喜欢什么形式内容更容易接受。比如分段落展示、脑图、图标等等。

* 第三项 我理解是输出的风格、形式等吧。比如加不加emoji。

* 第四项是关于编程偏好的设定，比如你用什么语言、什么架构、什么框架等等等。

每个选项不一样，我这里演示第三个 关于 **Wirte** 的风格。

这里我需要提醒他，如果我访问的内容是英文时，你输入内容也相同。

这样才能与世界通话嘛。 😂

🔔提示内容：
`If the tab content is in English, please generate content that is also in English.`

输出：

![](https://s2.loli.net/2025/06/13/3IhGqwVxMu4X1id.png)

如果没有这句提示，他会根据对话语言，输出中文夸赞内容。😮‍💨

![](https://s2.loli.net/2025/06/13/iPckQz3Ajgpl9eS.png)


# **多标签对话**

比如以Hugo 文档为例，他有四个页面。 抛出quick start 还有三个。如果我直接问模型可能他会告诉我内容，但我想自己再去看看文档又不知道再哪段（比如内容多很多）时。 可以都打他 用`@`去选择这些tab页。

![](https://s2.loli.net/2025/06/13/GwFem6aAlCI21Wn.png)

我上面问了关于修改主题的话题，然后问哪个文档有提到。

![](https://s2.loli.net/2025/06/13/dkBMXGtQY725ZmC.png)

除此还能想到比如一个新闻多个页面去总结、对比。 不同产品或网站的东西对比、询问等。

# **分屏(Split)**

在arc里感觉分屏好爽。但在这里我接受dia的操作很难受。

拖拽可以实现分屏，但才做不当就是拖成了单tab窗口。😓

`command+拖拽 `操作我拖拽第一个tab会和第二个进行组合。但再和第三个我就不得通过右键 add split 实现三个窗口分屏显示。

组合的分屏，右键其他tab选择 `Open as Split` 可能更舒服吧。

拆分： tab 右键 `Separate All Tabs`。

![](https://s2.loli.net/2025/06/13/taS2rzVjvLukWFH.gif)

# **多用户切换**

基操，简单水一嘴。

![](https://s2.loli.net/2025/06/13/iNcXktJEyonUrVf.png)

这个配置在设置-Profiles进行设置。

![](https://s2.loli.net/2025/06/13/Lw8y726jmNKXstB.png)

以上是我目前使用后想到的内容。 虽然他看起来说是药搞什么新的形式 巴拉巴拉的。 但目前我觉得是把AI和浏览器的整和的雏形。

比如我想在收藏夹找一个吃灰很久的内容， 我不知道那个浏览器现在能支持。

所以我目前还是当个AI对话窗口，他目前比直接将链接粘贴到 对话窗口要省一步，其他的我还没体验到有多好。 现在我依然喜欢使用arc作为主力，dia只是在我想让他打开时通过arc跳转到dia。 这一步我也省了一下，就是 [Arc通过applescritp跳转到Dia访问](https://blog.51ai.vip/posts/gemini制作applescritp一次过/) 。
