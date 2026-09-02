---
title: "hopen-once.sh 的 CLI 设计取舍"
date: 2026-09-02
lastmod: 2026-09-02
draft: false
tags: [shell, cli, bash, herdr, design]
type: "post"
description: "从 hopen-once.sh 的一个 pane 命名参数出发，看 shell CLI 设计中「语法等价」「容错」和「生态兼容」三个取舍。"
source: "https://github.com/chenweil/herdr-recipes"
author: "wren"
showTableOfContents: true
---

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

实现是 `scripts/hopen-once.sh:149` 的 `_expand_name`：

```bash
# 把 "-N a,b,c" 里的逗号分隔值展开成多条 NAMES。跟 -N 重复 flag 等价：
#   -N "a,b,c"          = -N a -N b -N c
#   -N "a, b, c"        = -N "a" -N " b" -N " c"  （trim 头尾空白）
# 不含逗号就走原样追加，跟 _expand_kind 对称。
_expand_name() {
  local raw="$1"
  case "$raw" in
    *,*)
      local part
      # IFS=',' read -ra 是 bash 标准分割；分隔后逐个 trim 头尾空白。
      local IFS=','
      local -a _parts
      _parts=($raw)
      for part in "${_parts[@]}"; do
        # trim leading + trailing whitespace
        part="${part#"${part%%[![:space:]]*}"}"
        part="${part%"${part##*[![:space:]]}"}"
        NAMES+=("$part")
      done
      ;;
    *)
      NAMES+=("$raw")
      ;;
  esac
}
```

里面几个决定都有原因，逐个说。

**先 `case *,*` 判断，再分割。**不含逗号就原样追加，`-N "兜底"` 不会被任何规则误伤。代价是：pane 名里真想带一个逗号是做不到的，我没留转义。这个取舍我认——给 `-N "a\,b"` 加一层转义，整个 parser 的复杂度要抬一档，而需要名字里带逗号的人大概不存在。

**trim 为什么不用 `xargs`。**最顺手的写法是 `name=$(echo "$name" | xargs)`，我试过，放弃：

- `xargs` 会重新解析引号。名字里有一个落单的 `"`，它就报 `xargs: unterminated quote` 并退出 1。脚本顶部是 `set -euo pipefail`，命令替换里的非零退出会把整个脚本带走。也就是说"我想给 pane 起名叫 `review"最终版"`"的结局不是名字难看，是脚本死在解析阶段。
- `xargs` 还会把连续空格压成一个，`review  agent` 变成 `review agent`。这是静默改用户输入，比报错更难查。

所以换成 `local IFS=','` 加两行 `${part#"${part%%[![:space:]]*}"}`：零子进程，除了首尾空白什么都不碰。

**为什么不用 `IFS=',' read -ra NAMES`。**这是 bash 里分割字符串的标准答案，但 `read -ra` 是整体覆盖，会把前面 `-N` 已经攒下的值抹掉。重复 flag 和逗号分隔必须能混着用——`-N 实现A -N "实现B,review"` 是 3 个名字。所以分割先落到局部数组 `_parts`，再逐个 append 到全局 `NAMES`。顺带说一句，`_expand_name` 里那行注释写的是"IFS=',' read -ra 是 bash 标准分割"，但函数里并没有 `read -ra`——注释描述的是分割思路，实现换成了词分割，原因上面这条。注释比代码旧，这是自己写文档最常欠的债。

**一个还开着的口子。**`_parts=($raw)` 是不带引号的展开，除了词分割还会做路径名展开。实测：

```
$ _expand_name "a,*,c"          # 测试目录里有 aaa.txt / bbb.txt / t.sh
[a] [aaa.txt] [bbb.txt] [t.sh] [c]
```

`set -f` 能堵掉，但它是全局开关，`hopen.sh` 是 source 进来的，为了一个 `*` 出现在 pane 名里而动整个脚本的词法不划算。真想修的话，换成 `IFS=',' read -ra _parts <<< "$raw"` 就行——`read` 不做 glob，而且 `_parts` 是局部数组，覆盖问题不存在。我暂时选择把这一条记下来而不是改掉。

## 6. 位置参数溢出：最省事的写法

`hopen-once.sh` 的位置参数本来是传 agent kind 的：

```bash
./scripts/hopen-once.sh codex codex pi claude
```

但用户有时候会把 pane 名字也混在位置参数里：

```bash
./scripts/hopen-once.sh -l 21 pi pi codex pi-top pi-bot cd-right
```

这个例子里，`-l 21` 只有 3 个 pane，位置参数给了 6 个。多出来的 3 个是什么？意图很明显：当 pane 名字用。

规则是：位置参数**全部**先过一遍 `_expand_kind` 进 `KINDS[]`，然后由 `_cap_kinds_to_panes`（`scripts/hopen-once.sh:214`）把超出 pane 数的部分切出来当名字。注意那一步是**前插**，不是追加：

```bash
_cap_kinds_to_panes() {
  local max_panes="$1"
  if [ "${#KINDS[@]}" -gt "$max_panes" ]; then
    local extra=("${KINDS[@]:$max_panes}")
    echo "hopen-once: kind 数 ${#KINDS[@]} 超过布局 $LAYOUT 的 pane 数 $max_panes" >&2
    echo "             多余部分当 NAMES: ${extra[*]}" >&2
    KINDS=("${KINDS[@]:0:$max_panes}")
    if [ "${#NAMES[@]}" -eq 0 ]; then
      NAMES=("${extra[@]}")
    else
      local old_names=("${NAMES[@]}")
      NAMES=("${extra[@]}" "${old_names[@]}")
    fi
  fi
}
```

而 `NAMES[]` 是按视觉顺序消费的（`NAMES[0]` 对应第一个 pane）。所以溢出的名字占的是**最前面**那几个视觉位，`-N` 传的排在后面——跟你在命令行里先敲哪个无关。实测：

```
$ hopen-once.sh -l 22 -N x -N y codex codex pi claude A B
hopen-once: kind 数 6 超过布局 22 的 pane 数 4
             多余部分当 NAMES: A B

  left-top     <- A
  right-top    <- B
  left-bottom  <- x
  right-bottom <- y
```

`-N` 写在最前面，名字却落在后两个 pane 上。这条我还没想好要不要改成追加：前插唯一的好处是「溢出优先」，代价就是混用时顺序反直觉。实用建议只有一条——**一条命令里只用一种命名方式**，要么全 `-N`，要么靠溢出。

混用还有个副作用：两边加起来超过 pane 数，被砍的是尾部，也就是 `-N` 那批。`-l 22 -N x -N y -N z codex codex pi claude A B` 会得到：

```
hopen-once: name 数 5 超过布局 22 的 pane 数 4
             多余 name 被忽略: z
```

报错里点名了被丢的是谁，但顺序这件事它不说，得回到这一节来看。

这是整个工具里最省事的语法，但它解决了一个真实场景：用户在 shell 里随手打命令，不想记 `-k` 和 `-N` 的边界。

## 7. `K:N` 批量重复：少打字

4 个 pane 全是同一个 agent 时，重复打 4 次 `-k codex` 很蠢。所以支持 `K:N` 语法：

```bash
./scripts/hopen-once.sh -l 22 -k codex:4
```

实现是 `scripts/hopen-once.sh:121` 的 `_expand_kind`：

```bash
# 把 "K" 或 "K:N" 展开成 1 或 N 条 KINDS。N 必须 1-9 开头的正整数。
# `-k codex:4` 等价于 `-k codex -k codex -k codex -k codex`，但只打一次。
_expand_kind() {
  local raw="$1"
  case "$raw" in
    *:*)
      local k="${raw%:*}" n="${raw#*:}"
      if [ -z "$k" ]; then
        echo "hopen-once: '$raw' 缺少 kind 部分" >&2; return 2
      fi
      if [[ "$n" =~ ^[1-9][0-9]*$ ]]; then
        local i=0
        while [ $i -lt "$n" ]; do
          KINDS+=("$k")
          i=$((i + 1))
        done
      else
        echo "hopen-once: '$raw' 的计数 '$n' 不是正整数" >&2; return 2
      fi
      ;;
    *)
      KINDS+=("$raw")
      ;;
  esac
}
```

展开本身没什么意思，一个乘法。值得写的是两处"为什么故意写窄"。

**为什么正则是 `^[1-9][0-9]*$`，不是 `^[0-9]+$`。**

- 挡掉 `codex:0`。展开 0 条不会报错，只会让 `KINDS` 少一截，然后走到布局推断报"kind 数量 2 不支持自动布局"——用户在错误信息里看到的数字，和自己输入的完全对不上，得回头一行行数。在语法层拒绝，错误就落在真正出错的那个 token 上。
- 挡掉前导零。`[ $i -lt "$n" ]` 里 `08` 是安全的，但这段逻辑哪天被重写成 `(( i < n ))`，或者顺手加个上限判断 `(( n > 4 ))`，bash 就把 `08` 当八进制，运行时炸一句 `value too great for base`。正则里先禁掉，比留地雷便宜。
- 顺带挡掉 `+2`、`0x4`、`1e3`、空 kind（`:3`）。全都 `return 2`，顶层 `|| exit 2`：

```
$ hopen-once.sh -l 22 -k codex:0
hopen-once: 'codex:0' 的计数 '0' 不是正整数
$ hopen-once.sh -l 22 -k codex:08
hopen-once: 'codex:08' 的计数 '08' 不是正整数
$ hopen-once.sh -l 22 -k codex:2:3
hopen-once: 'codex:2:3' 的计数 '2:3' 不是正整数
```

最后那条是我想要的效果：多冒号不猜意图。`${raw#*:}` 取到的是 `2:3`，被正则一挡就死了，不会出现"半个 token 被当成 kind 名"的静默降级。

**为什么不设上限。**`-k codex:12` 语法上合法，因为"不能超过 pane 数"这件事不归 `_expand_kind` 管，归 §6 里那个 `_cap_kinds_to_panes` 管：它把 `KINDS` 截到 `max_panes`，多出来的部分整段转成 `NAMES`。

一个函数同时兜住两个入口：显式 `-k` 传多了，和位置参数传多了。数量规则只在这一处，两个入口共用；`_expand_kind` 只管形状，不管语义。

代价你大概已经猜到了：`-l 22 -k codex:12` 里多出来的 8 个 `codex` 会被当成 pane 名字，四个 pane 全叫 codex。第一次遇到会觉得被整了，但这正是"多余即命名"贯彻到底的样子，我没给它加特例——加了以后，`-k` 和位置参数就得各自记一套规则，那才是真难用。

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
# 2 个 kind → 11（左右对分）
./scripts/hopen-once.sh codex pi

# 3 个 kind → 12（左大 + 右两格）
./scripts/hopen-once.sh codex codex pi

# 4 个 kind → 22（2×2）
./scripts/hopen-once.sh codex codex pi claude
```

查表只有三条：

```bash
case "${#KINDS[@]}" in
  2) LAYOUT=11 ;;
  3) LAYOUT=12 ;;
  4) LAYOUT=22 ;;
  *) # 报错 + exit 2 ;;
esac
```

不是懒得多写几行，是**多写也没有对应的布局可以选**。`hopen.sh` 里 `_panes_for` 把 layout 代号和 pane 数钉死了：

| layout | 11 | 12 | 21 | 111 | 13 | 31 | 22 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pane 数 | 2 | 3 | 3 | 3 | 4 | 4 | 4 |

支持的全部布局里，pane 数上限就是 4。5 个、6 个 kind 落不到任何布局上——`13` 和 `31` 都还是 4 格，`111` 也不是“1 行 1 列 1 行”之类的七格，它是三列横排。所以数量一超出 2/3/4，唯一诚实的答复就是报错：

```
hopen-once: 没有 --layout，且 kind 数量 6 不支持自动布局（2 → 11，3 → 12，4 → 22）
             传 --layout CODE 显式指定，或调整 kind 数为 2、3 或 4
             支持: 11 12 21 22 13 31 111
```

三行分别说“现在不行”、“怎样才能行”、“能选什么”。错误信息里把下一次该怎么敲直接写出来，这条我在 shell 工具里已经成习惯了：用户不会翻 `--help`，但报错那几行一定会看。

反过来，最常见的用法就让它默认掉——3 个 pane 十有八九就是要 `12`，不值得为此多敲 `-l 12`。

（`11` 是刚补的，之前只有 3、4 两个数量能推断，2 个 pane 的场景纯属漏了。）

## 10. 几个实际的设计取舍

从 `-N` 这一个参数出发，我有几点具体感受：

**语法等价比标准写法重要**。同一个意思接受多种写法。短 flag 和长 flag 等价，重复 flag 和逗号分隔等价，位置参数和显式 flag 在语义重叠时也能互通。用户的习惯不一样，脚本应该去适配，不是反过来。

**容错比简洁重要**。用户写 `-N "a, b, c, d"` 带空格，脚本自动 trim；用户多传几个位置参数，脚本判断意图而不是直接报错。CLI 是给人用的，不是给 parser 用的。

**透传比枚举重要**。脚本不应该维护一份"所有可能的值"的列表。遇到不认识的东西，最安全的做法是原样传下去，让下游决定怎么处理。这既减少了维护负担，也避免了生态锁定。

这三个点很小，但用到 `hopen-once.sh` 上之后，我发现自己几乎不再需要翻文档了。脚本接住了我随手打的各种写法。

如果你也在写 shell 工具，parser 设计上多花一点心思是值得的。用户不会记住你的 API，但会记住"这个工具用起来很顺手"这件事。
