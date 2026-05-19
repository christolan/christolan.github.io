---
title: 看懂 Transformer Block：Attention、MLP、Residual 和 Norm 到底怎么配合
date: 2026-05-19 11:11:33
published: true
categories:
  - AI
tags:
  - AI
  - 面试
  - Transformer
cover: /images/understanding-transformer-block-residual-norm.svg
top_img: /images/understanding-transformer-block-residual-norm.svg
---

# 看懂 Transformer Block：Attention、MLP、Residual 和 Norm 到底怎么配合

**一句话：Transformer block 不是一团神秘公式，它每一层都在做四件朴素的事——attention 负责“看上下文”，MLP 负责“自己加工”，residual connection 负责“不丢原信息”，LayerNorm / RMSNorm 负责“不让数值失控”。**

![Transformer Block 核心结构图](/images/understanding-transformer-block-residual-norm.svg)

很多人在理解 Transformer 时，会先学到 self-attention、Q / K / V、position encoding、KV Cache、decoding 这些概念。它们相对容易建立直觉，因为都能和“模型如何读上下文、如何生成下一个 token”连起来。

但一旦进入 residual connection、LayerNorm、RMSNorm、Pre-Norm，难度会突然上升。原因不是这些概念本身多复杂，而是它们不直接回答“模型生成了什么”，而是在回答另一个更底层的问题：**这么深的网络，为什么能稳定训练下去？**

这篇文章就只讲一件事：一个 Transformer block 里面，各个组件到底怎么配合。

---

## 从 token 表示开始

LLM 不直接处理文字。输入文本会先被切成 token，每个 token 再被映射成一个向量。比如“苹果发布了新手机”进入模型后，不是以自然语言句子的形式存在，而是一组向量：

```text
“苹果”   → [0.12, -0.33, 0.88, ...]
“发布”   → [...]
“新手机” → [...]
```

这些向量就是模型内部理解文本的方式。最开始，每个 token 的表示还比较基础；经过一层层 Transformer block 后，每个 token 的表示会不断吸收上下文、形成更高层的语义。

所以可以先把 Transformer block 理解成一个“向量加工车间”：输入是一组 token 向量，输出还是一组 token 向量，只是每个向量都变得更懂上下文。

---

## Self-Attention：让 token 看上下文

Self-attention 的作用，是让每个 token 去看同一句话里的其他 token，然后决定自己应该吸收哪些信息。

比如这句话：

```text
苹果发布了新手机，它很受欢迎。
```

这里的“它”到底指什么？单看“它”这个 token 是判断不出来的。模型需要让“它”去关注前面的“苹果”“新手机”等 token，结合上下文才能得到更合理的表示。

所以 self-attention 做的事可以口语化成三步：

```text
我是谁？
我应该重点看哪些 token？
我从它们那里拿到什么信息？
```

这就是为什么 attention 经常被说成“信息路由”。它不是直接存储所有知识，而是在当前上下文中建立 token 之间的关系，让每个 token 的表示从“孤立的词”变成“带上下文的词”。

---

## MLP / FFN：让 token 自己加工

Self-attention 负责“看别人”，MLP / FFN 负责“自己消化”。

一个 token 通过 attention 拿到了上下文信息，但这些信息还需要进一步组合、抽象和变换。MLP / FFN 就负责这一步。它通常会把隐藏向量先扩展到更高维度，经过非线性激活函数，再压回原来的维度。

可以简单理解成：

```text
Self-Attention：信息交流层
MLP / FFN：语义加工层
```

比如模型看到“巴黎是法国的首都”，attention 可以让“巴黎”“法国”“首都”互相关联；MLP / FFN 则进一步把这些关系加工成更抽象的语义特征。模型内部当然不是用中文在想，但这个类比有助于理解两者的分工。

一个 Transformer block 的主体，其实就是 attention 加 MLP：先让 token 和上下文交流，再让 token 自己加工。

---

## Residual Connection：不要把原信息弄丢

Residual connection 看起来名字很吓人，但操作非常简单：

```text
输出 = 原始输入 + 这一层新算出来的变化
```

在 attention 子层里，可以写成：

```text
x = x + SelfAttention(...)
```

在 MLP 子层里，可以写成：

```text
x = x + MLP(...)
```

也就是说，模型不是每一层都把旧表示彻底替换掉，而是在旧表示的基础上加一部分新信息。

这个设计对深层网络非常关键。如果没有 residual connection，每一层都对信息做一次彻底变换。层数一深，早期信息可能被不断冲掉，梯度也很难稳定传回前面的层。模型越深，这个问题越严重。

Residual connection 给信息和梯度开了一条“高速公路”。原始信息可以更顺畅地传到后面，每一层只需要学习“在原基础上改一点”。这有点像改文章：不是每一轮都重写全文，而是在原稿上做局部修改。后者明显更稳定，也更容易控制。

所以 residual connection 的核心意义是：**让深层网络更容易训练，让信息和梯度更容易流动。**

---

## LayerNorm / RMSNorm：不要让数值尺度失控

理解了 residual connection，再看 LayerNorm / RMSNorm 就顺很多。

Residual connection 会不断做：

```text
x = x + 新信息
```

但如果每层都在加，隐藏向量的数值尺度可能越来越不稳定。比如一开始向量里的数值大概是：

```text
[0.2, -0.1, 0.5, ...]
```

经过很多层以后，如果没有控制，可能变成：

```text
[23.7, -51.2, 88.4, ...]
```

这只是一个夸张例子，但表达的是同一个问题：如果激活分布不断漂移，训练会变得非常困难，可能收敛很慢，甚至直接发散。

LayerNorm / RMSNorm 的作用，就是在每一层处理前，把 token 表示的尺度整理到相对稳定的范围。它们不负责“理解语义”，也不是为了直接提升表达能力，而是为了让深层 Transformer 可以稳定训练。

LayerNorm 会同时做 re-centering 和 re-scaling，也就是减去均值、再除以标准差。RMSNorm 更轻量，它不减均值，只根据 root mean square 控制整体尺度。很多现代 LLM 使用 RMSNorm，是因为它计算更简单，效果通常也足够好。

现阶段不需要被公式吓住，只要先记住：

```text
LayerNorm / RMSNorm 都是为了稳定数值尺度
RMSNorm 是更轻量的版本
```

---

## Pre-Norm：为什么现代 LLM 喜欢先 Norm 再计算

Normalization 放在哪里，也会影响训练稳定性。

早期 Transformer 常见的是 Post-Norm：

```text
Self-Attention
↓
Residual Add
↓
LayerNorm
```

也就是先计算、相加，再做 normalization。

现代 LLM 更常见的是 Pre-Norm：

```text
LayerNorm
↓
Self-Attention
↓
Residual Add
```

也就是先把输入状态整理稳定，再送进 attention 或 MLP，最后和原输入做 residual add。

Pre-Norm 的好处是更适合深层网络训练。每个子模块拿到的输入尺度更稳定，同时 residual path 更顺畅，梯度更容易沿着这条路径传播。对于几十层、上百层的 LLM，这类稳定性差异会变得非常重要。

因此，一个现代 Pre-Norm Transformer block 可以简化成：

```text
x = x + SelfAttention(Norm(x))
x = x + MLP(Norm(x))
```

这两行公式基本就概括了很多现代 Transformer block 的骨架。

---

## 把整层串起来

现在重新看一层 Transformer block，就不应该再觉得它是一堆零散术语了。它的流程大概是：

```text
输入 x
 ↓
LayerNorm / RMSNorm
 ↓
Self-Attention：看上下文
 ↓
Residual Add：把原来的 x 加回来
 ↓
LayerNorm / RMSNorm
 ↓
MLP / FFN：自己加工
 ↓
Residual Add：继续保留原信息
 ↓
输出到下一层
```

不同模型会有细节差异，比如使用 LayerNorm 还是 RMSNorm，attention 后是否有额外投影，MLP 用 GELU、SwiGLU 还是其他激活函数。但核心思想没有变：**attention 建立上下文关系，MLP 做非线性加工，residual 保留信息通路，norm 稳定训练尺度。**

这也是为什么 Transformer 能堆很多层。每一层不是推翻上一层，而是在上一层表示上做增量加工；每一层处理前又通过 normalization 把数值状态整理好。这样模型才能在很深的情况下继续训练，而不是很快变得不可控。

---

## 为什么这对工程理解有用

如果只把 Transformer 看成“一个会生成文本的黑盒”，这些结构细节似乎离业务很远。但一旦你做 AI Native 应用、模型部署、性能优化或面试准备，它们会频繁出现。

理解 attention 和 MLP 的分工，能帮助你理解为什么 KV Cache 主要缓存 K / V，而不是缓存所有中间计算；理解 residual connection，能帮助你理解为什么深层模型不是简单堆层数那么粗暴；理解 LayerNorm / RMSNorm，能帮助你理解为什么大模型训练非常依赖数值稳定性，也为什么推理框架会专门优化 normalization kernel。

更重要的是，这些概念会让你从“背术语”进入“看结构”。你不再只是记住 Transformer 有 attention、FFN、norm、residual，而是知道它们为什么必须一起出现：一个负责上下文，一个负责加工，一个负责保留，一个负责稳定。

---

## 核心要点

- Transformer block 的主体是 self-attention + MLP / FFN：前者负责上下文交互，后者负责语义加工
- Residual connection 的形式是 `输出 = 原输入 + 新变化`，它让深层网络更容易保留信息和传播梯度
- LayerNorm / RMSNorm 的作用是稳定隐藏向量的数值尺度，避免深层训练中激活分布不断漂移
- LayerNorm 做 re-centering + re-scaling，RMSNorm 主要做 re-scaling，因此更轻量
- 现代 LLM 常用 Pre-Norm：`x = x + Module(Norm(x))`，因为它对深层网络训练更稳定
- 一句话记忆：attention 负责“看上下文”，MLP 负责“自己加工”，residual 负责“不丢原信息”，norm 负责“不让数值失控”
