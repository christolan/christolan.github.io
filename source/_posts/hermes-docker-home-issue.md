---
title: 从一次 Docker 迁移，看 Hermes Agent 里的 HOME 边界
published: true
date: 2026-05-17 12:16:29
cover: /images/hermes-docker-home-issue.png
top_img: /images/hermes-docker-home-issue.png
categories:
  - AI
tags:
  - AI
  - Agent
  - Docker
  - GitHub CLI
  - Hermes Agent
  - 工具链
---

**一次看起来很普通的 VPS 到 Docker 迁移，最后变成了对 Agent 运行时边界的一次小型考古：`HOME` 到底是谁的 `HOME`？**

最近我把 Hermes Agent 从 VPS 迁移到了 Docker。迁移本身并不复杂：数据目录挂载到容器里，gateway 跑起来，消息平台接上，日常使用基本恢复。真正让我停下来研究的，是 GitHub CLI。

我在容器里手动配置了 `gh auth login`，登录过程看起来也没问题，但回到 Hermes Agent 里使用工具时，GitHub CLI 却像是没有配置过一样。最开始我以为是 `gh` 的问题，或者是 token 没写进去。但查着查着，发现问题不在 GitHub CLI，而在 HOME。

## 一个很小但很烦的差异

Docker 里的 Hermes 使用 `HERMES_HOME` 作为持久化数据根目录，比如：

```text
HERMES_HOME=/opt/data
```

这个目录里放配置、session、memory、skill、日志等 Hermes 自己的状态。为了避免工具把配置写到容器里不持久的 `/root`，Hermes 又给工具子进程准备了一个 home：

```text
/opt/data/home
```

于是就可能出现这样的情况：

```text
Hermes gateway/main process:
HOME=/opt/data
HERMES_HOME=/opt/data

Tool subprocess:
HOME=/opt/data/home
HERMES_HOME=/opt/data
```

这就解释了为什么 GitHub CLI 会变得奇怪。你在某个 shell 里看到的 `~/.config/gh`，和 Hermes 工具子进程真正读取的 `~/.config/gh`，未必是同一个地方。

这类问题最麻烦的地方在于，它不是“命令失败”那么直接，而是所有东西看起来都对：token 有，配置文件也有，目录也持久化了。但只要 HOME 不是同一个，工具就会像失忆一样。

## HERMES_HOME 和 HOME 应该是两件事

研究之后，我觉得这里真正需要区分的是两个概念。

`HERMES_HOME` 应该是 Hermes 自己的状态根目录。它负责放 `config.yaml`、`.env`、sessions、skills、memory、cron 等等。这些是 Agent runtime 的内部状态。

`HOME` 则应该是这个 profile 里的“用户工具目录”。Git、SSH、GitHub CLI、npm、pip、各种 cloud CLI，都应该把自己的用户级配置写在这里。

所以在 Docker 默认 profile 下，一个更清晰的模型应该是：

```text
HERMES_HOME=/opt/data
HOME=/opt/data/home
```

对于 named profile，则可以是：

```text
HERMES_HOME=/opt/data/profiles/<profile>
HOME=/opt/data/profiles/<profile>/home
```

这样 `HERMES_HOME` 管 Hermes，`HOME` 管工具。两者职责不同，但同一个 profile 内，主进程和工具子进程对 `HOME` 的理解应该一致。

## 隔离边界应该在哪里

我理解 Hermes 为什么要做 per-profile HOME isolation。不同 profile 可能代表不同身份、不同 Git author、不同 SSH key、不同 GitHub 账号，隔离这些配置是合理的。

但我不太认同同一个 profile 里再拆出两套 HOME。因为这并不会带来真正的安全隔离：主进程和工具子进程仍然属于同一个 Agent、同一个 profile、同一个容器、同一组挂载目录。它们不是两个安全域，只是两个容易让人搞混的路径语义。

尤其是在 Docker 场景下，我更倾向于把 container 本身当成更干净的隔离边界。如果我真的想隔离两个 Agent，我会更愿意启动两个 container，挂载两套不同的数据目录，而不是在同一个 container 里靠不同 HOME 去模拟隔离。

这不是说 profile isolation 没有价值，而是隔离层次要清楚：

```text
不同 profile / 不同 container：应该隔离
同一个 profile 内部：应该一致
```

## 最后变成了一个 issue

这个问题后来被整理成一个 GitHub issue 提给了 Hermes Agent 官方：

[Docker/profile mode should avoid different HOME values between Hermes process and tool subprocesses](https://github.com/NousResearch/hermes-agent/issues/27250)

标题很朴素，核心观点也很简单：

```text
main process HOME == subprocess HOME
```

至少在同一个 active profile 里应该如此。跨 profile 可以不同，但同 profile 里不应该有两个互相竞争的 HOME。

## Agent 工具越来越像一套操作系统

这次排障有趣的地方在于，它不是一个单纯的 Docker 问题，也不是 GitHub CLI 问题，而是 Agent runtime 逐渐复杂之后自然暴露出来的边界问题。

过去我们说一个 CLI 工具，最多关心它的配置文件在哪里、PATH 是否正确。但 Agent 不一样。它会启动子进程、执行 shell、读写文件、调用 `gh`、跑 npm、管理 memory、加载 skill，还可能从聊天平台入口触发一整套自动化流程。

这时候，`HOME` 这种原本很底层、很朴素的环境变量，就突然变成了产品体验的一部分。

如果 Agent 说“我已经配置好了 GitHub CLI”，但真正执行任务时读的是另一个 HOME，用户就会困惑。如果 skill 里写了 `~/workspace`，但 Python 展开和 shell 展开不是同一个目录，维护成本就会慢慢堆起来。

所以我越来越觉得，AI Agent 的工程复杂度不只在模型和 prompt，也在这些非常传统的系统边界上：进程、环境变量、文件系统、权限、容器、凭据、工作目录。模型越强，越会把这些基础设施问题暴露得更明显。

这次迁移最终没有停留在“把 GitHub CLI 修好”上，而是变成了一个关于 Hermes Agent 设计边界的 issue。我觉得这正是使用开源 Agent 有意思的地方：你不是只在用一个工具，也是在和它一起发现那些还没被完全想清楚的边界。
