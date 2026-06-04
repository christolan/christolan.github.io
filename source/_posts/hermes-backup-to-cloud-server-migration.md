---
title: 从 Docker backup 到云服务器：Hermes Agent 迁移实录
published: true
date: 2026-06-04 13:28:01
cover: /images/hermes-backup-to-cloud-server-migration.png
top_img: /images/hermes-backup-to-cloud-server-migration.png
categories:
  - AI
tags:
  - AI
  - Agent
  - Docker
  - Hermes Agent
  - 工具链
---

**一次 Agent 迁移真正困难的地方，往往不是把文件复制过去，而是确认它在新环境里还像原来一样“活着”。**

昨天我把 Hermes Agent 从 Docker backup 文件迁移到了当前这台云服务器。表面上看，这只是一次恢复：把备份里的配置、会话、工作区和运行状态搬到新机器，然后把 gateway 拉起来。但真正做下来，我发现这类迁移和普通 Web 服务不太一样。一个 AI Agent 不是只有一个进程，它更像一套小型操作系统：有长期记忆，有 session history，有 cron，有浏览器，有子进程，有 delegation，有 MCP，还有 QQ 这样的消息入口。

所以迁移完成以后，我没有只看“服务是否启动”，而是做了一轮从底层路径到上层聊天入口的验收。最后的结论是：Hermes 主链路整体可用，但也暴露出两个不阻塞使用的问题，一个是 Mem0 依赖缺失，一个是 QQ approval button 的 `dm` / `c2c` 授权漂移。

## 先确认 Agent 站在正确的地面上

迁移后的第一件事，是确认 runtime identity 和路径基线。对 Hermes 这种工具型 Agent 来说，路径不是细节，而是状态边界。`HERMES_HOME`、`HOME`、工作目录、profile、tool subprocess 的环境变量，只要有一个理解错了，后续就可能出现“配置明明存在但工具读不到”的问题。

这次新环境里的 Hermes 运行在 `/root/.hermes`，而官方安装目录在 `/usr/local/lib/hermes-agent`。我重点确认了 config、`.env`、sessions、state database、workspace repos、GitHub CLI 配置这些关键位置是否都落在迁移后的预期路径里。这个阶段的意义不是追求输出好看，而是先回答一个基础问题：现在这个 Agent 到底在用哪套状态？

这也呼应了我之前那篇关于 Docker 迁移里 `HOME` 边界的记录。Agent 的状态不只是文件，还是进程、环境变量和工具链共同解释出来的结果。如果 `gh`、`npm`、`git`、browser、cron 各自看到的是不同的 home，那么迁移表面成功，实际使用时还是会不断踩坑。

## 验收清单比“能聊天”更重要

确认路径之后，我按模块做了一组 smoke test。底层先看 Hermes CLI 和 doctor，再看 config/profile 是否一致；中间层检查 gateway service 的环境、migration path mapping、repositories/workspace 和基础工具；上层再检查 web search、browser、cron、background process、delegation、MCP、session_search、memory、QQ 收发。

这轮检查里，大部分结果是正向的。gateway 正常运行，QQ 可以收到消息也可以发送响应；web search 和 browser 可以访问外部页面；cron scheduler 能识别已有的 `release-watch` 定时任务；background process 可以启动、等待并捕获输出；delegation 子代理能被拉起并返回结果；Flomo MCP 也已经注册并启用。

我特别注意 Flomo 的检查边界。因为 Flomo 是个人笔记资产，所以只验证了 MCP server 是否注册、工具 schema 是否存在，没有读取、搜索或写入任何笔记内容。这也是 Agent 迁移里很容易被忽略的一点：验收不等于把所有能力都实际调用一遍。对个人数据或外部服务来说，应该先区分“注册层验证”和“内容层验证”。

## session 和 memory 是 Agent 的连续性

普通服务迁移最关心数据库能不能连上，Agent 迁移则还要关心会话和记忆是否连续。Hermes 的 `state.db`、sessions 目录、内置 memory 和 user profile 都属于这部分。

这次 `session_search` 能正常返回历史会话，本地 `state.db` 也存在并能读到 messages / sessions 表，说明迁移没有把过去的上下文切断。对一个长期使用的 Agent 来说，这很关键。因为它不只是“能回答问题”，而是应该知道过去做过什么、哪些偏好不能违背、哪些项目路径和工作流已经约定过。

不过，memory 这块也暴露出一个问题：配置里曾经启用了 Mem0 provider，插件层看起来 installed / available，但 Hermes venv 里实际没有 `mem0` Python 包。结果就是日志里出现 `mem0 package not installed`，并触发 Mem0 sync failure 和 circuit breaker。

这个问题不影响内置 memory、session_search 和 QQ 主链路，所以它不是阻塞项。但它提醒我：迁移后不能只看配置文件“写了什么”，还要看运行时依赖“真的在不在”。尤其是 provider / plugin 这类扩展能力，经常会出现配置迁过去了、依赖没迁过去的情况。

后续我没有选择安装 `mem0ai`，因为我并不想继续使用 Mem0。更合理的清理方向是停用外部 Mem0 provider，保留 Hermes 内置 memory / user profile，然后删除或归档 `mem0.json` 和 `.env` 里的 `MEM0_API_KEY`。迁移之后的修复，不一定是把所有旧能力都恢复，有时是借迁移机会把不再需要的外部依赖清掉。

## QQ approval button 暴露了平台语义差异

第二个问题更有意思：QQ 普通收发正常，但 approval button 点击会被判定为 unauthorized。日志里能看到，operator openid 明明和当前用户一致，却仍然被拒绝。

后来定位到原因是私聊命名不一致。Hermes 通用 session key 使用 `dm`：

```text
agent:main:qqbot:dm:<user_openid>
```

而 QQ interaction 事件侧使用的是 `c2c`。当前授权逻辑只接受 `c2c`，不接受 `dm`，于是同一个用户点击按钮，也会因为 session type 不匹配而被误拒绝。

这个问题的最小修复其实很清楚：在 QQ adapter 的 approval authorization 中，把 `dm` 和 `c2c` 当作私聊同义词，同时继续要求 `operator_openid == chat_id`。这样既不放宽用户身份校验，也不改写 pending approval 的原始 session key。

但我最终没有在本机直接热修 Hermes Agent 官方代码。原因也很简单：本机改官方安装目录，后续升级容易被覆盖，也会让环境变得不可解释。更好的方式是把它记录为已知兼容性问题，必要时整理成 issue 或 PR 反馈给官方。在此之前，QQ 主链路继续可用，涉及危险命令审批时改用文本授权或避开 button 依赖。

## 迁移不是复制，而是重新建立信任

这次迁移给我的最大经验是：Agent 迁移不能只用“服务启动成功”来验收。它至少需要覆盖四层。

第一层是身份和路径，包括 `HERMES_HOME`、`HOME`、profile、workspace、config 和 `.env`。第二层是工具执行能力，包括 shell、git、browser、web、background process。第三层是长期状态，包括 sessions、state database、memory、cron。第四层是外部入口和集成，包括 QQ、MCP、Flomo、approval workflow。

只有这四层都过一遍，才能说这个 Agent 在新机器上重新建立了可信运行环境。否则它可能“看起来能聊天”，但一到真正执行任务，就在某个隐藏边界上失败。

这也是为什么我越来越觉得，AI Agent 的工程复杂度不只在模型能力。模型负责推理，但 Agent runtime 负责把推理落到现实世界里。只要它要读文件、跑命令、连服务、记住历史、等待审批，它就会面对传统系统工程里那些老问题：路径、权限、依赖、进程、网络、状态、日志。

## 最后的状态

这次 Docker backup 到云服务器的迁移，最终验收结果可以概括为一句话：Hermes 当前整体可用，QQ 主链路和核心工具链都正常。

通过的部分包括 Runtime identity/path、Hermes CLI/doctor、Config/profile、Gateway service env、Migration path mapping、Repositories/workspace、基础工具、Web/Browser、Cron、Background process、Delegation、MCP/Flomo 注册、session_search、state.db/sessions、built-in memory、QQ 收发。

剩下两个非阻塞问题则分别处理：Mem0 缺依赖，不再按“安装依赖”修复，而是计划停用并清理外部 provider；QQ approval button 的 `dm` / `c2c` 授权漂移，不在本机热修官方代码，后续更适合走 issue / PR 或等待上游修复。

迁移完成以后，我对这类 Agent 运维有了一个更朴素的判断标准：不要只问它是否在线，要问它是否还保留了原来的身份、记忆、工具和边界。只有这些都对了，Agent 才是真的迁过去了。
