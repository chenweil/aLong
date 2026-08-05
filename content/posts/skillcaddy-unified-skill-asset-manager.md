---
title: "Skillcaddy：统一管理 AI Agent Skill 的轻量方案"
date: 2026-06-30
lastmod: 2026-07-17
draft: true
type: "post"
description: "在多 Agent 协作场景下，全局装 skill 会膨胀上下文、项目间拷贝又难同步版本。Skillcaddy 用「中央仓库 + 按需软链」的思路，把 skill 抽离到项目级管理，同时保持单一来源。"
categories: []
tags: ["ai", "agent", "skill", "claude-code"]
showTableOfContents: true
image: "/images/skillcaddy/central-library.jpg"
---

## 是什么 / 定位

如果你同时使用 Claude Code、Codex、OpenCode、Pi 等多个 AI Agent，肯定会遇到一个问题：

**skill 到底装在哪里？**

- 装全局（`~/.claude/skills/`、`~/.agents/skills/`） → context 膨胀，多个项目互相污染
- 装项目里拷贝 → 改了一个地方，其他项目还要手动同步

Skillcaddy 的思路很简单：既然 `symlink` 就能解决，那就做一个管理 symlink 的小工具。

**一个中央 skill 仓库，按需软链到各项目的 `.claude/skills/` 或 `.agents/skills/` 目录。**

它不做语义分析、不做云端同步 —— 纯粹是本地文件管理的自动化，适合追求极简工具链的开发者。

{{< figure src="/images/skillcaddy/14f1a1086ce027655ff6f2754cb49004_MD5.jpg" alt="Skillcaddy 核心理念：中央仓库 + 按需软链" caption="一个 AISkills 仓库，多个 Agent 共享" >}}

## 核心功能

### 集中式的 skill 仓库位于 `~/skillcaddy`

所有 skill 源码放这里，统一管理，包括 git 版本追踪。

{{< figure src="/images/skillcaddy/1b442b33cc501f2643d77cde7702092d_MD5.jpg" alt="Skillcaddy init 创建的 AISkills 目录结构" caption="~/AISkills 作为所有 skill 的根目录" >}}

### 一个命令，按需 link 到当前项目

`sc-link`（或 `skillcaddy link`）会把当前项目需要的 skill 从中央仓库软链到本地 `.claude/skills/`。
不想用的时候 `sc-unlink` 清除。

{{< figure src="/images/skillcaddy/d4286847d14bfdce162db580084c3f78_MD5.jpg" alt="sc-link 命令效果演示" caption="一行命令完成 skill 安装" >}}

### 跨 Agent 共享

无论 `.claude/skills`、`.agents/skills` 还是其他 convention，都能同时指向同一份源码。

{{< figure src="/images/skillcaddy/afd775683d1330ce205ca440479ce437_MD5.jpg" alt="Claude Code 与 OpenCode 同时使用同一 skill" caption="Claude Code 和 OpenCode 共享 soft-link" >}}

{{< figure src="/images/skillcaddy/d1e0b12b76e74041d4b3a61b1741291c_MD5.jpg" alt="Codex 也能读取同一个 skill" caption="Codex 通过同一个软链访问 skill" >}}

## 实际体验

用 Skillcaddy 开发过几个项目，最明显的体感差异：

- 启动 Agent 时 context 占用减少（只链接当前需要的 skill）
- skill 升级在 `~/AISkills` 下 git 搞定，不需要在多个项目间穿梭
- 与**设计类 skill**（例如 [Design-MD](https://getdesign.md/claude/design-md)）配合良好：sc-link 后自动重构页面

{{< figure src="/images/skillcaddy/4199c750b229dc5b1b78494443c2c0d8_MD5.jpg" alt="用 Skillcaddy 管理设计类 skill 后的页面效果" caption="Skillcaddy + Design-MD skill 产出页面" >}}

再分享两个实战技巧：

### 1. 截图标注式沟通

调试 UI 时，Codex/Claude Code 识图能力强。直接截图+标注让它修改，比文字描述效率高 10 倍。

{{< figure src="/images/skillcaddy/451862a8c577c1d5a42998ea3e6515bd_MD5.jpg" alt="截图标注调试" caption="截图 + 标注 → Agent 直接出修改" >}}

### 2. 模型搭配省额度

先用贵的模型（codex）做 MVP 探索快 5h，额度耗尽后切便宜的 MiniMax M3 做后续维护（修页面、查报错、写文档、git commit）。

灵活搭配：
| 阶段 | 模型 | 耗时 | 花费 |
|:---|:---|:---|:---|
| MVP + 架构 | codex | ~5h | 高 |
| 维护 + 文档 | MiniMax M3 | - | 几块钱 |

{{< figure src="/images/skillcaddy/4a198f980a164cacdd811f1a10284c08_MD5.png" alt="MiniMax M3 接手维护工作" caption="MiniMax M3 负责后续维护 → 低成本收尾" >}}

## 踩坑提示

- ⚠️ **Windows 下 symlink 默认需要管理员权限**：要么改组策略，要么用 `mklink /J` 做 junction（Skillcaddy 暂未处理）
- ⚠️ **skill git 仓库不要随意 force push**：多个项目依赖它，改了历史链会断
- ⚠️ **不是所有全局 skill 都应移出**：高频系统级 skill（如 shell 执行、git 操作）留在全局反而省事
- ⚠️ **只读项目慎用 symlink**：CI 里跑不一定解析链接，建议 fallback 为 copy

## 优缺点

**✅ 喜欢**
- 极简（就几个 shell 命令，本质是 `ln -s` 的 wrapper）
- 符合 Unix 哲学：One thing done well
- 不引入新的格式 / 生态 / 语言依赖

**❌ 不支持**
- 没有云端同步（对比 SkillHub、Claude Marketplace 这类中心化市场）
- 没有版本锁定（不像 `npm install foo@1.2.3` 那样精确锁版本）
- IDE 集成为零（全靠手动命令行）

## 适用人群

- 多 Agent 用户（Claude Code + Codex + OpenCode + Pi 混搭开发者）
- 追求极简、愿意命令行操作
- 不想被 skill marketplace 绑定
- 想把 skill 一样当 git repo 管理

不适合：只装一个 Agent 的用户 / 喜欢一站式云端管理的人。

## 小结

Skillcaddy 没发明新概念，只是把 symlink 这个老手艺包装得足够低摩擦。
如果你已经在多 Agent 之间疲于管理 skill 的位置，它会让你清爽不少。

GitHub: <https://github.com/chenweil/skillcaddy> · 附 [中文 README](https://github.com/chenweil/skillcaddy/blob/main/README_CN.md)

{{< figure src="/images/skillcaddy/dbbb690a7549f91c662cf631844da831_MD5.png" alt="GitHub README 截图" caption="项目 README 详细文档" >}}

完结  
祝好  
🍀
