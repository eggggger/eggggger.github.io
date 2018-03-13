---
title: 【译】你应该要知道的 Linux 知识 - Julia Evans
date: 2016-12-12 14:40:31
tags: 
  - Linux
category: 写码 
---

我在 Twitter 上问了网友们更有兴趣了解 Linux 的什么东西。其中有很多回复非常不错，这里整理出了一个列表（其中有许多是可以在任何 Unixy 系统上讨论的，另一些则特定于 Linux）

<!--more-->
* tcp/ip & 网络相关

* 什么是 port/socket?

*   seccomp (Linux 内核提供的一种沙盒机制)

*   systemd (系统服务管理器，启动守护进程等)

* IPC (进程间通信，管道)

* r, w, x (文件权限)，SetUid, SetGid, Sticky Bit (文件特殊权限), chown 如何工作？

* shell 如何使用 fork & exec？

* 怎么让我的电脑变成路由器？

* process groups (进程组), session leaders (会话领导进程), shell job control (工作管理)

* 内存分配, 堆的工作原理, malloc 做了什么？

* ttys，终端是如何工作的？

* process scheduling (进程调度)

* drivers (驱动程序)

* Linux 和 Unix 的区别？

* kernel (内核)

* 现代 X (窗口系统协议) 服务器？

* X11 (X 协议 11 版本) 如何工作？

*   Linux’s zero-copy (零拷贝) API (sendfile, splice, tee)

*  dmesg (显示内核缓存区信息) 还能做什么？

* 内核模块如何工作？

* 嵌入式相关: realtime (实时)namespaces (环境隔离)， cgroups (进程组群)，docker (容器)， SELinux (安全增强 Linux)，AppArmor (内核安全模块，应用程序访问控制)

*   debuggers (调试器)

*   线程和进程的区别？

*   如果 Unix 是基于命令行的，桌面环境如 GNOME 是怎么实现的？

*   man 是如何工作的？

*   kpatch (内核热补丁技术 by Red Hat), kgraph (内核热补丁技术 by SUSE), kexec (内核热切换)

*   栈相关的： C 的变量实际上是 stack slot (栈槽)？tf (trap flag ?) 怎么做 setjmp
    和 longjmp 的工作?

*   package management (包管理器)

*   mounts and vfs (挂载和虚拟文件系统)

有很多理由表明这些知识点都很不错：

1. 这个月我要画至少 11 张关于 Linux 的图，它们都是不错的点子。

2. 这个列表里的有些东西我都不知道是什么 (dbus, SELinux) ，要不就只是听到过一点 (seccomp， X11 如何工作，以及其他更多)，它提醒我还有许多有趣的东西需要去学习。

3. 它让我知道自己掌握到了什么程度 — 虽然在不查资料的情况下我不能解释清楚其中的很多东西，但我至少知道从哪里去了解它们。

此外，我想提醒你们可以在网上写一些有趣的博客/图，例如 “dmesg 能做什么”就是一个不错的主题，你也很有可能因此去学习了解它 (我在维基百科上阅读了[dmesg](https://en.wikipedia.org/wiki/Dmesg)，现在，我比以前了解的更深入了)

> 本文转载自：[众成翻译](http://www.zcfy.cc)
> 译者：[eggggger](http://www.zcfy.cc/@eggggger)
> 链接：[http://www.zcfy.cc/article/1990](http://www.zcfy.cc/article/1990)
> 原文：[https://jvns.ca/blog/2016/11/21/things-to-learn-about-linux/](https://jvns.ca/blog/2016/11/21/things-to-learn-about-linux/)
