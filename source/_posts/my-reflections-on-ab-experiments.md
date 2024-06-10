---
title: 相伴三年，我对 AB 实验的一些感悟
date: "2024-05-20T23:03:00+00:00"
published: true
feature: ""
---

总结当下我对 AB 实验优缺点的思考。

<!-- more -->

## 什么是 AB 实验

AB 实验是一个理科概念，我们将被测试的许多个体分为两组，确保两组之间的变量唯一，那么最终这两组表现出来的差异性，就是这个变量导致的。

中国互联网一直是小步快跑的节奏，为了验证跑出的每一步是否正确，需要对部分用户生效新功能，观察这些用户的指标与其他用户的差异，来判断这个新的功能是否为产品带来了收益。

## AB 实验之熵：冗余的代码

既然需要新功能只对部分用户生效，那么必然需要同时保留新老功能，然后通过一些手段控制代码执行时的逻辑分叉。相当于类似的功能，项目里会有多套代码来实现。这听起来似乎没什么，但是一旦实验多起来了，项目里几乎就有数倍于单个用户所有功能的代码。

海量的实验，给项目带来了海量的逻辑分叉，在这样的项目里开发，只能说是如履薄冰。不仅仅是开发者熟悉项目的成本巨大，对于测试同学来讲，测试工作量更是巨大。

这些冗余的代码，也必然让我们的编译产物体积暴涨。这也是为什么，在单个用户的眼里，很难理解一个简单的 app 安装包体积居然如此巨大。

## AB 实验之熵：难以捉摸的行为

如今的互联网 app，讲究一个千人千面。这已经不仅仅体现在大数据相关的一些功能上，app 本身的行为也是如此，因为这个 app 上可能存在着成千上万的 AB 的实验，组合下来有难以枚举的可能性。

正是因为实验上千人千面，所以你很难保证你写的代码在所有的组合下都是没有 bug 的。这就导致测试很难保证代码的质量，一些奇怪的问题往往需要功能上线以后，命中某些特定实验才能触发。

作为开发者，都希望自己的代码 bug 越少越好。但是难以枚举的可能性，让你通过流程图来表达一个功能的逻辑变得极其困难。在这样的土壤里，那些循规蹈矩、科班出生的种子都很难发芽，往往都是被打磨成防御性编程的老油条。

## AB 实验之功：明确的收益

作为一个大头兵，我们很难将公司本年度的增长与自己的贡献挂上钩。

这样的现状下，一次次实验中表现出来的差异，就是可以衡量我们产出的工具。你可以通过指标的变化，来感知到自己所作的事情为这个产品带来了什么，这会对自身工作的认同感有很大的提升。尤其当你自己发起了一个需求，自己建立了相关的指标，然后通过实验观察到，自己的需求让这项指标正在往好的方向变化，成就感油然而生。

有了明确的收益，你就可以在绩效评估、晋升等流程中，拿出证明自己工作产出的证据。

我曾经在使用淘宝的过程中感慨：为什么我经常需要点击“全部订单”，而这个入口却做的这么难点呢？想起阿里跳槽过来的同事说，在阿里做需求从来不开实验，我产生了一个猜测：是不是因为这个入口的调整很难被证明是有价值的，所以大家都不愿意花时间与精力来推进这个优化呢？

## AB 实验之功：发现隐蔽的问题

作为开发者，不害怕 bug 有多严重，而是害怕 bug 有多隐蔽。

在我推进的功能中，就有通过实验指标发现隐藏 bug 的记录。有一些 bug 的表现，从用户角度来讲感知不是很明显，但其实默默影响到了用户的行为。这些行为上的细微差异，只有通过线上海量的数据来观察，才能发现一丢丢蛛丝马迹。

很多时序上非常微妙的差异，就会导致我们访问变量的时候拿到了预期之外的值，这样刁钻的时序可能很难通过某个人的使用来触发，而线上大量用户的使用，则让这样的 case 被捕捉到的概率大大提升。

## 总结：我们为什么喜欢，又为什么讨厌 AB 实验

我们喜欢 AB 实验，是因为：

1.  AB 实验可以为我们的日常工作提供明确的价值衡量
2.  AB 实验可以帮助我们发现隐蔽的 bug

我们讨厌 AB 实验，是因为：

1.  AB 实验代码污染了我们的项目
2.  过多的 AB 实验，让开发者难以理清代码逻辑

为了让 AB 实验的优点放大、缺点缩小，我认为有一件事情是十分紧要的：**及时清理 AB 实验的代码，减少 AB 实验为代码增加的熵。**