---
title: 删代码到底有多难
date: '2024-03-09T08:40:00+00:00'
published: true
feature: ''
---
## 引言

阮老师最近一期的周刊中，聊到了技术债的问题

[科技爱好者周刊（第 292 期）：所有代码都是技术债](https://www.ruanyifeng.com/blog/2024/03/weekly-issue-292.html)

下面是相关内容的节选：

> 所有代码都是技术债
> 
> 代码是公司的资产，公司总是鼓励大家多写代码。但是，很多人（尤其是管理层）没有意识到，代码也是负债。
> 
> 代码越多，债越多，这就是程序员常说的"技术债"。
> 
> 今天我想谈谈，什么是"技术债"？为什么你拥有的代码太多，不是一件好事。
> 
> "技术债"（technical debt）源自著名程序员沃德·坎宁安（Ward Cunningham）的一篇文章。他写了一句话："交付代码就像负债累累。"
> 
> 你的代码一旦进入生产环境，就像背上一笔债务，将来需要不断支付利息，除非代码不再使用。
> 
> 这个比喻获得了共鸣，人们把代码带来的负担，称为"技术债"。
> 
> 为什么代码好比负债累累？这有两个原因。
> 
> 第一个原因是，由于各种限制，代码的实现有问题，包含了 Bug，或者选择了有问题的组件，后期需要修改或重写。
> 
> 第二个原因是，即使代码是完美的，但由于技术进步，它会逐渐腐化过时，后期需要不断维护和更新，这通常比原始开发成本更高。
> 
> 这意味着，无论多么小心，上线的代码总是有"技术债"。 可以这样说，所有的代码都是技术债。
> 
> "技术债"的可怕之处，在于你必须按时偿还，如果拖着不还，它就会像雪球一样越滚越多，维护成本越来越高，直到再也无法维护，只能放弃这段代码。
> 
> 既然所有代码都是技术债，程序员写代码时，就必须考虑到它的长期成本，尽量减轻自己或别人日后的负担（利息）。
> 
> 一个基本的事实是 代码越少，技术债越小；没有代码，就没有技术债。从这个角度看，软件开发的正确做法是下面两点。
> 
> （1）冗余的代码都要删除。
> 
> （2）只实现那些必须实现的功能，除非绝对必要，不要引入新功能。新功能必然带来新的代码，而且新功能一旦添加，就很难废除，总是会保留下来。

## 现状

我们公司是一家 AB 实验文化非常浓厚的公司，几乎所有的功能，都要经过 AB 实验。

即给一部分用户下发参数 A 走线上逻辑、另一部分用户下发参数 B 走新需求逻辑，通过对比 A 和 B 的各项指标，来评判 B 的功能是否应该上线。

在经过 AB 实验之后，最终不管新需求是否上线，A 和 B 的代码都会残留在我们的代码里。除非你主动删除代码，再协调 QA 人力帮忙回归，才能顺利将不需要的代码给删掉。

但众所周知，**协调外部人力，是一件非常麻烦的事情**。至少你要说明自己申请人力的理由、阐明收益，然后对方根据收益大小，将手里的事情排个优先级，才能决定要不要、以及什么时候要帮你处理这件事情。

这样明确地门槛摆在这里，大家删除代码的意愿自然就不强烈了。虽然日常开发中，难免要和各种实验做斗争，增加了很多阅读代码的干扰，大家普遍还是**乐于自己吃苦，不愿麻烦别人**。欠下巨额债务，依旧死撑硬扛。

## 尝试

忍无可忍的我，在组内发起了代码清理计划。大体的想法是：每隔一段时间统一收集一下可以清理的代码、统一提需求，申请 QA 人力测试。

清理计划本身比较容易得到大家的认可，毕竟大家不是不想清理，而是不想花费时间精力协调外部人力罢了。

但是得到认可并不等同于得到行动上的支持。我发起了好几次清理，最终主动提交代码的人几乎没有。还是得一个个地去聊，去分配任务，才会有人提交代码。这或许就是职场的常态：**让别人做的事情，越简单、越不用动脑子，才越容易得到行动上的支持**。

为此，我需要将流程尽可能地简化，将繁琐的事情全都处理好，然后将任务分解成十分明确的指令，传达给组内对应的人，最终才能将整个流程顺利地跑起来。

## 展望

在一次次清理需求的尝试中，需要沉淀出一套明确可执行地方法论，才能将这件事情的影响力向外拓展。

**对于一个团队来讲，应该做到只需要找到一个负责的人，就可以将流程顺利地跑起来。这个人需要做的事情是什么，就是我的方法论的核心。**

为此，还需要经过一些探索，我的计划是：在 2024 全年产出中，可以拿出一份明确的方案，并且找到一个其他的团队落实出来。这份方案应该包括以下一些部分：

*   如何将清理任务简化成大家都没有负担的子任务（便于借助大家的力量）
    
*   如何量化清理本身的收益（给清理负责人一些动力）