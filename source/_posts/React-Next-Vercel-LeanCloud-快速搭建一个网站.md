---
title: React/Next/Vercel + LeanCloud 快速搭建一个网站
published: true
categories: 技术
tags:
  - 前端
  - Vercel
  - LeanCloud
abbrlink: 1100269214
date: 2024-07-07 21:09:48
---

## 引言

如果你是一个前端开发者，想要独立上线一个网站，那么你大概会遇到下面这些问题：

- 前端如何部署？
- 域名和 SSL 证书如何配置？
- 后端使用什么框架？
- 后端如何部署？
- 使用什么数据库？
- 如何自动化部署？
- 能不能前后端放在同一个项目里，使用 JavaScript 全栈开发？

如果这些困难让你感到第一步很难迈出，那么可能你的很多想法都会止步于自己的脑海，而无法变成现实。中国互联网多年的经验已经证明了，快速的跑通全流程，然后再逐步迭代优化，才是效率最高的方案。

值得高兴的是，应该有不少人都曾有过这样的苦恼，所以现在市面上诞生了大量的解决方案。今天我要介绍的就是其中的一种。我的技术选型可能不是各方面都完美的最优解，但是它一定足够简单，简单到一位合格的前端工程师，可以在极短的时间就能跑通全流程。

## 数据库部分

对于一名前端工程师来说，数据库部分可能会比较棘手。所以我先说一说这部分的问题如何解决。

[LeanCloud](https://leancloud.app/) 是一家提供 serverless 云服务的公司。如果你有折腾博客的经历，那么大概会知道 [Valine](https://valine.js.org/) 这个评论系统，它就是基于 LeanCloud 实现的存储功能。

可以通过具体的 JavaScript 代码，让开发者们感受一下它有多方便：

SDK 安装

```shell
npm install leancloud-storage --save
```

初始化 SDK

```javascript
const AV = require('leancloud-storage');

// 下面的信息在 LeanCloud 控制台中找到
AV.init({
  appId: 'your-app-id',
  appKey: 'your-app-key',
  serverURL: 'https://your_server_url',
});
```

数据操作

```javascript
// 声明 class
const Todo = AV.Object.extend('Todo');

// 构建对象
const todo = new Todo();

// 为属性赋值
todo.set('title', '工程师周会');
todo.set('content', '周二两点，全体成员');

// 将对象保存到云端
todo.save().then(
  (todo) => {
    // 成功保存之后，执行其他逻辑
    console.log(`保存成功。objectId：${todo.id}`);
  },
  (error) => {
    // 异常处理
  }
);
```

看到这里我相信大家就已经知道 LeanCloud 的数据存储有多简单了。基于我个人的使用感受，对 LeanCloud 的印象大概如下：

- 完成了对数据库的封装，可以直接使用 JavaScript SDK 操作数据库
- 提供了 web 端的数据库管理界面，让开发者们可以直接在浏览器中手动管理数据
- 创建项目、拿到 SDK 的初始化信息，直接就可以开始使用，无需关心部署和运维
- 免费额度足够个人小型项目使用

## 前后端部分

我选择的是 Next.js 框架。Next.js 基于 React，也是目前 [React 官方文档](https://react.dev/learn/start-a-new-react-project) 放在首位的框架。它的优势在于可以方便地在项目里写后端 api，与前端统一打包部署，相当于一个全栈框架。

后端 api 层的作用如下：

- 方便根据自己的需求，对数据库操作进行适当地封装，将一些耗时计算放到服务端来做
- 在后端使用 LeanCloud 的 SDK 进行数据操作，然后提供接口供前端调用，这样更加安全

Next 项目部署使用 Vercel，因为 Next 本身就是 Vercel 公司的产品，所以 Vercel 部署 Next 项目极其简单。基于我个人的使用感受，对 Vercel 的印象大概如下：

- 将 Next 项目推到 GitHub 上，然后在 Vercel 上导入，就可以实现自动部署
- 免费的托管
- 自带域名和 SSL 证书
- 自带流水线（默认 push 代码即可触发）

## 总结

下面是整套方案的简单架构图：

{% image /images/arch.png 基本架构 bg:'#ffffff' %}

我希望这篇文章能够帮你快速出发，让你的想法迅速进入迭代流程，然后打磨出你自己心爱的产品。
