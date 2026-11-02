---
title: "官方婉拒了我的 PR，于是我给它写了个 Mod"
date: 2026-11-02
lastmod: 2026-11-02
draft: false
tags: [herdr, mod, 开发工具]
type: "post"
description: "Herdr 的 Files 面板不支持复制文件路径。PR 被婉拒后，我按官方建议做了一个 Mod：Along。"
source: "https://github.com/chenweil/herdr-along"
author: "wren"
showTableOfContents: true
---

上个月给 Herdr 提了个 PR，只加一个右键菜单项。被拒了。

于是有了这个东西。

# 起因

每天和 agent 对话，重复最多的动作是：把文件路径贴进对话框。

用 Herdr 的时候，这个动作有个断点。Files 面板能看目录、能点开文件，但想拿到路径，只能手动拼。我在 [herdr-app](https://herdr.dev) 的 Files 面板上右键试了好几次，菜单里没有这一项。

于是去提了个 PR，加上「复制路径」。

# 官方说：这不该由核心来做

PR 被婉拒了。回复很客气，意思也很清楚：这类功能他们打算交给 Layout 和 Mod 去做，核心保持精简。

我一开始有点不服，后来想想这个取舍是合理的。编辑器类工具的菜单会无限膨胀，每个用户要的都不一样。核心只提供稳定的面板和扩展点，剩下的让社区去长，长期看更划算。

那就按他们的方式来。

# 于是有了 Along

Along 是 Herdr 的一个 Mod，给 Files 和 Changes 面板加了一个右键菜单：

![](https://img.51ai.vip/20261102135229567.png)

在 [herdr.dev/mods](https://herdr.dev/mods/) 搜 `along` 就能找到，代码在 [chenweil/herdr-along](https://github.com/chenweil/herdr-along)。

# 安装

Herdr 的 Mod 是通过 agent 装的。把这句话丢给你的 agent：

```text
Read ~/.config/herdr-app/AGENTS.md, then add this layout:
https://github.com/chenweil/herdr-along/tree/main/along
```

仓库里有两个包，按需要选：

| 包 | 类型 | 说明 |
| --- | --- | --- |
| `along` | Layout | 整个窗口，Files 和 Changes 换成 Along 的版本。装完在设置 `Look` → `LAYOUT` 里选 **along** |
| `along-panels` | Mod | 只有面板，不动你现有的窗口布局。Files 默认停在右侧，Changes 自己摆 |

只要面板的话，把地址换成 `.../tree/main/along-panels`，提示词里的 `layout` 换成 `mod`。

不想用 agent 也可以手装：把 `along/layouts/along.json` 复制到 `~/.config/herdr-app/layouts/`，`along/mods/` 下每个目录复制到 `~/.config/herdr-app/mods/`，重启 Herdr，切到 `along` 布局。

装完打开设置，`Look` 里的 `LAYOUT` 下面会多一个 **along**：

![](https://img.51ai.vip/20261102135618640.png)

# 用起来

在 Files 或 Changes 的文件上右键：

![](https://img.51ai.vip/20261102135849215.png)

三个动作：

- **Copy Path**：绝对路径
- **Copy Relative Path**：相对当前面板目录或仓库根目录
- **Copy Name**：只有文件名

值为空、或者和上一行重复的行会自动去掉，菜单不会出现两行一样的。

顺手还加了两个东西：面板头部的 **Machines** 按钮，能列出 herdr-core 里的机器并开关；Changes 面板底部有提交栏，填完信息点 Commit 就提交。

日常最常用的还是复制路径。粘到对话框里，agent 不用再猜我说的是哪个文件。

# 它是怎么做的

没有从零写。Along 的 `along-files` 和 `along-changes` 是 Herdr 自带 `files`、`changes` 面板的改造版。Herdr 本体是 Apache 2.0，改完继续沿用同样的协议，归属写在 NOTICE 里。

这个 Mod 没什么野心：官方已经把面板写好了，我只补上用户真正会按的那几个动作。

# 几句提醒

Mod 是受信任的代码，不在沙箱里跑。Along 会调 `sh` / `powershell` 列目录、调 `git` 取仓库状态、用 `readTextFile` 读预览。它不联网，但 Changes 面板的 Commit 是真的会提交。

介意的话只留 Files 面板的复制功能，把 `along-changes` 从安装目录里删掉就行。

# 最后

如果你也在用 Herdr，也受够了手动拼路径，可以试试。

- Mod 列表：<https://herdr.dev/mods/>（搜 along）
- 源码：<https://github.com/chenweil/herdr-along>

有想加的功能，Issue 或者 PR 都欢迎。这次没进核心，但走 Mod 的路子反而更自由。
