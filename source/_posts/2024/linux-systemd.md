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
紧接着 CPU 的程序计数器被初始化为一个特定的內存地址，这个地址在 x86 架构中通常是 `0xFFFFFFF0`，存储在只读存储器（ROM）中的 BIOS 就是从这个特定的內存地址开始执行。所以没有 CPU 是无法启动主板上的 BIOS 的。

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

#### 关于 `aa55`

> `0xAA55`（在小端序下显示为 `aa55`）。它是 MBR 的有效标志位（Magic Number）。如果 BIOS 扫描完扇区没发现这最后两个字节是 55 AA，它会认为这个设备不可引导，直接跳到下一个硬件。

## 2.引导程序（Bootloader）阶段

如果系统配置了多个内核版本或操作系统，引导程序会显示一个菜单，供用户选择要启动的系统，包括：启动 Linux 内核的不同版本、启动其他操作系统（如 Windows）、救援模式、单用户模式或恢复选项。

Bootloader 读取其配置文件，通常包括内核文件的位置和启动参数。
- 在 BIOS 模式下：`/boot/grub/grub.cfg`
- 在 UEFI 模式下：`/boot/efi/EFI/BOOT/grub.cfg`。

引导程序根据 GRUB 的配置将默认内核镜像和 `initrd` 镜像加载到内存后，传递启动参数到内核，描述挂载点、根文件系统位置等，即将控制权移交给内核。

- **加载内核映像**：启动加载程序将压缩的内核映像（如vmlinuz）从硬盘加载到内存中。内核映像通常是一个gzip或其他格式压缩的二进制文件。
- **加载 `initrd`/`initramfs`**：如果使用initrd（初始RAM盘）或initramfs（初始RAM文件系统），启动加载程序也会将这些文件加载到内存中，以便内核在启动时使用。
- **解压内核**：内核映像被加载到内存后，解压缩程序会运行并将压缩的内核映像解压到适当的内存位置。
- **跳转到解压后的内核**：一旦解压完成，控制权会被移交给解压后的内核代码的入口点。

## 3.内核（Kernel）初始化阶段

当引导程序（Bootloader）将控制权移交给内核入口点后，Linux 内核开始接管整个硬件系统。虽然内核的内部机制极其复杂，但从启动流程的角度来看，它主要完成了三件事：**自我解压**、**驱动加载** 与 **挂载根分区**。

### 3.1 自解压与硬件探测

你看到的内核文件通常叫 `vmlinuz`，末尾的 **'z'** 说明它是压缩格式。内核启动的第一步是执行头部的解压代码，将自身载入内存。随后，内核开始扫描 CPU、内存、中断控制器等核心硬件，并初始化基础的内存管理逻辑。

### 3.2 initramfs：挂载磁盘
这是一个经常被忽略但至关重要的步骤。
由于现代内核不可能内置所有硬件（如 NVMe、RAID、加密分区）的驱动，它会先解压一个随身携带的“小包”—— **initramfs**（Initial RAM Filesystem）。这是一个存在于内存中的临时根文件系统，里面包含了挂载真正硬盘所需的驱动模块。

### 3.3 切换根目录
一旦 initramfs 协助内核识别到了真正的硬盘，内核就会执行 **切换根目录** 的操作：

1. **挂载根分区**： 将真正的硬盘分区（Root FS）挂载到系统的 `/` 目录。

2. **启动初始化进程**： 内核完成底层初始化后，会寻找硬盘上的第一个可执行文件。在现代 Linux 中，这个文件通常位于 `/sbin/init`。

此时，内核完成了它的使命，它通过执行 `exec` 调用启动了系统中的第一个用户态进程：`systemd`。

## 4.初始化（Init/Systemd）阶段

当内核完成所有底层硬件和文件系统的挂载后，启动磁盘上的第一个程序：`/sbin/init`，通常就是 `systemd`。

## 4.1 诞生 PID 1
`systemd` 是内核启动的第一个用户态进程，因此它的进程 ID（PID）永远是 **1**。作为所有后续进程的“始祖”，它承载了管理整个用户空间生命周期的重任。如果 PID 1 崩溃，内核会立即触发 **Kernel Panic**，导致整个系统挂起。

## 4.2 从“串行”到“并行”

传统的 SysVinit 采用脚本串行启动方式（A 启动完再启动 B），导致启动时间冗长且依赖关系难以维护。systemd 引入了**有向无环图 (DAG)** 的设计理念：

- **并行化**： 只要服务之间没有硬性的先后依赖，`systemd` 会同时拉起成百上千个服务。
- **按需激活**： 配合 `Socket Activation` 技术，即使某个服务还没完全启动，`systemd` 也可以预先创建它的网络套接字。请求会被内核缓存，直到服务就绪后再进行处理。

### 4.3 达到目标状态 (Targets)

`systemd` 弃用了老旧的“运行级别 (Runlevels)”概念，转而使用 **Targets**（目标单元）。

- **multi-user.target**： 相当于传统的 Runlevel 3，系统进入多用户、仅命令行的稳定状态。
- **graphical.target**： 相当于 Runlevel 5，在命令行基础上启动显示管理器和桌面环境。

当 `systemd` 成功加载了目标 Target 及其依赖的所有 Unit（服务、挂载点、设备等）后，屏幕上会出现熟悉的欢迎界面。

---

# systemd

## 1. 从 Init 到 Systemd：变革

systemd 的出现，是将 Linux 系统管理从“执行脚本”进化到了“状态管理”。

## 2. 核心组件：Unit（单元）

在 systemd 的世界里，**万物皆单元 (Unit)**。单元是系统管理的基本对象，每种单元都由一个配置文件（Unit File）描述。

**基础管理单元**

* **Service (服务)**：最常用的单元，管理后台守护进程（如 nginx, sshd）。支持自动重启、环境变量配置及权限限制。
* **Target (目标)**：单元的逻辑组。它不代表具体的进程，而是一组状态的集合（如 multi-user.target）。它取代了传统的运行级别（Runlevels）。
* **Mount (挂载点)**：管理文件系统挂载。systemd 会监控 /etc/fstab 并自动生成对应的 .mount 单元，确保挂载点与服务之间的依赖关系正确。
* **Automount (自动挂载)**：提供按需挂载功能。只有当路径被访问时，才会触发真正的挂载操作。

**资源与隔离单元**

* **Slice (切片)**：用于 **资源管理**。它将系统划分为不同的层级（如 `user.slice`, `system.slice`），通过 Cgroups 限制该切片下所有进程的 CPU、内存占用。
* **Scope (范围)**：用于管理 **外部进程**。如果一个进程不是由 systemd 直接启动的（例如用户通过 SSH 登录后的会话，或 Docker 启动的容器进程），systemd 会通过 `.scope` 单元将其纳入监控，确保其资源消耗可控。

> **注意**： 这里的 `.slice` 和 `.scope` 是 systemd 与 **Cgroups** 深度集成的体现，而非传统的虚拟机文件。

**触发与监控单元**

* **Socket (套接字)**：实现“延迟启动”的核心。systemd 预先监听端口，当流量到达时才启动对应的 Service。
* **Timer (定时器)**：现代化的 `cron` 替代品。支持毫秒级精度和基于日历或开机时间的灵活触发。
* **Path (路径)**：监控文件或目录的变化。一旦指定路径发生变动（如配置文件被修改），立即触发关联的服务。
* **Device (设备)**：将内核识别到的硬件抽象为单元。允许服务在特定硬件（如网卡、硬盘）就绪后才启动。

**状态管理单元**

* **Snapshot (快照)**：（注：在最新版本的 systemd 中已不推荐使用，建议弱化描述）。它主要用于保存当前所有单元的状态，以便在执行复杂操作后快速回滚到之前的运行状态。

## 3. Unit 文件结构规范

systemd 单元文件采用类似 Windows INI 文件的文本格式。一个标准的 `.service` 文件通常由三个核心区块组成：`[Unit]`、`[Service]` 和 `[Install]`

### 1. [Unit] 区块：元数据与依赖关系

这个区块主要定义单元的描述信息以及它与其他单元的启动顺序依赖。
* **Description**：简短的服务描述，显示在 `systemctl status` 中。
* **Documentation**：文档地址（URL 或本地路径）。
* **After**：定义启动顺序。例如 `After=network.target` 表示在网络就绪后再启动本服务。
* **Requires**：强依赖。如果依赖的单元启动失败，本单元也会失败。
* **Wants**：弱依赖。即便依赖的单元启动失败，本单元仍尝试启动（推荐用法）。

### 2. [Service] 区块：执行逻辑

这是 `.service` 文件特有的区块，定义了如何运行、管理和监控进程。

* **Type**：启动类型。
  * `simple`（默认）：直接运行主进程。
  * `forking`：主进程会派生子进程，父进程退出（传统守护进程模式）。
  * `notify`：进程启动后发送信号给 systemd（最可靠，配合 `sd_notify` 使用）。
* **ExecStart**：启动服务时执行的完整命令（必须使用绝对路径）。
* **ExecReload**：重启服务时的命令。
* **Restart**：配置自动重启策略（如 `on-failure`）。
* **User / Group**：指定运行服务的用户/组，遵循最小权限原则。
* **WorkingDirectory**：进程运行的根目录。

### 3. [Install] 区块：启用设置

定义了 `systemctl enable` 时的行为，即建立软链接的规则。

* **WantedBy**：指定该服务属于哪个 Target。最常用的是 `multi-user.target`。
* **Alias**：服务的别名。

## 4. 单元文件的存放位置与优先级

systemd 的强大之处在于其分层配置机制，systemd 按照以下顺序搜索 Unit 文件，优先级由高到低：

| 优先级   | 存放路径             | 描述                 | 适用场景                               |
| -------- | -------------------- | -------------------- | -------------------------------------- |
| 1 (最高) | `/etc/systemd/system/` | 管理员手动创建或修改 | 自定义服务或对系统服务的 override      |
| 2        | `/run/systemd/system/` | 运行时生成的单元     | 临时生成的服务，重启后失效（内存存储） |
| 3 (最低) | `/lib/systemd/system/` | 发行版或软件包自带   | 软件安装时自动放置的原始配置           |

## 5. systemd 命令行工具集

### 5.1 systemctl：核心控制台

`systemctl` 是最常用的工具，用于检查和控制 systemd 系统及服务的状态。

* **服务管理基础**：
  * `systemctl start <unit>`：立即启动。
  * `systemctl stop <unit>`：立即停止。
  * `systemctl restart <unit>`：重启（先关再开）。
  * `systemctl reload <unit>`：重新加载配置文件而不重启进程。
  * `systemctl status <unit>`：查看服务的运行状态、实时日志流及 Cgroup 信息。
* **开机自启管理**：
  * `systemctl enable <unit>`：设置开机自启（在 /etc/systemd/system/ 下创建软链接）。
  * `systemctl disable <unit>`：取消开机自启。
  * `systemctl is-enabled <unit>`：检查是否已设为自启。
* **系统状态概览**：
  * `systemctl list-units`：列出当前内存中活跃的单元。
  * `systemctl list-unit-files`：列出磁盘上所有的单元文件及其状态。
  * `systemctl list-dependencies <unit>`：**神器**。以树状结构列出某个服务的所有依赖项，帮你理清启动链路。

### 5.2 journalctl：统一日志管理器

systemd 引入了二进制日志系统 **journald**，彻底改变了以往到处找 `/var/log/*.log` 的混乱局面。

* **高效查询**：
  * `journalctl -u <unit>`：只看某个服务的日志。
  * `journalctl -f`：类似 tail -f，实时滚动显示最新日志。
  * `journalctl -xe`：显示最近的报错日志，并附带系统解释（排障首选）。
  * `journalctl --since "2026-03-20 18:00:00"`：按时间段过滤日志。
* **容量控制**：
  * `journalctl --disk-usage`：查看日志占用了多少硬盘空间。
  * `journalctl --vacuum-size=500M`：强制清理，只保留 500MB 日志。

### 5.3 systemd-analyze：启动性能“体检”

* **启动耗时**： `systemd-analyze`。直接显示内核和用户态启动的总耗时。
* **寻找“元凶”**： `systemd-analyze blame`。按启动耗时降序排列所有单元，一眼看出是谁拖慢了你的服务器。
* **可视化分析**： `systemd-analyze plot > boot.svg`。生成一张精美的矢量图，直观展示每个服务的启动时序和重叠情况。

### 5.4 辅助配置工具（三剑客）

systemd 还接管了一些系统基础配置，提供了更统一的命令：

* **hostnamectl**：管理主机名。不再需要手动修改 `/etc/hostname`。
  * 例：`hostnamectl set-hostname debian-k3s-node1`
* **localectl**：管理语言设置和键盘布局。
* **timedatectl**：管理时间和时区。
  * 例：`timedatectl set-timezone Asia/Shanghai`
