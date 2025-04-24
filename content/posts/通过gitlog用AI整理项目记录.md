---
date: 2025-02-19
# description: ""
# image: ""
lastmod: 2025-02-19
showTableOfContents: false
# tags: ["",]
title: "通过gitlog用AI整理项目记录"
type: "post"
---

# git log 列出时间段内容
导出2024全年内容到文本： `git log --since="2024-01-01" --until="2024-12-30" --pretty=format:"%h - %an, %ad : %s" --date=iso > commits_2024.txt`

# 讲这个文件加入单独的知识库中
此处使用[cherry studio]()进行知识库添加操作。

![](https://s2.loli.net/2025/02/19/t9unavDreIF3q8h.png)

但文件直接加入，模型我选的 `baai/bge-m3`。


# 使用模型对导出的内容进行处理
`
请根据提供的gitcommitlog进行总结。 总结的内容是2024年1月1日至2024年12月31日一共开发多少新功能，主要功能名称是什么，修复或优化多少功能点其中调整优化次数多的top3。
`


总结内容如下：
![](https://s2.loli.net/2025/02/19/G586pqHcAFCornu.png)

通过总结，我们可以将此段内容加入总结中或者用来PPT内容的中。