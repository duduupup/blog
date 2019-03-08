---
title: CentOS7界面卡死的解决方法
date: 2019-03-08 14:08:33
categories:
  - Linux
tags:
  - CentOS
  - Tools
---
CentOS7的gnome界面经常会卡死,可以ssh(current_user@host)连接进入shell,通过如下命令令其复活:
```
pkill -9 gnome-shell
```
由于gnome界面经常卡死,可以将该命令写入到用户的~/.bashrc文件中,如下所示:
```
alias gnome-relive="pkill -9 gnome-shell"
```
这样就可以使用gnome-relive命令令gnome界面复活了