---
title: Don't Starve Together Dedicated Server 使用问题记录
date: "2024-05-12T15:30:00+00:00"
published: true
feature: ""
permalink: /dst-dedicated-server/
---

## 如何找到启动游戏的存档路径

启动 Don't Starve Together Dedicated Server，会弹出一个 terminal，从 terminal 中最顶部打印出来的日志就可以找到游戏存档的路径：

```markup
System Memory:
        Memory Load: 52%
        Available Physical Memory: 7669m/16234m
        Available Page File: 4706m/17258m
        Available Virtual Memory: 134213499m/134217727m
        Available Extended Virtual Memory: 0m
[00:00:00]:
Process Memory:
        Peak Working Set Size: 12m
        Working Set Size: 12m
        Quota Peak Page Pool Usage: 182k
        Quota Page Pool Usage: 182k
        Quota Peak Non Paged Pool Usage:14k
        Quota Non Paged Pool Usage: 13k
        Page File Usage: 3m
        Peak Page File Usage: 3m
[00:00:00]: PersistRootStorage is now APP:Klei//DoNotStarveTogether/Cluster_1/Master/
[00:00:00]: Starting Up
[00:00:00]: Version: 605310
[00:00:00]: Current time: Tue Jun 04 08:47:41 2024

[00:00:00]: Don't Starve Together: 605310 WIN32
[00:00:00]: Build Date: 2628
[00:00:00]: Mode: 64-bit
[00:00:00]: Parsing command line
[00:00:00]: Command Line Arguments:
[00:00:00]: Initializing distribution platform
[00:00:00]: Initializing Minidump handler
[00:00:00]: ....Done
```

关键信息是 `PersistRootStorage is now APP:Klei//DoNotStarveTogether/Cluster_1/Master/`，其中 `APP:Klei` 通常就是 Windows 系统的“文档”目录下的 Klei 文件夹。而在 Linux 系统上，这个路径会更加清晰。找到存档位置以后，就能自由编辑存档的内容了。

## 如何找到 Klei ID

直接打开：[https://accounts.klei.com/account/info](https://accounts.klei.com/account/info)

通过 steam 账号登陆，找到 KU 开头的就是。Klei ID 的作用，是在使用 dedicated server 的时候将自己配置成管理员，这样就可以通过一些命令行来操作服务器了。

## 如何找到服务器的 token

直接打开：[https://accounts.klei.com/account/game/servers?game=DontStarveTogether](https://accounts.klei.com/account/game/servers?game=DontStarveTogether)

通过 steam 账号登录，然后根据页面提示添加一个新服务器，找到 pds 开头的那个就是。服务器 token 的作用，是添加到存档里的 `cluster_token` 文件里，用来运行 dedicated server。

## 如何添加服务器 mod

首先打开 Don't Starve Together Dedicated Server 的安装目录。

然后打开 mods/dedicated_server_mods_setup.lua 文件。这个文件里面 -- 开头的行都是注释，这些注释解释了支持的两个函数的作用

- ServerModSetup：启动时加载单个 mod
- ServerModCollectionSetup：启动时加载 mod 合集

如果不在这里添加调用，那么即便在存档里配置了 mod，在启动 dedicated server 时也不会下载和加载对应的 mod，也就自然没有作用。一个修改后的 dedicated_server_mods_setup.lua 文件大致如下。

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

ServerModSetup("2287303119")
ServerModSetup("362175979")
ServerModSetup("378160973")
```

我们只需要找到 mod 的 id，调用 ServerModSetup 函数即可。mod 的 id 即是创意工坊中浏览该 mod 时 url 中末尾的一串数字。

## 如何通过指令操作服务器存档

如果成功配置了管理员，那么管理员就可以通过游戏内的指令才操作存档，具体方法可以自行搜索。

但是如果因为某些原因配置管理员失败，那么就可以通过 steam 启动 Don't Starve Together Dedicated Server 时候弹出的那个 terminal 来输入指令，具体的指令和游戏内指令的写法一摸一样，例如我想手动存档的时候，输入 `c_save()` ，回车即可执行。一些常用的指令整理如下：

- 存档

```markup
c_save()
```

- 回滚（默认回滚到上一个存档点，可以传入一个 int 来指定回到第前 n 个存档点）

```markup
c_rollback()
```

- 关闭服务器（不传参数或者传 true 会在退出前存档，传 false 不会存档）

```markup
c_shutdown()
```

## Klei ID 不生效怎么办

某些情况下，你在存档里面被识别到的 id 会与 Klei ID 不同，比如创建世界的时候选择了“离线”模式。

这种时候，通过 Klei ID 设置的管理员、黑白名单都不会生效。而想要找到此时自己真正的 id，需要关注自己连接服务器时，dedicated server 的 terminal 里面打印出来的日志：

```markup
[00:00:33]: New incoming connection 192.168.60.1|64659 <6998879110428916125>
[00:00:33]: Client connected from [LAN] 192.168.60.1|64659 <6998879110428916125>
[00:00:33]: Client authenticated: (OU_76xxxxxxxxxx) xxxxx
[00:00:52]: [Join Announcement] xxxxx
[00:01:00]: Resuming user: session/5E9063427451CE33/C7GSC53084I80HOGI64HG46/0000000128
[00:01:00]: Spawning player at: [Load] (4.34, 0.00, -273.72)
[00:01:00]: Sim unpaused
```

其中 `Client authenticated: (OU_76xxxxxxxxxx) xxxxx` 这一行日志中 OU 开头的这个就是离线世界里的 id。也正是因为这个 id 会发生变化，所以如果把一个存档从离线改为在线，登录的时候会再次出现选人界面，就好像是第一次进入这个世界一样。
