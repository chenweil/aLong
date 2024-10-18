---
title: "LLM_fine Tuning"
date: 2024-08-15T13:39:15+08:00
tags: [LLM,AI]
type: "post"
---

# 微调 fine-tuning
微调是一种监督学习过程,在这个过程中可以使用一组带标签的示例数据来更新LLM的权重.使其为特定任务生成良好完成的能力.

指令微调特别擅长提高模型在各种任务中的性能.
![image.png](https://s2.loli.net/2024/08/15/zdjLFeb17cpSwvO.png)

比如想让模型翻译能力增强,那就给他一些示例是包括翻译这句话之类的说明.即时完成示例允许模型学习生成遵循给定说明的响应.
## 微调大致步骤
1. 准备训练数据,需要特定的格式. 也可以通过数据集+模版来处理使其既是模版又是数据集(指令数据集).
2. 将数据集划分为训练验证和测试.然后使用计算出的损失来更新标准反向传播中的模型权重(standard backpropagation)。多批次重复操作.
![image.png](https://s2.loli.net/2024/08/15/NnuGVBHyRJAmeSQ.png)
3. 更新完进行最终的性能评估.通过测试得出精度.
![image.png](https://s2.loli.net/2024/08/15/7V1TbfgcANhPyk5.png)
4. 最终得到一个微调模型(Instruct LLM).
![image.png](https://s2.loli.net/2024/08/15/4dINakbHTDw2WRJ.png)