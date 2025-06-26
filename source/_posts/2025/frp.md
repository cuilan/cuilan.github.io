---
title: frp 内网穿透配置
excerpt: 详细介绍如何配置 frp 内网穿透服务，包括服务端和客户端的完整配置方法，以及使用 systemd 进行服务管理的最佳实践 
index_img: /img/2025/frp.png
date: 2025-06-26 15:07:00
tags:
  - Linux
  - network
  - 内网穿透
categories:
  - Linux
---

# 前提条件

在开始之前，请确保你已经：

在两台服务器上下载了对应操作系统和架构的 [frp 发布版本](https://github.com/fatedier/frp/releases)。

将 `frps` 和 `frps.toml` 文件放在公网服务器上。

将 `frpc` 和 `frpc.toml` 文件放在内网机房服务器上。

为了方便管理和符合 Linux 标准：

* 二进制文件路径: `/usr/local/bin/frps` 和 `/usr/local/bin/frpc`
* 配置文件目录: `/etc/frp/`
* 配置文件路径: `/etc/frp/frps.toml` 和 `/etc/frp/frpc.toml`

如果你的路径不同，请在下面的 systemd 配置文件中进行相应修改。

---

# 第一步：公网服务器 (frps) 配置

这台服务器拥有公网 IP，作为 frp 的服务端。

## 1. 创建配置文件 frps.toml

在公网服务器上，创建配置文件 `/etc/frp/frps.toml`，并填入以下内容：

```toml
# /etc/frp/frps.toml

# frps 服务端绑定的端口，用于接收 frpc 的连接
bindPort = 7000

# 身份验证令牌，frpc 必须使用相同的令牌才能连接成功
# !! 请务必修改为自己的复杂密码 !!
auth.token = "your_secure_token_here"

# 【可选】Web 服务绑定的 HTTP 端口。
# 当 frpc 配置了 type = http 的代理时，可通过 "http://公网IP:vhostHTTPPort" 访问
# 如果 80 端口未被占用，可以设置为 80
vhostHTTPPort = 8080

# 【可选】Web 服务绑定的 HTTPS 端口。
# 如果 443 端口未被占用，可以设置为 443
vhostHTTPSPort = 8443

# 【可选】Dashboard 仪表盘配置，方便通过浏览器查看 frp 状态
# 仪表盘访问端口
dashboardPort = 7500
# 仪表盘登录用户名
dashboardUser = "admin"
# 仪表盘登录密码 (!! 请修改 !!)
dashboardPassword = "your_dashboard_password"

# 【可选】日志配置
log.to = "/var/log/frps.log" # 日志文件路径
log.level = "info"          # 日志级别：trace, debug, info, warn, error
log.maxDays = 7             # 日志文件最长保留天数
log.disablePrintColor = true # 如果写入文件，建议禁用颜色输出

# 【可选】如果你拥有一个域名，并将其泛解析 (*.yourdomain.com) 指向了你的公网服务器 IP，
# 那么可以开启子域名支持，这样就可以通过 "test.yourdomain.com" 来访问内网服务，非常方便。
# subdomainHost = "yourdomain.com"
```

## 2. 创建 systemd 服务文件 frps.service

在公网服务器上，创建 systemd 服务文件 `/etc/systemd/system/frps.service`：

```ini
[Unit]
Description=FRP Server
After=network.target

[Service]
Type=simple
# 建议使用非 root 用户运行，比如 nobody，或创建一个专门的 frp 用户
# 如果使用 nobody，请确保 /etc/frp/ 和 /var/log/ 对 nobody 用户有读写权限
# User=nobody
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.toml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

---

# 第二步：内网机房服务器 (frpc) 配置

这台服务器没有公网 IP，作为 frp 的客户端，它将主动连接到公网的 frps 服务。

## 1. 创建配置文件 frpc.toml

在内网服务器上，创建配置文件 `/etc/frp/frpc.toml`，并填入以下内容：

```toml
# /etc/frp/frpc.toml

# 公网 frps 服务器的 IP 地址
serverAddr = "your_server_public_ip"

# 公网 frps 服务器的连接端口 (必须与 frps.toml 中的 bindPort 一致)
serverPort = 7000

# 身份验证令牌 (必须与 frps.toml 中的 auth.token 一致)
# !! 请务必修改为与服务端相同的密码 !!
auth.token = "your_secure_token_here"

# 【可选】日志配置
log.to = "/var/log/frpc.log" # 日志文件路径
log.level = "info"          # 日志级别
log.maxDays = 7             # 日志文件最长保留天数
log.disablePrintColor = true

# --- 以下是代理规则配置 ---
# 你可以根据需要定义多个代理

# [[proxies]] 块定义一个代理
# 示例1：代理内网的 SSH 服务 (22 端口)
[[proxies]]
name = "local-ssh"
type = "tcp"
localIP = "127.0.0.1" # 本地需要被代理的服务的 IP 地址
localPort = 22        # 本地服务的端口
# remotePort 是在 frps 服务器上开放的端口
# 访问方式：ssh -p 6000 user@your_server_public_ip
remotePort = 6000

# 示例2：代理内网的一个 Web 服务 (例如运行在 8080 端口)
# [[proxies]]
# name = "local-web-app"
# type = "http"
# localIP = "127.0.0.1"
# localPort = 8080
# # customDomains 数组指定了访问此服务的域名
# # 访问方式: http://webapp.yourdomain.com:8080 (如果frps配置了vhostHTTPPort)
# # 如果你的 frps 配置了 subdomainHost = "yourdomain.com"，可以这样写：
# # subdomain = "webapp"
# # 访问方式: http://webapp.yourdomain.com:8080
# customDomains = ["webapp.yourdomain.com"]

# 示例3：代理内网的另一个服务，例如 MySQL 数据库 (3306 端口)
# [[proxies]]
# name = "local-mysql"
# type = "tcp"
# localIP = "127.0.0.1"
# localPort = 3306
# # 访问方式：在公网服务器上或其他地方连接 localhost:33060 即可连接到内网的 MySQL
# remotePort = 33060
```

**重要提示**:

* `serverAddr` 必须填写你的公网服务器的实际 IP 地址。
* `auth.token` 必须与 `frps.toml` 中的完全一致。
* 根据你的实际需求，修改或添加 `[[proxies]]` 规则。

## 2. 创建 systemd 服务文件 frpc.service

在内网服务器上，创建 systemd 服务文件 `/etc/systemd/system/frpc.service`：

```ini
[Unit]
Description=FRP Client
After=network.target

[Service]
Type=simple
# User=nobody
ExecStart=/usr/local/bin/frpc -c /etc/frp/frpc.toml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

---

# 第三步：启动并管理服务

在两台服务器上分别执行以下命令来管理服务。

## 在公网服务器上操作 frps 服务

```bash
# 1. 重新加载 systemd 配置，使其识别新的 frps.service 文件
sudo systemctl daemon-reload

# 2. 启动 frps 服务
sudo systemctl start frps

# 3. 设置 frps 服务开机自启
sudo systemctl enable frps

# 4. 查看 frps 服务状态，确保它正在运行且没有错误
sudo systemctl status frps

# 其他常用命令：
# 停止服务: sudo systemctl stop frps
# 重启服务: sudo systemctl restart frps
# 查看日志: sudo journalctl -u frps -f
```

## 在内网服务器上操作 frpc 服务

```bash
# 1. 重新加载 systemd 配置
sudo systemctl daemon-reload

# 2. 启动 frpc 服务
sudo systemctl start frpc

# 3. 设置 frpc 服务开机自启
sudo systemctl enable frpc

# 4. 查看 frpc 服务状态
sudo systemctl status frpc

# 其他常用命令：
# 停止服务: sudo systemctl stop frpc
# 重启服务: sudo systemctl restart frpc
# 查看日志: sudo journalctl -u frpc -f
```

完成以上步骤后，你的 frp 穿透就已经搭建完成并通过 systemd 进行管理了。你可以通过访问 frps 的 Dashboard (`http://your_server_public_ip:7500`) 来查看客户端连接状态和流量信息。