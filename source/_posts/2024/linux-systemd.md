---
title: Linux 系统管理：深入探索 systemd
excerpt: 在现代 Linux 系统中，systemd 已经成为系统和服务管理的新标准。systemd 提供了一系列创新功能，旨在提高系统的效率和可靠性。无论你是系统管理员还是对 Linux 内核感兴趣的用户，了解 systemd 都是必不可少的。
index_img: /img/2024/systemd.png
date: 2024-11-01 09:40:00
tags:
  - Linux
  - kernel
  - systemd
categories:
  - Linux
---

# Linux 系统管理的历史

Linux 系统管理的历史可以追溯到 Unix 时代，随着 Linux 操作系统的发展而演进。

## sysvinit

在 Unix 系统中，系统启动和管理最初是通过一系列脚本和程序手动完成的。这些脚本通常位于 /etc/rc 目录下，并且在系统启动时按顺序执行。
SysVinit：随着时间的推移，System V Init（简称 SysVinit）成为了 Unix 系统中最广泛使用的初始化系统。它定义了运行级别（runlevels），系统在这些级别之间切换以改变系统状态（例如，单用户模式、多用户模式等）。SysVinit 通过一系列的脚本和链接来管理服务的启动和停止。

[SysVinit - ArchLinux](https://wiki.archlinuxcn.org/wiki/SysVinit)

## upstart 事件驱动

当 Linux 内核首次发布时，它采用了与 Unix 类似的系统管理方法。Linux 的早期发行版，如 Slackware 和 Debian，都采用了 SysVinit 作为它们的初始化系统。Upstart：在 2000 年代初，Ubuntu 开发了 Upstart 框架，旨在解决 SysVinit 的一些局限性，如服务的并行启动和更复杂的服务依赖关系。Upstart 引入了事件驱动的启动和停止机制。

[浅析 Linux 初始化 init 系统: UpStart](https://linux.cn/article-4423-1.html)
[ReplacementInit - Ubuntu](https://wiki.ubuntu.com/ReplacementInit)

## systemd 的诞生

2010 年，Red Hat 的开发者 Lennart Poettering 开发了 systemd，旨在解决 SysVinit 和 Upstart 的不足。systemd 不仅是一个初始化系统，它还是一个全面的系统和服务管理框架。它提供了并行启动、更强的依赖管理、日志记录、网络管理等功能。

systemd 是一个 Linux 系统基础组件的集合，提供了一个系统和服务管理器，运行为 PID 1 并负责启动其它程序。功能包括：支持并行化任务；同时采用 socket 式与 D-Bus 总线式启用服务；按需启动守护进程（daemon）；利用 Linux 的 cgroups 监视进程；支持快照和系统恢复；维护挂载点和自动挂载点；各服务间基于依赖关系进行精密控制。systemd 支持 SysV 和 LSB 初始脚本，可以替代 sysvinit。除此之外，功能还包括日志进程、控制基础系统配置，维护登录用户列表以及系统账户、运行时目录和设置，可以运行容器和虚拟机，可以简单的管理网络配置、网络时间同步、日志转发和名称解析等。

[systemd 项目主页](https://systemd.io/)
[systemd source code](https://github.com/systemd/systemd)

---

# Linux 启动流程

提到 systemd 不得不提一下 Linux 的启动流程，这样才能清楚 systemd 在 Linux 系统中的地位和作用。

Linux 从按下电源键到进入用户交互界面整个启动流程大致可以分为四个阶段：

1. BIOS/UEFI 阶段
2. 引导程序（Bootloader）阶段
3. 内核（Kernel）初始化阶段
4. 初始化（Init/Systemd）阶段

## 1.BIOS/UEFI 阶段

当按下计算机的电源键后，BIOS/UEFI 阶段是整个系统启动的第一步。在这个阶段，硬件和固件开始协作，为加载操作系统做准备。

按下电源键后，电源开始供电，电源提供稳定的电压给主板、CPU、内存、硬盘等硬件。电源良好的信号（Power Good Signal）发送给主板，通知硬件已准备好运行。
紧接着 CPU 的程序计数器被初始化为一个特定的內存地址，存储在只读存储器（ROM）中的 BIOS 就是从这个特定的內存地址开始执行。所以没有 CPU 是无法启动主板上的 BIOS 的。

BIOS 或 UEFI 开始执行初始化程序（上电自检），并根据引导设备的优先级将系统控制权交给硬件启动项（比如硬盘/网络/U 盘等）。也就是我们 BIOS 上的启动菜单，这一步是可以被打断的。当我们按下 F12 或者 ESC 键（根据主板芯片组而异）就会弹出选择启动项的界面，而且这些按键高度依赖硬件。

BIOS 选择好硬件启动项之后就开始执行硬件设备上的初级引导程序代码，对于 MBR 硬盘来讲是最开始的一个扇区（512 字节）將被加载到內存，並执行行其中的初始化代码来加载下一阶段的 Bootloader 。

> MBR 主引导记录是一个 512 字节的扇区，位于硬盘的第一扇区（0 道 0 柱 1 扇区）。
> GPT 表则定位 EFI 系统分区（ESP），在 ESP 分区中查找引导管理器文件（如 `BOOTx64.EFI`）。

可以使用 `dd` 命令读取 MBR 里的内容 `dd if=/dev/sda of=mbr.bin bs=512 count=1`

使用 `od` 命令来查看 `od -xa mbr.bin`

```shell
bj-soho-test-47 ➜  ~ od -xa mbr.bin
0000000    0000    0000    0000    0000    0000    0000    0000    0000
        nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
*
0000660    0000    0000    0000    0000    21c1    f4d1    0000    0000
        nul nul nul nul nul nul nul nul   A   !   Q   t nul nul nul nul
0000700    0002    feee    ffff    0001    0000    6daf    7470    0000
        stx nul   n   ~ del del soh nul nul nul   /   m   p   t nul nul
0000720    0000    0000    0000    0000    0000    0000    0000    0000
        nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
*
0000760    0000    0000    0000    0000    0000    0000    0000    aa55
        nul nul nul nul nul nul nul nul nul nul nul nul nul nul   U   *
0001000
bj-soho-test-47 ➜  ~
```

## 2.引导程序（Bootloader）阶段

如果系统配置了多个内核版本或操作系统，引导程序会显示一个菜单，供用户选择要启动的系统，包括：启动 Linux 内核的不同版本、启动其他操作系统（如 Windows）、救援模式、单用户模式或恢复选项。

Bootloader 读取其配置文件，通常包括内核文件的位置和启动参数，在 BIOS 模式下：`/boot/grub/grub.cfg`，在 UEFI 模式下：`/boot/efi/EFI/BOOT/grub.cfg`。

引导程序根据 GRUB 的配置将默认内核镜像和 `initrd` 镜像加载到内存后，传递启动参数到内核，描述挂载点、根文件系统位置等，即将控制权移交给内核。

## 3.内核（Kernel）初始化阶段

## 4.初始化（Init/Systemd）阶段

---

# systemd

在现代 Linux 系统中，systemd 已经成为系统和服务管理的新标准。systemd 提供了一系列创新功能，旨在提高系统的效率和可靠性。无论是系统管理员还是对 Linux 内核感兴趣的用户，了解 systemd 都是必不可少的。

## systemd 的核心概念

### 单一系统初始化 (init)

在传统的 Unix 系统中，系统初始化是通过 init 进程来完成的。init 进程负责启动和管理系统的各个服务。然而，init 进程存在一些限制，例如启动顺序和依赖关系的管理不够灵活。

### systemd 的目标

systemd 的目标是替代传统的 init 进程，提供更强大的系统和服务管理功能。systemd 的核心概念包括：

1. **目标 (Target)**：systemd 使用目标来定义系统运行的不同状态。例如，multi-user.target 表示多用户模式，graphical.target 表示图形界面模式。系统管理员可以根据需要启动不同的目标，以满足不同的需求。
2. **服务 (Service)**：systemd 使用服务来管理系统的各个组件。服务可以是一个后台进程，也可以是一个守护进程。systemd 可以自动启动、停止和重启服务，并管理服务的依赖关系。
3. **单元 (Unit)**：systemd 使用单元来描述服务的配置信息。单元文件通常以 .service 结尾，定义了服务的启动参数、依赖关系和其他配置信息。
4. **挂载点 (Mount Point)**：systemd 可以自动挂载和卸载文件系统。挂载点文件通常以 .mount 结尾，定义了文件系统的挂载参数和依赖关系。
5. **设备 (Device)**：systemd 可以自动管理设备的加载和卸载。设备文件通常以 .device 结尾，定义了设备的配置信息和依赖关系。
6. **路径 (Path)**：systemd 可以监控文件系统的变化，并在文件系统发生变化时执行相应的操作。路径文件通常以 .path 结尾，定义了要监控的文件路径和依赖关系。
7. **套接字 (Socket)**：systemd 可以管理网络套接字。套接字文件通常以 .socket 结尾，定义了套接字的配置信息和依赖关系。
8. **定时器 (Timer)**：systemd 可以管理定时任务。定时器文件通常以 .timer 结尾，定义了定时任务的配置信息和依赖关系。
9. **快照 (Snapshot)**：systemd 可以创建系统的快照，以便在需要时恢复系统到之前的状态。快照文件通常以 .snapshot 结尾，定义了快照的配置信息和依赖关系。
10. **容器 (Container)**：systemd 可以管理容器。容器文件通常以 .scope 结尾，定义了容器的配置信息和依赖关系。
11. **虚拟机 (VM)**：systemd 可以管理虚拟机。虚拟机文件通常以 .slice 结尾，定义了虚拟机的配置信息和依赖关系。

## systemd 的安装和配置

### 安装 systemd

在大多数 Linux 发行版中，systemd 已经默认安装。如果没有安装，可以使用以下命令进行安装：

```bash
sudo apt-get install systemd
```

### 配置 systemd

systemd 的配置文件通常位于 /etc/systemd/ 目录下。其中，最重要的配置文件是 systemd.conf 和 systemd-journald.conf。

systemd.conf 文件定义了 systemd 的全局配置参数，例如日志级别、默认目标等。systemd-journald.conf 文件定义了 systemd-journald 的配置参数，例如日志文件的位置和大小等。

除了全局配置文件，每个服务都有自己的配置文件。这些配置文件通常以 .service 结尾，位于 /etc/systemd/system/ 目录下。例如，nginx 服务的配置文件是 /etc/systemd/system/nginx.service。

要修改服务的配置，只需编辑相应的配置文件即可。修改完成后，需要重新加载 systemd 的配置，并重新启动服务：

