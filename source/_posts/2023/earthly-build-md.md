---
title: Earthly好用的Dockerfile构建工具
excerpt: 像构建 Makefile 一样 构建 Dockerfile，熟悉的语法、快速、一致的构建。
index_img: /img/2023/earthly.png
date: 2023-07-28 15:00:00
tags:
---

# Earthly

## 安装

### Mac

```bash
brew install earthly && earthly bootstrap
```

### Linux

```bash
sudo /bin/sh -c 'wget https://github.com/earthly/earthly/releases/latest/download/earthly-linux-amd64 -O /usr/local/bin/earthly && chmod +x /usr/local/bin/earthly && /usr/local/bin/earthly bootstrap --with-autocomplete'
```

### Windows(WSL2)

```bash
sudo /bin/sh -c 'wget https://github.com/earthly/earthly/releases/latest/download/earthly-linux-amd64 -O /usr/local/bin/earthly && chmod +x /usr/local/bin/earthly && /usr/local/bin/earthly bootstrap --with-autocomplete'
```

## 登录

登录 [earthly](https://cloud.earthly.dev/) 创建账号。

```bash
earthly account login --token {your token} && earthly org select {your organization}
```

## 本地编译

```bash
earthly github.com/earthly/hello-world+hello --name="$USER"
```
