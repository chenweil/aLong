---
title: "用 herdr pane layout 搭建并行编码工作流"
date: 2026-09-01
lastmod: 2026-09-01
draft: false
tags: [herdr, ai, agent, workflow, shell]
type: "post"
description: "如何用 herdr 的 pane layout 把多个 AI 编码 agent 放进同一台终端，让实现、调试、review 并行进行，而不是排队等待。"
source: "https://github.com/chenweil/herdr-recipes"
author: "wren"
showTableOfContents: true
---

git库: [herdr-recipes](https://github.com/chenweil/herdr-recipes)

> 前提一安装[herdr](https://herdr.dev/)

我现在的终端里通常会同时开着三四个 AI agent。左上角在写功能 A，右上角在写功能 B，下方有另一个 agent 专门做 review。我不需要等一个跑完再起下一个。

这篇文章记录的是我具体怎么搭的。

## 1. 单 agent 串行的瓶颈

最近同时在做三件事：实现用户列表分页、修登录态过期判断、review 同事刚提的 PR。三件事互不依赖，但用单个 agent 只能排队。

```
实现分页 → 等它跑完
修复登录态 → 等它跑完
review PR → 等它跑完
```

agent "思考"的时候终端空着，更麻烦的是上下文污染：跑完第 1 个任务的 agent 记住了大量实现细节，再让它做第 3 个 review 时，上下文已经偏了。

## 2. 把团队座位投射到终端上

herdr 是一个终端多路复用器，和 tmux / zellij 类似，但它对"开新 workspace、按比例 split pane、在指定 pane 里启动命令"这套操作有原生支持。

我的映射关系很简单：

- 一个 pane = 一个 agent
- 一个 layout = 一套座位表
- 一个 workspace = 一个项目上下文

把"团队怎么坐"配成模板，一键开出来。

## 3. 6 种布局代号

`hopen.sh` 用两位数字表示预定义布局，数字就是该侧 pane 的数量，herdr 自动按比例 split。

我用得最多的是 `22`，日常两个实现 + 一个 review + 一个兜底刚好填满。其余几个：

- **`12`**：左 1 / 右 2。一个主导带两个辅助。
- **`21`**：左 2 / 右 1。两个主导一个辅助。
- **`13`**：左 1 / 右 3。一个主导带三个辅助。
- **`31`**：左 3 / 右 1。三个主导一个辅助。
- **`111`**：三等分横排。三个 agent 并排，最简单的多 pane 布局。

## 4. 两种开布局的方式

项目里两种方式，对应两种使用习惯。

### 4.1 静态模板：`hopen.sh` + `hopen-agents.conf`

适合固定组合。在 `hopen-agents.conf` 里按 layout 和视觉位置写死每个 pane 的 agent 和 prompt：

```toml
# hopen-agents.conf
[22.top-left]
kind = "codex"
prompt = "实现用户列表的分页接口"

[22.top-right]
kind = "codex"
prompt = "修复登录态过期判断"

[22.bottom-left]
kind = "pi"
prompt = "review 上面的两个实现"

[22.bottom-right]
kind = "claude"
prompt = "备用：兜底调试"
```

一键打开：

```bash
./scripts/hopen.sh 22
```

布局开好，四个 agent 自动启动，每个 pane 里的 agent 都会收到初始 prompt。

### 4.2 命令行 ad-hoc：`hopen-once.sh`

适合临时搭台子。不用改配置文件，直接在命令行描述：

```bash
# 3 个 pane，layout 自动推断为 12
./scripts/hopen-once.sh codex codex pi

# 4 个 pane，2×2 布局
./scripts/hopen-once.sh -l 22 codex codex pi claude

# 给每个 pane 起名 + 发 prompt
./scripts/hopen-once.sh -l 22 \
  -k codex -N "实现A" -p "实现用户列表分页" \
  -k codex -N "实现B" -p "修复登录态过期" \
  -k pi   -N "review" -p "review 上面的两个实现" \
  -k claude -N "兜底"
```

用完关 workspace 就行，不留配置残留。

## 5. 一个真实场景：2×2 并行

![](https://img.51ai.vip/20260901151707110.png)

我日常用得最多的是 `22` 布局。典型流程：

```
┌─────────────┬─────────────┐
│  codex-A    │  codex-B    │
│  实现功能A  │  实现功能B     │
├─────────────┼─────────────┤
│  pi         │  claude     │
│  review     │  兜底/调试   │
└─────────────┴─────────────┘
```

1. 在当前项目目录执行 `hopen-once.sh -l 22 -k codex -k codex -k pi -k claude`
2. 四个 pane 同时启动，各自进入对应 agent 的 REPL
3. 我给 codex-A 发"实现用户列表分页"，给 codex-B 发"修复登录态"
4. pi 负责 review 两个实现，claude 空闲时跑测试
5. 全部完成后关掉整个 workspace

通常 5 分钟能收尾，串行做一般要翻倍还不止。

## 6. 命名 = 可读性

多 pane 并行最实际的问题是：切 pane 时不知道哪个里面在跑什么。

项目里做了两层：

**Pane 级别**：每个 pane 自动获得可读 label，比如 `left-top`、`right-bottom`。也可以自定义：

```bash
./scripts/hopen-once.sh -l 21 \
  -k pi -N "实现A" \
  -k codex -N "实现B" \
  -k pi   -N "review"
```

![](https://img.51ai.vip/20260901152118556.png)

herdr 的 pane list 直接显示这些名字，不用来回切换确认。

**Workspace 级别**：新 workspace 按当前目录自动取名。git 仓库用当前分支名，普通目录用 basename。撞名时加 `+a`、`+b` 后缀，26 个字母用完退化成时间戳。

## 7. 一键键位

光有脚本不够，得在"想开布局"的瞬间就能开出来。

我在 `~/.config/herdr/config.toml` 里绑了这些：

```toml
[[keys.command]]
key = "1"
modifiers = ["prefix"]
action = "run-command"
command = "scripts/herdr-pane-switch.sh 1"

[[keys.command]]
key = "1"
modifiers = ["prefix", "alt"]
action = "run-command"
command = "scripts/hopen.sh 12"
```

其余类推到 6。之后的操作就是：

1. 在当前项目目录
2. 按 `prefix + alt + 3`，对应 `22` 布局
3. 四个 pane 同时出现，agent 自动启动
4. 按 `prefix + 1..4` 在 pane 之间切换

全程不超过 2 秒。

## 8. 这套思路为什么能成立

这个项目的本质不是"管理 herdr 配置"，而是把 AI agent 的分工投射到终端布局上。

几个实际观察：

- 没有依赖关系的任务，让不同 agent 同时做比串行快得多。
- 用什么布局，取决于"团队"有几个人、谁是主导。`22` 适合两两并行，`13` 适合一个主 agent 带三个辅助。
- pane 和 workspace 的名字就是工牌，名字起得清楚，切换成本就低。
- 需要超过两个步骤才能开出来的工作流，坚持不了太久。

这套思路不只适用于 herdr。任何支持 pane split 和 workspace 的终端多路复用器都能套。脚本调用的 API 不同，但"多 agent 并行"这个想法本身是通用的。

如果你也在同时用多个 AI 编码工具，可以把"开布局 + 起 agent"沉淀成一个键位，终端会像 AI 团队办公室一样好用。

下一步再写写 `hopen-once.sh` 的设计.
