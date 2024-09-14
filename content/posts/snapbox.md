---
title: "Snapbox"
date: 2024-09-14T10:42:26+08:00
tags: [Tools]
---

# 一款macOS的APP
app网站 : https://snapbox.app/

我在[v2ex](https://www.v2ex.com/t/1062087)上看到此贴.


作者3.5天的时间通过Claude 3.5 Sonnet协助开发的macOS App.需要M系列芯片的设备.

看起来像是`spotlight`,`raycast`这种工具的方式启动,调出后实现与AI的一系列操作.
比如翻译、对话、优化内容等工作.

支持ollama和服务提供商API.

配置内容简单,支持prompt.

# 接入配置
![](https://s2.loli.net/2024/09/14/z83RnpaOZh56Hwu.png)

呼出主界面时,进入 `settings`.

![](https://s2.loli.net/2024/09/14/CpLyGafc6xA7eRm.png)

设置界面中的 **SELECT PROVIDER**中选择模型的厂商.

我这里配置过`Ollama`和`Custom Endpoint`.`Custom Endpoint`我选择了 **siliconflow**.

配置时,输入完 **CUSTOM ENDPOINT URL** 和 ** CUSTOM ENDPOINT API KEY**后可以获取模型.

![](https://s2.loli.net/2024/09/14/itNTZsmAGuX17Ik.png)

勾选自己使用的模型名字即可.如果API服务商不支持获取可以自己点击`+`号添加.

![](https://s2.loli.net/2024/09/14/Gmbf93cek286JBT.png)

# 功能演示

我觉得此软件的特色是:

此软件支持全局快捷键,呼出程序,快捷进入某特定prompt等.

选择文字后会被程序捕获. 免去了复制粘贴.


* 常规对话

![](https://s2.loli.net/2024/09/14/jAsTeFm9kKUSqGV.png)

复制粘贴、打字与AI交互. 快捷上支持重新生成、继续、继续输入.

放个视频演示:

https://watch.wave.video/6UP7usIlnYJ9vC0B

* Reactor 模式

此模式是监听你的指针选择,剪切板内容.快速响应.

放个视频演示:

https://watch.wave.video/KRjgWJQ5nJmSj0yX

# 最后

看到作者仅用几天时间借助大模型开发出一个功能丰富的app，令人惊叹。

既体现了大模型的强大能力，也展示了作者高超的开发技巧和创新思维。

反思自己也在日常工作中尝试利用AI模型来优化脚本编写和项目代码调整，确实感受到了效率的提升。

展望未来，AI辅助开发很可能成为主流趋势。but我们仍需谨慎对待，在享受AI带来便利的同时，也要注意培养自身解决问题的能力，而不是过度依赖AI。

完结~
感谢观看
