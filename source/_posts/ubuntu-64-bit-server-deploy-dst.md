---
title: 64 位 Ubuntu 部署饥荒独立服务器
date: "2024-06-05T17:42:00+00:00"
published: true
feature: ""
---

## 关键点

- 安装 steamcmd
- 安装 Don't Starve Together Dedicated Server
- 准备一个存档
- 给存档添加 cluster_token.txt
- 添加服务器 mod 初始化命令
- 饥荒独立服务器，启动！

  <!-- more -->

## 【服务器】安装 steamcmd

steam 有官方的文档，告诉你应该如何安装 steamcmd：[https://developer.valvesoftware.com/wiki/SteamCMD:zh-cn#Linux](https://developer.valvesoftware.com/wiki/SteamCMD:zh-cn#Linux) ，**官方强调了不要使用 root 用户来操作**，其中核心的部分如下：

```shell
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install lib32gcc-s1 steamcmd

```

安装成功的标志，是可以在任何目录下直接用 steamcmd 命令。很多教程会把这一步弄的十分麻烦，其实完全没有必要。

## 【服务器】安装 Don't Starve Together Dedicated Server

有了 steamcmd 以后，安装 Don't Starve Together Dedicated Server 就很容易了，需要注意的一点是调整一下安装路径，后续的使用会方便很多，默认的安装路径不仅层级深，而且因为目录名字包含空格等一些特殊字符，会有很多坑。这一点几乎所有的教程都包含了。需要输入的命令和顺序如下：

```shell
steamcmd
Steam>force_install_dir /home/ubuntu/dst_server
Steam>login anonymous
Steam>app_update 343050 validate

```

其中 `force_install_dir` 指定自定义的安装路径，/home/ubuntu/dst_server 是我使用的目录，可以按需修改。343050 是饥荒在 steam 上的 app id。

如果某天突然游戏里找不到服务器了，可能是因为客户端和服务器版本不匹配，重新执行一下上面的命令，就可以更新服务器到最新版本。

安装完成后，进入 /home/ubuntu/dst_server 目录，长这样：

```shell
ubuntu@VM-4-6-ubuntu:~/dst_server$ ll
total 39560
drwxrwxr-x  8 ubuntu ubuntu     4096 Jun  6 09:37 ./
drwxr-x---  8 ubuntu ubuntu     4096 Jun  6 09:30 ../
drwxrwxr-x  4 ubuntu ubuntu     4096 Jun  6 09:37 bin/
drwxrwxr-x  4 ubuntu ubuntu     4096 Jun  6 09:37 bin64/
drwxrwxr-x 11 ubuntu ubuntu     4096 Jun  6 09:37 data/
-rwxrwxr-x  1 ubuntu ubuntu   243381 Jun  6 09:36 dontstarve.xpm*
drwxrwxr-x  2 ubuntu ubuntu     4096 Jun  6 09:37 linux64/
drwxrwxr-x  2 ubuntu ubuntu     4096 Jun  6 09:37 mods/
drwxrwxr-x  4 ubuntu ubuntu     4096 Jun  6 09:37 steamapps/
-rwxrwxr-x  1 ubuntu ubuntu 40226316 Jun  6 09:33 steamclient.so*
-rwxrwxr-x  1 ubuntu ubuntu        7 Jun  6 09:35 version.txt*

```

继续进入 bin64 目录，长这样：

```shell
ubuntu@VM-4-6-ubuntu:~/dst_server/bin64$ ll
total 5608
drwxrwxr-x 4 ubuntu ubuntu    4096 Jun  6 09:37 ./
drwxrwxr-x 8 ubuntu ubuntu    4096 Jun  6 09:37 ../
-rwxrwxr-x 1 ubuntu ubuntu      66 Jun  6 09:33 dontstarve*
-rwxrwxr-x 1 ubuntu ubuntu 5476768 Jun  6 09:33 dontstarve_dedicated_server_nullrenderer_x64*
-rwxrwxr-x 1 ubuntu ubuntu  236596 Jun  6 09:33 dontstarve.xpm*
drwxrwxr-x 2 ubuntu ubuntu    4096 Jun  6 09:37 lib64/
drwxrwxr-x 2 ubuntu ubuntu    4096 Jun  6 09:37 scripts/
-rwxrwxr-x 1 ubuntu ubuntu       6 Jun  6 09:33 steam_appid.txt*

```

其中 dontstarve_dedicated_server_nullrenderer_x64 就是我们需要用到的可执行文件，直接调用就会启动一个饥荒独立服务器。

## 【本地】准备一个存档

有很多种创建存档的方案，我最推荐的方式是直接进入游戏创建。这样可以使用游戏内的编辑器修改存档配置。创建的过程中需要注意：

- 使用一个好记的名字，方便从存档中找出对应的文件夹，也方便后续在游戏里找到自己的服务器；
- 如果公网部署最好添加密码，内网部署可以随意；
- 配置好服务器 mod，后续手动配置 mod 会很麻烦。

配置好以后开始游戏，进入到选人的界面就可以退出了。创建好的存档通常会在 文档\\Klei\\DoNotStarveTogether 目录下面，一个存档就是一个形如 Cluster_x 的目录，其中 x 是一个递增的数字。

进入存档里面，用记事本打开 cluster.ini 文件，通过 cluster_name 可以匹配到刚刚创建好的那一个。

## 【本地】给存档添加 cluster_token.txt

进入 klei 的官网 [https://accounts.klei.com](https://accounts.klei.com) 使用 steam 账号登录，然后进入 [https://accounts.klei.com/account/game/servers?game=DontStarveTogether](https://accounts.klei.com/account/game/servers?game=DontStarveTogether) 页面，就可以创建服务器 token。一个合法的 token 是一串 pds- 开头的字符。

我们需要在前面创建好的 Cluster_x 目录下创建一个 cluster_token.txt 文件，然后将 pds- 开头的这一串字符复制粘贴进去。

这样一个可用的存档就准备好了。

## 【服务器】添加服务器 mod 初始化命令

如果配置了服务器 mod，那么需要有一些操作来加载这些 mod，否则不会生效。

首先确认用到 mod 的 id，然后编辑 /home/ubuntu/dst_server/mods 目录下的 dedicated_server_mods_setup.lua 文件。这个文件默认都是注释，解释了 `ServerModSetup` 和 `ServerModCollectionSetup` 两个方法的作用。

在文件后面添加对 `ServerModSetup` 方法的调用，有几个 mod 就添加几个 ，就会在启动服务器的时候自动下载这些 mod：

```markup
--There are two functions that will install mods, ServerModSetup and ServerModCollectionSetup. Put the calls to the functions in this file and they will be executed on boot.

--ServerModSetup takes a string of a specific mod's Workshop id. It will download and install the mod to your mod directory on boot.
        --The Workshop id can be found at the end of the url to the mod's Workshop page.
        --Example: http://steamcommunity.com/sharedfiles/filedetails/?id=350811795
        --ServerModSetup("350811795")

--ServerModCollectionSetup takes a string of a specific mod's Workshop id. It will download all the mods in the collection and install them to the mod directory on boot.
        --The Workshop id can be found at the end of the url to the collection's Workshop page.
        --Example: http://steamcommunity.com/sharedfiles/filedetails/?id=379114180
        --ServerModCollectionSetup("379114180")

ServerModSetup("xxxxxxxx")

```

## 【服务器】饥荒独立服务器，启动！

进入前面提到过的 bin64 目录下，执行 `./dontstarve_dedicated_server_nullrenderer_x64` 命令，就会开始启动存档，同时打印出大量的日志，其中靠前的部分会有这样一段：

```shell
[00:00:00]: Token file not found: /home/ubuntu/.klei//DoNotStarveTogether/Cluster_1/cluster_token.txt, success: F, len: 0
[00:00:00]: Token file not found: /home/ubuntu/.klei//DoNotStarveTogether/Cluster_1/cluster_token.txt, success: F, len: 0

```

这里面包含了一些重要的信息： /home/ubuntu/.klei//DoNotStarveTogether/Cluster_1 就是默认用来启动独立服务器的存档路径。

上面的这一次启动是不会成功的，因为我们还没有把正确的存档放上去。这次启动只是为了自动创建好一些外层的目录，看到下面的日志就可以中断退出了。

```shell
[00:00:03]: !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[00:00:03]: !!!! Your Server Will Not Start !!!!
[00:00:03]: !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[00:00:03]: No auth token could be found.
[00:00:03]: Please visit https://accounts.klei.com/account/game/servers?game=DontStarveTogether
[00:00:03]: to generate server configuration files

```

然后需要使用 sftp 工具连接到服务器，做两件事情：

- 删除 /home/ubuntu/.klei//DoNotStarveTogether 目录下的 Cluster_1 目录；
- 将准备好的 Cluster_x 上传到 /home/ubuntu/.klei//DoNotStarveTogether 目录下，改名为 Cluster_1；

再次执行 `./dontstarve_dedicated_server_nullrenderer_x64` 命令，就可以正常启动了。

## 总结

按照关键点一步步地来，理解了这个过程以后其实部署饥荒独立服务器非常简单。
