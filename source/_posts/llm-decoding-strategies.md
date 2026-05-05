---
title: LLM 解码策略：temperature、采样与 beam search，到底在选什么
published: true
categories: AI
tags:
  - AI
  - 面试
  - 解码策略
abbrlink: 9461694414
date: 2026-05-01 12:00:00
---

# LLM 解码策略：temperature、采样与 beam search，到底在选什么

**一句话：所有解码策略本质上只做一件事——从模型输出的概率分布里挑一个 token。区别只在于你用什么规则去挑，以及你要确定性还是要多样性。**

---

## 基础管道：Logits → Softmax → 概率分布

模型最后一层输出的是 **logits**——词汇表里每个 token 对应一个实数，有正有负，范围不受约束。这还不是概率。

把它变成概率靠的是 softmax：

```
P(i) = exp(logit_i) / Σ exp(logit_j)
```

做完这一步，你手里就有了一个合法概率分布：每个 token 一个概率，全加起来等于 1。后续的 temperature、top-K、top-P、greedy decoding、beam search——全是在这个分布上做文章。要么改分布的形状，要么换挑选的规则。

---

## Temperature：一个旋钮控制"脑洞程度"

Temperature 是调起来最简单的参数。它在 softmax 之前动刀，把每个 logit 除以 T：

```
P(i) = exp(logit_i / T) / Σ exp(logit_j / T)
```

效果非常直观：

- **T → 0**：分布变得极尖。概率全部塌缩到得分最高的那个 token 上，等价于 greedy decoding。每次选最稳的，输出像教科书。
- **T = 1**：不改变原始分布，模型本身觉得该什么样就什么样。
- **T > 1**：分布被拉平。高分 token 被压下去，低分 token 被抬上来，模型开始"乱想"——创意上去了，废话也上去了。

关键理解：temperature 不改候选集，只改概率的集中程度。低 T = 安全可预测，高 T = 多样但危险。

---

## Top-K：一刀切，简单粗暴

Top-K 的思路更直接：softmax 之后，只保留概率最高的 K 个 token，其余全扔掉。然后在这 K 个里重新归一化、采样。

好处是你永远不会采到一个概率 0.001% 的离谱 token 毁掉整段输出。但毛病也很明显：K 是个死数。

有时候前 50 个 token 已经覆盖了 99% 的概率质量，K=50 太松了；有时候前 50 个才覆盖 60%，K=50 又切掉了太多合理选项。分布是动态的，K 是静态的，这就是矛盾。

---

## Top-P（Nucleus Sampling）：动态门槛，自适应裁剪

Top-P 解决了 top-K 的僵化问题。它不固定候选个数，而是设一个累积概率阈值 P（通常 0.9）。把 token 按概率从高到低排队，一路加进候选集，直到累积概率 ≥ P，然后在这个"核心集"里采样。

分布尖锐集中时（比如"The sky is..."，后面基本就那几个词），候选集自然很小；分布平坦、不确定性高时，候选集自动扩大。这种自适应能力让 top-P 在实际使用中普遍优于 top-K。这也是为什么 OpenAI 等提供方公开了 `top_p`，却没公开 `top_k`。

---

## 三者怎么配合

Temperature、top-K、top-P 可以叠加，但执行顺序很重要：**先用 top-K / top-P 过滤候选集，再用 temperature 调分布，最后采样**。

但实际中 top-K 和 top-P 一般是二选一，不是一起上——同时用会过度约束输出，反倒显得死板。

**实战参数速查：**

| 场景 | Temperature | Top-P | 原因 |
|---|---|---|---|
| 创意写作 | 0.8–0.9 | 0.95 | 要的就是随机和多样性 |
| 代码生成 | 0.1–0.3 | 0.9 | 精确第一，创意是负担 |
| RAG / 事实问答 | 0.0–0.2 | 0.9 | 死磕证据，别自由发挥 |

---

## Beam Search：当你要的不是"有意思"而是"最好"

Beam search 跟采样是两种哲学。它不随机抽，而是系统地探索多条生成路径，挑出全局得分最高的那一条。

### 怎么干活的

Beam search 每一步维护 B 条候选序列（B 叫 beam size）。每个位置，它对 B 条候选各自做一次前向传播，算出 B × vocab_size 个可能的下一 token，然后按累积 log probability 对所有扩展排名，只留前 B 名。就这么一步步推进，直到每条路径都生成结束标记或达到最大长度。

Beam size = 1 时，退化为 greedy decoding——一条路走到底，永远选局部最优。Beam size 放大（通常 3–8），算法就能从"局部看着挺好但会把整句带进沟里"的早期错误中恢复。

一个实现细节：beam search 不需要对每个 token 单独跑 B 次前向传播。B 条路径可以批量处理，计算成本约是 greedy 的 B 倍，而不是 B × vocab_size 倍。

### Length Normalization：防止短序列刷分

Beam search 天然偏爱短序列。因为 log probability 全是负数，序列越长，累积的负数就越多，纯粹因为长度吃亏。标准修正是 length normalization：

```
score = Σ log P(t_i) / length^α    (α 一般取 0.6–0.7)
```

没有这一手，模型为了刷分会产生不自然的极短输出。

### 代价

Beam search 产出的文本语法连贯、逻辑自洽——对机器翻译、语音识别、文本摘要这类"存在明确正确答案"的任务来说，这是你想要的。但对开放式生成，问题很大：

1. **没多样性**：同一 prompt 跑 10 次，10 次结果几乎一模一样。算法会系统性地收敛到最高概率路径。

   这背后有一个反直觉的机制。很多人以为 beam size=2 意味着两条路各司其职——一条永远跟 top-1，一条永远跟 top-2。实际上完全不是这样。

   每一步的真实操作是：每条 beam 跑一次前向传播，得到各自的概率分布，然后从所有 beam × 词表的候选里做**全局 top-B 筛选**。这意味着——同一条父序列的两个延伸同时进入 top-B 是家常便饭，而另一条父序列可能直接被淘汰，谱系断掉。

   举个例子。Step N 时手里有两条 beam：`"the cat"`（累积概率 0.20）和 `"a dog"`（0.12）。下一步：

   | 来自 | 下一个 token | 路径总概率 |
   |------|-------------|-----------|
   | `"the cat"` | `"sat"` (概率 0.4) | 0.20 × 0.4 = **0.080** |
   | `"the cat"` | `"slept"` (概率 0.3) | 0.20 × 0.3 = **0.060** |
   | `"a dog"` | `"ran"` (概率 0.3) | 0.12 × 0.3 = 0.036 |
   | `"a dog"` | `"barked"` (概率 0.2) | 0.12 × 0.2 = 0.024 |

   新的两条 beam 全部来自 `"the cat"`，`"a dog"` 这一脉直接断掉。这种"赢家通吃"的行为就是多样性塌缩的根源——看起来 beam size=5，实际上五个 beam 可能全是同一句话的微调版本，只在最后几个 token 上略有差异。

2. **安全但平庸**：最高概率的序列往往是最套路的那个。Beam search 尤其爱输出"There are several reasons why..."这种开头，因为训练数据里这种句式统计上就是最多。

3. **重复陷阱**：Beam search 可能陷入循环，反复输出同一个短语，因为模型对"继续一个已经重复的模式"赋予了高概率。

### 针对多样性的改进

意识到多样性塌缩问题后，学界提出了一系列改进方案：

- **Diverse Beam Search**（2018）：在评分时对来自同一父序列的"姐妹 beam"施加惩罚，强制 beam 在不同父序列之间分流。
- **Stochastic Beam Search**：用 Gumbel-top-k 技巧将随机采样引入 beam search，使选择不再完全确定，保持探索性。
- **Group Beam Search**：把 beam 分成若干独立小组，组内做 beam search 但组间不共享市场，从结构上保证多样性。
- **Nucleus Sampling（top-P）+ temperature**：这是业界最终的务实选择——既然 beam search 的多样性问题这么难根治，不如直接在对话场景用采样方案。

---

## 为什么聊天模型拥抱采样

ChatGPT、Claude 这些对话模型，几乎都在用 temperature + top-P 采样，不用 beam search。根本原因在于聊天和翻译是两种性质完全不同的任务。

**多样性是刚需，不是副作用。** 用户说"讲个故事"，不能每次都讲一模一样的故事。采样引入可控随机，对话才有"活的"感觉。

**没有唯一正确答案。** 翻译有参考译文对照，beam search 追最大似然序列的逻辑成立。但在开放式聊天里，"概率最高"不等于"最好"。对"How are you?"概率最高的答法可能是"I'm fine, thanks"——但这是最无聊的回复。

**长度和重复惩罚。** 基于采样的方法可以配套 repetition penalty、presence penalty 等抑制已出现 token 的技术。这些手段很难干净地嵌入 beam search 的评分框架。

---

**核心要点**

- 一切从 **logits → softmax → 概率分布** 开始。每种解码策略都是从这个分布里挑选 token 的不同方式。
- **Temperature**：在 softmax 前调分布形状。T→0 变 greedy，T=1 不变，T>1 拉平增加随机性。
- **Top-K**：硬留概率最高的 K 个，简单但僵化。
- **Top-P（Nucleus Sampling）**：动态凑到累积概率 ≥ P，自适应能力强，通常优于 top-K。
- 执行顺序：先 top-K/top-P 过滤 → 再 temperature 调分布 → 最后采样。
- **Beam Search**：维护 B 条并行路径，做全局最优选择。Beam size=1 退化为 greedy。全局择优机制天然导致多样性塌缩——多条 beam 常收敛到同一父序列，其余谱系被淘汰。
- Beam search 适合翻译/摘要（"有正确答案"的任务），但输出僵硬、重复、缺多样性——不适合开放式对话。
- 现代聊天模型用 temperature + top-P 采样，因为对话要多样性、没有唯一正确答案，且兼容重复惩罚。
- Length normalization（`score / length^α`）防止 beam search 不公平地偏爱短序列。

## Reference

本文源于以下面试题：

- [04-30 temperature、top_k、top_p 分别控制什么？背后数学逻辑是什么？怎么组合使用？](../daily/2026-04-30.md)
- [05-01 Beam Search 的原理是什么？和 greedy decoding 有什么区别？为什么现代对话模型不用它？](../daily/2026-05-01.md)
