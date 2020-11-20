---
layout: post
title: Change CapsLock to Ctrl in Gnome3
date: 2020-11-04 11:33
category:
author:
tags: [linux]
summary:
---

我使用了 xbindkeys 和 xmodkey 两个方式设置键位映射，最终都没有成功。
感觉应该是**Gnome 接管了 Keyboard 的设置**

上网查到了用`gnome-tweak-tool`是可以设置 Ctrl 位置的。

[using gnome tweak tool](https://askubuntu.com/a/33792)

> **13.10+**:
>
> Install and use gnome-tweak-tool > Keyboard & Mouse > Keyboard > Additional Layout Options > Caps Lock behavior.
>
> **Pre 13.10**:
>
> Open the Keyboard Preferences dialog (System -> Preferences -> Keyboard). On the layout tab, click the Options... button. Expand the Ctrl key position section and select Swap Ctrl and Caps Lock.
>
> Those settings should be applied each time you log in, and will only affect your user account.
