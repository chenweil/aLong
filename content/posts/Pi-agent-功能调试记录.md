---
title: "Pi agent 功能调试记录"
date: 2026-08-05
lastmod: 2026-08-05
draft: false
tags: [pi, agent, cli, ai]
type: "post"
description: "记录 Pi agent 中删除保护、小鱼（zen）、状态栏和 hjob 任务管理等功能的调试过程与实现结果。"
source: "https://linux.do/t/topic/2707997"
author: "wren"
showTableOfContents: true
---

> 本文根据我与 Pi agent 的需求对话整理而成，保留原始需求与实现结果，不是 AI 代写的成文内容。

## 会话记录

- 日期：2026-08-05
- 会话文件：`~/.pi/agent/sessions/<project-session>/2026-08-05T05-17-18-427Z_019fd05a-c8db-7253-98ab-c90aa763278f.jsonl`
- 处理事项：删除保护、小鱼（zen）、status lines 调整，以及额外的 hjob 任务管理。

## 1. 删除保护

### 原始需求（8 句）

1. Pi 没有权限控制，希望增加一个控制删除动作的开关：开启时可以删除，关闭时在删除前提醒我是否允许操作。
2. 支持通过 command 进行开关控制。
3. 删除当前目录下的 `x.md`。
4. 删除功能需要在每个项目目录运行后增加 `.pi/operation_log` 文件，记录删除操作的时间（系统时区）、指令详情、是否允许（yes/no）以及结果（ok/fail）。
5. 输入 `/reload`（原记录中的未完成片段）。
6. 删除目录下的 `md` 文件和 `png` 文件。
7. 后来发现“删除保护：关（删除命令需确认）”的文案语义相反：删除保护开启时需要确认，关闭时直接删除，也不记录日志。
8. 将“删除命令需确认”改为“需确认”。

### 实现产物

实现文件：`~/.pi/agent/extensions/delete-guard.ts`

- 支持 `/delete-guard on|off|toggle|status`。
- 删除保护开启：删除前需要确认，并将时间、指令、是否允许和执行结果写入 `.pi/operation_log`。
- 删除保护关闭：直接放行，不提示，也不记录日志。

### 效果

删除保护开启后，执行删除操作会先触发确认提示：

![](https://i.imgur.com/PbftBAd.png)

允许或拒绝后的日志记录：

![](https://i.imgur.com/19tERNX.png)

## 2. 小鱼（zen 扩展）

### 原始需求（3 句）

1. 建议将 Pi command `zen` 的小鱼气泡放在鱼的上一行；与鱼保持当前的水平距离，但不要和鱼处在同一行。
2. 看到效果后，希望气泡与鱼再近一点；鱼转向时，也要注意气泡的位置。
3. 补充测试命令：`/zen`（原始记录中写作 `/zem`）。

### 实现产物

实现文件：`~/.pi/agent/extensions/zen/lib/working-ship.ts`

- 气泡从鱼所在行移到鱼的上一行，水平位置保持不变。
- `FISH_BUBBLE_GAP` 从 2 改为 1，让气泡更靠近小鱼。
- 鱼转向时，气泡始终跟随鱼头切换到对应一侧，已逐帧验证。

### 效果

![](https://img.51ai.vip/2026-08-06-03.25-ykw9R4m0.gif)

一只小鱼在水里游，等待结果；滚动内容已屏蔽。

## 3. status lines 调整

### 原始需求（6 句）

1. Pi agent 的 status line 可以展示几行？
2. 希望每行展示三列，布局设想如下：
   - 第一行：第一列是项目名与 `git:branch±`，第二列是 `[###…] XX%`（上下文）与 context length，第三列是接入配置名、provider/model（thinking）和 goal 状态。
   - 第二行：第一列是 session ID，第二、第三列暂时作为占位。
   - 第三行：显示时间和电量。
3. `session:019fd05a` 这个 ID 有点短。
4. 但完整信息可能太长，因此去掉第二列和第三列，只保留一列。
5. 实际效果示例：`~/playground/pi [#…] 10% • 1M deepseek • deepseek-v4-flash (high) / session:019fd05a-c8db-7253-98ab-c90aa763278f / 13:50:07 100%`。
6. 列间距过长，需要缩短；时间和电量合并为一列，放在第三行。

### 实现产物

实现文件：`~/.pi/agent/extensions/minimal-footer.ts`

- 第 1 行使用三列：项目与 git、上下文条、接入配置与模型及 goal 状态。
- 第 2 行只显示完整 session ID。
- 第 3 行显示时间与电量。
- 列间距自适应为 2 个空格；宽度不足时按比例截断内容。

### 效果

![](https://i.imgur.com/twMJVWz.png)

## 4. 额外：hjob 任务管理

### 原始需求（3 句）

1. Hermes 使用 `~/Library/Mobile Documents/com~apple~CloudDocs/Job.md` 管理任务，目前有手动粘贴任务和处理后标记 ✅ 两种流程。希望增加 Pi 指令：`/hjob` 默认执行 add，接收链接或文件路径并写入对应位置；`/hjob list` 展示任务序号、内容（例如翻译/下载）和结果（成功/待办）三列。
2. 任务较多时会显示 `共 35 条任务 … (widget truncated)`，希望能够展开或查看其他内容。
3. 支持分页查看。

### 实现产物

实现文件：`~/.pi/agent/extensions/hjob.ts`

- `/hjob <链接|路径>`：链接按 Download 任务处理，文件路径按 Translation 任务处理。
- `/hjob list [页码]`：显示三列表格（序号、类型与内容、结果），每页 10 条，支持翻页。
- 新任务显示为待办，不带 ✅；Hermes 处理完成后自动加上 ✅。

### 效果

![](https://i.imgur.com/wLwqBU9.png)

![](https://i.imgur.com/ZlzdXRx.png)

支持分页，也可以直接使用 `/hjob url/path` 为 Hermes 增加任务。

以上是本次调试记录。🍀
