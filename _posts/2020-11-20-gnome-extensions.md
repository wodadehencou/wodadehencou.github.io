---
layout: post
title: My Manjaro Extension Configuration
date: 2020-11-20 08:57
category:
author:
tags: [linux, gnome, manjaro]
summary:
---

`Manjaro Linux`是基于`Arch`的发行版，近来属于非常流行的一个发行版本。
在使用过程中，走过一些弯路，记下来，下次不要再犯。

Linux 桌面环境，最好用的还是`Gnome`，我选择了最新版本的`Gnome`。

## Gnome Extension

首先，`Manjaro`默认会 build in 一些 extensions，实际使用下来，这些默认的要比从 extension 网站上安装的兼容性更好，也更好用。

**特别注意：\***建议不要 update 默认的扩展，从 gnome extension 网站上升级一些扩展之后，反而不好用了。特别需要注意：如果系统在 suspend 或 sleep 之后重新进入，发现之前已经存在的 session 自动 logout 了，其实是由于一些不兼容的 extension 导致的。具体哪个我不清楚，但是删除了没用的 extension，删除了升级的，这个问题就不存在了。\*

### 默认推荐

1. `Dash to Dock`
   把原来在左边的启动栏移到了下方，并且可以在桌面上常驻。用下来感觉比`Dash to Panel`好用。

2. `Unite`
   用来显示 system tray，用来把窗口标题栏和 gnome 的 top panel 合并。之前用了`Top Icon Plus`和`Pixel Saver`来实现这两个功能，用下来都不如`Unite`稳定，系统集成的还是靠谱。

3. `Pop Shell`
   这个目前还在初始版本，但是功能非常强大，有些类似 Mac 上的`Alfred`，还能完全通过键盘切换窗口。启动之后会强制覆盖一些系统快捷键，还不能修改自己的快捷键设置。我还在适应这个扩展。

### 新装

1. `Bing Wallpaper Changer`
   每次安装 Gnome 必备的应用。每天自动更新桌面。

2. `ibus font setting`
   输入法我用的是`rime-四叶草`，gnome 和 ibus 集成比较好，我使用了`ibus-rime`。Gnome 貌似接管了 ibus 的候选字体设置，这个扩展可以选择 ibus 候选字体。

3. `Vitals`
   Top Panel 上的系统状态监控，用过几个，这个显示效果最好，比较稳定。推荐。
