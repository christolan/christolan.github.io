---
title: 基于 docker-dst-server 的饥荒独立服务部署
date: '2024-06-01T19:50:00+00:00'
published: false
feature: ''
---
# 概述

> WARNING：教程不适合完全的新手，需要一定的 Linux 服务器运维和 docker 知识。

部署饥荒独立服务器需要大量的命令行操作，为了简化部署过程，有人开源了 docker 部署的项目：[https://github.com/Jamesits/docker-dst-server](https://github.com/Jamesits/docker-dst-server) 。即便如此，依旧需要完成一些额外工作才能正常游戏，这个教程补充官方文档中缺失的一些细节。

我喜欢循序渐进，先用最少的步骤跑通整体流程、再给出进阶的操作优化游戏体验。

# 基础部分

## 1\. 启动 docker-dst-server

首先完成 jamesits/dst-server:latest 镜像的拉取。

官方 GitHub 项目的 readme 已经给出了启动的 docker 指令：

```markup
docker run -v ${HOME}/.klei/DoNotStarveTogether:/data -p 10999-11000:10999-11000/udp -p 12346-12347:12346-12347/udp -it jamesits/dst-server:latest

```

首先需要关注的部分是 `${HOME}/.klei/DoNotStarveTogether` ，这是主机上保存游戏存档数据的目录，可以根据需要调整。**为了方便描述，后面我假设这个目录为** `/data/dst` 。

其次需要关注的部分是 `-p 10999-11000:10999-11000/udp -p 12346-12347:12346-12347/udp` ，这是指定端口的映射。根据我的经验这个端口的映射很容易出问题，如果你的服务器是专门用来只部署饥荒的，不存在端口冲突的问题，那么我推荐直接使用 host 网络 `--network host`。

最后，如果需要 docker 容器在后台运行，可以加上 `-d` 参数。

最终启动命令为：

```markup
docker run -d -v /data/dst:/data --network host -it jamesits/dst-server:latest

```

**第一次的启动必然是跑不起来的**，因为默认生成的存档缺少正确的 cluster\_token.txt 文件，这次启动主要是为了自动生成部分文件，成功以后在 `/data/dst` 目录中的内容如下：

```markup
/data/dst/
├── Agreements
│   └── DoNotStarveTogether
│       └── agreements.ini
└── DoNotStarveTogether
    └── Cluster_1
        ├── Caves
        │   ├── backup/
        │   ├── save/
        │   ├── server_chat_log.txt
        │   ├── server.ini
        │   ├── server_log.txt
        │   └── worldgenoverride.lua
        ├── Master
        │   ├── backup/
        │   ├── save/
        │   ├── server_chat_log.txt
        │   ├── server.ini
        │   ├── server_log.txt
        │   └── worldgenoverride.lua
        ├── mods
        │   ├── dedicated_server_mods_setup.lua
        │   ├── INSTALLING_MODS.txt
        │   ├── MAKING_MODS.txt
        │   └── modsettings.lua
        ├── adminlist.txt
        ├── blocklist.txt
        ├── cluster.ini
        ├── cluster_token.txt
        └── whitelist.txt

```

这就说明容器启动后运行正常。

## 2\. 修改自动生成的配置

首先打开 klei 的官方网站，找到申请独立服务器的地方：[https://accounts.klei.com/account/game/servers?game=DontStarveTogether](https://accounts.klei.com/account/game/servers?game=DontStarveTogether) 通过 steam 账号登录。

输入名字、点击添加新服务器，随后可以找到形如下面这样的一串 token

```markup
pds-g^KU_xxxxxxxx^+xxxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxxx

```

然后修改 `/data/dst/DoNotStarveTogether/Cluster_1/cluster_token.txt` 文件，把上面的一串 token 复制粘贴进去，保存退出即可。

为了方便搜索到自己的服务器，推荐修改一下 `/data/dst/DoNotStarveTogether/Cluster_1/cluster.ini` 文件中的 cluster\_name

## 3\. 重启容器

修改好 cluster\_token.txt 后重启 docker-dst-server 的 docker 容器，此时服务器就已经可用，可以在列表中通过前面修改好的 cluster\_name 搜索到了。

Enjoy your game~

# 进阶部分

## 使用官方默认配置

基础部分创建出来的服务器使用的是 docker-dst-server 作者默认的配置，如果想要使用 klei 官方的默认配置，可以直接在 [https://accounts.klei.com/account/game/servers?game=DontStarveTogether](https://accounts.klei.com/account/game/servers?game=DontStarveTogether) 页面中点击“配置服务器”（修改好名称和简介方便搜索），再点击“下载配置”就会得到一个 MyDediServer.zip 文件。

*   【本地】将下载的 MyDediServer.zip 解压出来，得到一个名为 MyDediServer 的文件夹
    
*   【本地】把 MyDediServer 文件夹重命名为 Cluster\_1
    
*   【服务器】删除原本的存档 `/data/dst/DoNotStarveTogether/Cluster_1` 目录
    
*   【本地】将这个新的 Cluster\_1 整体上传到服务器的 `/data/dst/DoNotStarveTogether` 目录下代替原来的存档
    
*   【服务器】重启 docker-dst-server 容器
    

这样就会得到一个基于官方默认配置的服务器。
