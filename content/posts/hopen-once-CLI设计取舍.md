***

title: "hopen-once.sh 的 CLI 设计取舍"\
date: 2026-09-02\
lastmod: 2026-09-02\
draft: false\
tags: \[shell, cli, bash, herdr, design]\
type: "post"\
description: "从 hopen-once.sh 的一个 pane 命名参数出发，看 shell CLI 设计中「语法等价」「容错」和「生态兼容」三个取舍。"\
source: "<https://github.com/chenweil/herdr-recipes>"\
author: "wren"\
showTableOfContents: true
-------------------------

git库: [herdr-recipes](https://github.com/chenweil/herdr-recipes)

写 shell 工具时，我越来越倾向于让脚本接受尽量多的写法。

用户不会背你的 flag 设计，只会按自己最舒服的方式打命令。脚本越包容， adoption 越高。

这篇文章以 `hopen-once.sh` 的 pane 命名参数 `-N` 为例，看一个参数怎么同时支持三种语法，以及背后做了哪些取舍。

## 1. 问题：给 pane 起名字

`hopen-once.sh` 在 herdr 里打开一组 pane 布局，并在每个 pane 里启动 agent。4 个 pane 的 2×2 布局，你通常会给每个 pane 起个名字，方便后续切换时认得出：

```
┌──────────┬──────────┐
│ 实现A     │ 实现B    │
├──────────┼──────────┤
│ review   │ 兜底     │
└──────────┴──────────┘
```

起名字本身不难。但问题是：不同用户的命令行习惯不一样。

有人习惯重复 flag：

```bash
-N 实现A -N 实现B -N review -N 兜底
```

有人喜欢逗号分隔：

```bash
-N "实现A,实现B,review,兜底"
```

还有人觉得名字和 agent 写在一起更自然：

```bash
codex:实现A codex:实现B pi:review claude:兜底
```

三种写法，同一个意思。我要让它们都能工作。

## 2. 三种语法并存的理由

很多 shell 工具会选一种"标准写法"，文档里写"请使用 `-N A -N B -N C`"。

这种设计没错，但隐含了一个假设：用户会认真读文档。

实际情况：

* 用户 `--help` 一下，看到有 `-N`

* 用户凭直觉试 `-N "A,B,C"`

* 用户直接把名字当位置参数塞进去

脚本只认一种写法，后两种用户都会得到一个"参数错误"，然后离开。

三种语法并存的本质是：把"解析输入"的复杂度揽到脚本这边，让用户那边保持简单。

## 3. 短 flag 和长 flag 混用

`hopen-once.sh` 同时支持短 flag 和长 flag，而且允许混写：

```bash
# 纯短 flag
./scripts/hopen-once.sh -n -l 22 -k codex:4

# 纯长 flag
./scripts/hopen-once.sh --no-agents --layout 22 --kind codex:4

# 混写
./scripts/hopen-once.sh -n --layout 22 -k codex:4 --pane-name a,b,c,d
```

短 flag 适合老手快速输入，长 flag 适合脚本调用或新手可读性。混写照顾的是"记不住完整单词、但记得住简写"的用户。

我用一个统一的 flag 解析器把短 flag 和长 flag 映射到同一个内部变量。解析完之后，脚本不关心你是用 `-l` 还是 `--layout`，只关心 `layout` 这个变量已经有了值。

## 4. 重复 flag：按出现顺序累积

这是最标准的 CLI 写法：

```bash
./scripts/hopen-once.sh -l 22 \
  -k codex -N "实现A" \
  -k codex -N "实现B" \
  -k pi   -N "review" \
  -k claude -N "兜底"
```

`-N` 每出现一次，就把这个名字追加到一个数组里。最终顺序就是出现顺序，对应 pane 的视觉阅读顺序（从左到右、从上到下）。

这个写法的好处是明确：每个名字和对应的 `-k` 在同一个上下文中，读的时候一目了然。

## 5. 逗号分隔：降低打字负担

4 个 pane 都要起名的话，重复 `-N` 四次有点啰嗦。所以 `-N` 也接受逗号分隔：

```bash
./scripts/hopen-once.sh -l 22 -k codex:4 -N "a,b,c,d"
```

内部实现：

```bash
IFS=',' read -ra NAMES <<< "$pane_name_arg"
for name in "${NAMES[@]}"; do
  name=$(echo "$name" | xargs)  # trim
  pane_names+=("$name")
done
```

`xargs` 做 trim 是为了容忍 `-N "a, b, c, d"` 这种带空格的写法。

这个设计的思路是：同一个参数，用户可以按自己舒服的粒度传递——一个值、一组值、或者一个列表。

## 6. 位置参数溢出：最省事的写法

`hopen-once.sh` 的位置参数本来是传 agent kind 的：

```bash
./scripts/hopen-once.sh codex codex pi claude
```

但用户有时候会把 pane 名字也混在位置参数里：

```bash
./scripts/hopen-once.sh -l 21 pi pi codex pi-top pi-bot cd-right
```

这个例子里，`-l 21` 只需要 3 个 pane，但用户给了 6 个位置参数。多出来的 3 个是什么？意图很明显：当 pane 名字用。

脚本的逻辑是：先用前 N 个位置参数填满 pane（N = layout 的 pane 数），多余的自动成为 pane 名字。

这是整个工具里最省事的语法，但它解决了一个真实场景：用户在 shell 里随手打命令，不想记 `-k` 和 `-N` 的边界。

## 7. `K:N` 批量重复：少打字

4 个 pane 全是同一个 agent 时，重复打 4 次 `-k codex` 很蠢。所以支持 `K:N` 语法：

```bash
./scripts/hopen-once.sh -l 22 -k codex:4
```

解析逻辑：

```bash
if [[ "$kind" == *:* ]]; then
  name="${kind%%:*}"
  count="${kind##*:}"
  for ((i=0; i<count; i++)); do
    kinds+=("$name")
  done
else
  kinds+=("$kind")
fi
```

这本质上是一个展开语法：`codex:4` 被展开成 4 个 `codex`。它不改变 `kinds` 数组的语义，只是在入队之前做了一次乘法。

## 8. 别名 + 透传：不锁死生态

`hopen-once.sh` 预设了一组 kind 别名：

| 别名   | 真实 agent |
| ---- | -------- |
| `op` | opencode |
| `cc` | claude   |
| `cd` | codex    |
| `pi` | pi       |

但关键在于：**未列出的 kind 原样透传**。

用户装了 `aider`，直接写 `-k aider` 就能用，不需要我来维护一个"所有 agent 的列表"：

```bash
./scripts/hopen-once.sh -l 22 -k aider -k aider -k pi -k claude
```

`aider` 这个名字直接传给 herdr，herdr 去决定怎么启动它。

这个决定避免了两个长期问题：一是生态锁定，每出一个新的 AI 编码工具都要改脚本；二是版本滞后，用户装的工具已经改名了，脚本的别名表还是旧的。

## 9. 自动 layout 推断：让默认值替你思考

用户不传 `-l` 时，脚本根据 kind 的数量自动选 layout：

```bash
# 3 个 kind → 自动推断为 12
./scripts/hopen-once.sh codex codex pi

# 4 个 kind → 自动推断为 22
./scripts/hopen-once.sh codex codex pi claude
```

内部是一个查表：1 个用 `11`，2 个用 `21`，3 个用 `12`，4 个用 `22`，5 个用 `13`，6 个用 `31`，7 个以上用 `111`。

目的是让最常见的用法变成默认行为。用户想开 3 个 pane 的时候，90% 的情况就是想用 `12` 布局，不需要再敲 `-l 12`。

## 10. 几个实际的设计取舍

从 `-N` 这一个参数出发，我有几点具体感受：

**语法等价比标准写法重要**。同一个意思接受多种写法。短 flag 和长 flag 等价，重复 flag 和逗号分隔等价，位置参数和显式 flag 在语义重叠时也能互通。用户的习惯不一样，脚本应该去适配，不是反过来。

**容错比简洁重要**。用户写 `-N "a, b, c, d"` 带空格，脚本自动 trim；用户多传几个位置参数，脚本判断意图而不是直接报错。CLI 是给人用的，不是给 parser 用的。

**透传比枚举重要**。脚本不应该维护一份"所有可能的值"的列表。遇到不认识的东西，最安全的做法是原样传下去，让下游决定怎么处理。这既减少了维护负担，也避免了生态锁定。

这三个点很小，但用到 `hopen-once.sh` 上之后，我发现自己几乎不再需要翻文档了。脚本接住了我随手打的各种写法。

如果你也在写 shell 工具，parser 设计上多花一点心思是值得的。用户不会记住你的 API，但会记住"这个工具用起来很顺手"这件事。
