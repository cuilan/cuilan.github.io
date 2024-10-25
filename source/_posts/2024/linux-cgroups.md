---
title: Linux-cgroups
excerpt: Linux资源管理之cgroups简介
index_img: /img/2023/git.png
date: 2024-10-25 11:06:00
tags:
  - Linux
  - Docker
  - Containerd
categories:
  - 容器化
---



识别 Linux 节点上的 cgroup 版本 
cgroup 版本取决于正在使用的 Linux 发行版和操作系统上配置的默认 cgroup 版本。 要检查你的发行版使用的是哪个 cgroup 版本，请在该节点上运行 stat -fc %T /sys/fs/cgroup/ 命令：

stat -fc %T /sys/fs/cgroup/
对于 cgroup v2，输出为 cgroup2fs。

对于 cgroup v1，输出为 tmpfs。

