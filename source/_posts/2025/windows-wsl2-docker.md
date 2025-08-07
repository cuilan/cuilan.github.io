---
title: 在 Windows 环境下优雅的使用 Docker
excerpt: 拒绝 Docker Desktop 从我做起，无论是 Mac 还是 Windows，Docker 官方的 Docker Desktop 容器管理软件都是垃圾。在 Mac 上推荐使用：lima。
index_img: /img/2025/win-wsl-docker.png
date: 2025-08-06 17:18:00
tags:
  - Windows
  - Docker
  - WSL2
  - PowerShell
categories:
  - 容器化
---

# 一、先说期望

对于我一个每天重度依赖 Linux 开发环境的开发者来说，Docker 或者说容器化需求几乎是刚需。

Mac 虽然基本满足大部分需求，但对于容器化来说依然不足，尤其是 Docker Desktop for Mac，因此在 Mac 下使用 Docker 有以下几种方案：
1. 安装 `docker cli`，通过 `docker context` 使用远程 `Linux` 主机上的 `docker`。
2. 安装 `VMware` 或 `Parallels` 等虚拟机，甚至自己封装 `qemu` 脚本，配置复制且性能开销大。
3. 在 Mac 下更 **推荐使用 CNCF 生态的 `lima`**，可以在 Apple silicon 系列芯片上获得完美的 x86 开发体验。

Windows 环境下一直以来都存在一些痛点。不过自微软推出 **WSL2 (Windows Subsystem for Linux 2)** 以来，Windows 的开发体验得到了极大的改善。我们现在可以在 Windows 上获得一个近乎原生的 Linux 环境。

然而，在容器化方面，官方的 Docker Desktop for Windows 同样不如人意：
1.  **资源占用高**：启动后会占用大量内存和 CPU 资源。
2.  **收费策略**：对于大公司，Docker Desktop 不再是免费的。
3.  **不够“原生”**：它背后依然是虚拟机技术，有时会出现一些奇怪的网络或文件系统问题。

所以，我们的期望是：
-   彻底告别 Docker Desktop。
-   在 Windows 环境下，通过原生命令行工具（PowerShell/CMD）使用完整的 `docker cli` 功能。
-   能够与 VS Code 完美集成，特别是使用 Dev Containers 功能，享受容器化开发带来的便利。

# 二、方案选型

要在 Windows 上运行 Docker (Linux 容器)，本质上都需要一个 Linux 内核环境。目前主流的方案有两种：

## 传统虚拟机 (VM)

使用 VirtualBox、VMware 等传统虚拟机软件安装一个完整的 Linux 发行版，然后在其中安装和运行 Docker。

-   **优点**：环境隔离性好，与宿主机完全隔离。
-   **缺点**：
    -   资源占用巨大，启动和运行都比较笨重。
    -   与 Windows 的集成度差，文件共享、网络访问等配置繁琐。
    -   启动速度慢，无法做到即用即走。

## WSL2

WSL2 是微软官方提供的方案，它使用轻量级虚拟化技术，在 Windows 上提供了一个完整的 Linux 内核。

-   **优点**：
    -   性能接近原生 Linux，启动速度极快。
    -   资源占用相对较少。
    -   与 Windows 系统深度集成，可以无缝访问 Windows 文件系统，反之亦然。
    -   网络默认为 NAT 模式，WSL2 中的端口可以直接通过 `localhost` 访问，非常方便。
-   **缺点**：
    -   对于需要特定硬件直通等高级虚拟化功能的场景支持不足。

**结论**：对于开发场景，WSL2 无疑是最佳选择。它提供了最佳的性能和系统集成度，是实现我们“优雅使用 Docker”目标的基础。

# 三、实施步骤

## 1. 安装并配置 WSL2

首先，我们需要一个功能齐全的 WSL2 环境。

**开启虚拟机平台功能**

在“控制面板” -> “程序” -> “启用或关闭 Windows 功能”中，确保“虚拟机平台”和“适用于 Linux 的 Windows 子系统”已经勾选。

或者通过 PowerShell (管理员) 执行：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
执行后需要重启电脑。

**安装 WSL2**

现代的 Windows 10/11 已经可以非常方便地通过一条命令安装 WSL2 和默认的 Linux 发行版 (Ubuntu)。

```powershell
# 安装并升级 wsl2，需管理员运行。这将安装 WSL2 内核并默认安装 Ubuntu 发行版。
wsl --install

# 如果已经安装过，可以运行此命令更新 wsl2 内核至最新版
wsl --update
```

**管理 Linux 发行版**

以下是一些常用的 WSL 管理命令：

```powershell
# 查看在线可用的 Linux 发行版
wsl --list --online
# 简写
wsl -l -o

# 如果不想用默认的 Ubuntu，可以指定安装其他发行版
# wsl --install <DistroName>
# 例如：
wsl --install Debian

# 查看已安装的发行版的运行状态和 WSL 版本
wsl --list --verbose
# 简写
wsl -l -v

# 确保你的发行版运行在 WSL2 模式下。如果版本为 1，使用以下命令转换
# wsl --set-version <DistroName> 2

# 设置默认启动的 Linux 发行版
wsl --set-default <DistroName>

# 关闭正在运行的 Linux 发行版（相当于重启）
wsl --terminate <DistroName>

# 查看wsl版本、状态、默认发行版等信息
wsl --status

# 注销（删除）一个已安装的 Linux 发行版
wsl --unregister <DistroName>
```

**配置 `.wslconfig` (可选)**

你可以在你的 Windows 用户目录下 (例如 `C:\Users\YourUser`) 创建一个名为 `.wslconfig` 的文件来全局配置 WSL2 的行为，例如限制其最大内存占用。

```ini
[wsl2]
# 限制 WSL2 虚拟机最大内存为 8GB
memory=8GB
# 限制 CPU 核心数为 4
processors=4
# 关闭 localhostForwarding，当需要将 WSL 端口暴露给局域网时可能需要
# localhostForwarding=false
```
修改后需要重启 WSL (`wsl --shutdown`) 才能生效。

## 2. 在 WSL2 中安装 Docker Engine

现在我们进入 WSL2 的 Linux 环境中，安装原生的 Docker Engine。这里以 Ubuntu 为例。

首先，启动你的 WSL 终端：

```powershell
wsl
```

然后，在 Linux 环境中执行以下步骤：

**卸载旧版本**

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**设置 Docker 的 APT 仓库**

```bash
# 更新 apt 包索引并安装依赖
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 添加 Docker 官方 GPG 密钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 设置仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**安装 Docker Engine**

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**配置 Docker**

**1. 以非 root 用户运行 Docker**

为了避免每次都输入 `sudo`，需要将当前用户加入 `docker` 用户组。

```bash
sudo usermod -aG docker $USER
```
> **注意**：执行此命令后，你需要退出当前的 shell (`exit`) 然后重新进入 WSL，用户组权限才会生效。

**2. 配置 Docker Daemon 以接收远程连接**

这是实现 Windows 端 CLI 与 WSL 中 Docker Daemon 通信的关键一步。我们需要让 Docker 监听一个 TCP 端口。

修改 `/usr/lib/systemd/system/docker.service` 文件：
```bash
sudo vim /usr/lib/systemd/system/docker.service
```

在文件中添加以下内容。如果文件已有内容，请确保合并。这会让 Docker 同时监听默认的 Unix socket 和 2375 端口。

```
# vim /usr/lib/systemd/system/docker.service
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

**3. 启用并启动 Docker 服务**

新版本的 WSL 支持 `systemd`，我们可以像在普通 Linux 上一样管理服务。

```bash
# 重新加载
sudo systemctl daemon-reload
# 启动 Docker 服务
sudo systemctl restart docker
# 设置开机自启
sudo systemctl enable docker
# 检查 Docker 服务状态
sudo systemctl status docker
```

如果你的 WSL 版本较旧，不支持 `systemd`，你可能需要手动启动服务：

```bash
# 手动启动
sudo dockerd > /dev/null 2>&1 &
# 并将此命令加入到你的 .bashrc 或 .zshrc 文件末尾，以实现自动启动
# echo "sudo dockerd > /dev/null 2>&1 &" >> ~/.bashrc
```

至此，WSL2 中的 Docker 环境已经准备就绪。你可以通过 `docker version` 和 `docker run hello-world` 来验证。

## 3. 在 Windows 上配置 Docker CLI

回到 Windows，让 Windows 上的 Docker CLI 与 WSL 中的 Docker 服务交互。

**安装 Docker CLI**

我们可以使用包管理器如 `choco` 或 `scoop` 来单独安装 Docker CLI。

使用 `choco`:
```powershell
choco install docker-cli
```

**开发人员更推荐使用 `scoop`**，首先是安装:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

然后安装 `docker`:

```powershell
scoop install docker
```

安装完成后将获得两个可执行程序：`docker.exe` `dockerd.exe`，不过 dockerd.exe 仅可以运行 Windows container，对于我来说基本无用，有了 docker.exe 就足够了。

**配置 Docker 上下文 (Context)**

Docker CLI 提供了“上下文” (Context) 的概念来管理多个 Docker daemon。我们将创建一个指向 WSL2 中 Docker 服务的新上下文。

打开 PowerShell，执行：

```powershell
# WSL2 的网络是 NAT 模式，它的 2375 端口会自动映射到 Windows 的 localhost:2375
docker context create wsl --description "Docker in WSL2" --docker "host=tcp://localhost:2375"
```

**切换并使用新上下文**

```powershell
# 查看所有可用的上下文
docker context ls

# 切换到我们刚创建的 wsl 上下文
docker context use wsl

# 验证一下，这个 version 命令现在是由 Windows 的 docker.exe 发出，但由 WSL 中的 dockerd 执行并返回结果
docker version
```

现在，你可以在 Windows 的任何终端 (PowerShell, CMD) 中直接运行 `docker`、`docker-compose` 等命令，就像安装了 Docker Desktop 一样，但背后驱动它们的是我们轻量、高效的 WSL2 Docker Engine！

## 4. VS Code 集成 (Dev Containers)

这套方案的终极目标之一，就是流畅地使用 VS Code Dev Containers。

**安装 VS Code 扩展**

确保你已经安装了以下两个关键扩展：
1.  **Remote - WSL**: 让你直接在 VS Code 中打开 WSL 中的项目目录。
2.  **Dev Containers**: 提供容器化开发环境的核心功能。

**配置与使用**

因为我们已经在上一步中配置并切换了 Docker 上下文，VS Code 会自动使用这个配置。

1.  在 PowerShell 中，进入你的项目目录。
2.  输入 `code .` 启动 VS Code。
3.  如果你的项目中包含 `.devcontainer` 文件夹，VS Code 右下角会自动弹窗，询问你是否“在容器中重新打开 (Reopen in Container)”。
4.  点击确认，VS Code 就会调用 Docker CLI，根据 `.devcontainer/devcontainer.json` 的配置，在你的 WSL Docker Engine 中构建或拉取镜像，并启动开发容器。

整个过程无缝衔接，你既享受了 Windows 的桌面环境，又获得了 Linux 容器化开发的全部威力，同时还摆脱了臃肿的 Docker Desktop。

# 四、备份与恢复

WSL2 提供了方便的导入导出功能，可以用来备份你的整个 Linux 发行版（包括已安装的 Docker 和其他软件）。

```powershell
# 导出一个名为 ubuntu 的发行版到 D:\backup\ubuntu.tar
wsl --export ubuntu D:\backup\ubuntu.tar

# 从备份文件导入一个新的发行版，名为 my-ubuntu-backup，安装位置在 C:\wsl\my-ubuntu
wsl --import my-ubuntu-backup C:\wsl\my-ubuntu D:\backup\ubuntu.tar
```

# 五、总结

通过 `WSL2` + `原生 Docker Engine` 的组合，我们成功地在 Windows 上打造了一套轻量级、高性能且免费的 Docker 开发环境。这套方案不仅让我们摆脱了 Docker Desktop 的束缚，还通过 Docker Context 和 VS Code 扩展实现了与 Windows 生态的完美集成。希望这篇文章能帮助你在 Windows 上更优雅地进行容器化开发。