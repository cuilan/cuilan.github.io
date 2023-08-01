---
title: Git多用户配置
excerpt: 大多数情况下，我们本地会有多个 git 用户，在公司连接内网的 gitlab 账号，个人的项目需要使用到 github 账户，或一些其他别的账户，那么就需要配置多用户。
index_img: /img/2023/git.png
date: 2023-08-01 18:14:00
tags:
  - git
categories:
  - git
---

大多数情况下，我们本地会有多个 git 用户，在公司连接内网的 gitlab 账号，个人的项目需要使用到 github 账户，或一些其他别的账户，那么就需要配置多用户。

# git config

一般无论在 gitlab 还是 github 等代码托管平台上创建一个空项目，都会有以下两行提示：

```bash
git config --global user.name "xxx"
git config --global user.email "xxx@xxx.com"
```

设置完成后，会在 home 目录下生成： `~/.gitconfig` 文件，如果需要为单独某个 git 仓库设置，可以进入该仓库目录下，执行：

```bash
git config user.name "xxx"
git config user.email "xxx@xxx.com"
```

或修改 `.git/config` 文件：

```config
[user]
    name = xxx
    email = xxx@xxx.com
```

全部配置的方式虽然简单，但不够灵活；为每一个仓库分别配置，太过于复杂。

# Conditional Includes

在 git 2.13 版本加入了 [conditional includes](https://git-scm.com/docs/git-config#_includes) 配置，可以创建多个 gitconfig 文件，并针对不同的根目录使用不同的配置文件。

# 配置案例

拿 gitlab、github、gitee 举例：

## 创建多用户配置

可在任意目录（建议在 `~/` 目录）下分别创建 `.gitconfig_gitlab` 、 `.gitconfig_github` 、 `.gitconfig_gitee` 文件。

.gitconfig_gitlab
```config
[user]
    name = gitlabName
    email = xxx@gitlab.com
```

.gitconfig_github
```config
[user]
    name = githubName
    email = xxx@github.com
```

.gitconfig_gitee
```config
[user]
    name = giteeName
    email = xxx@gitee.com
```

## 修改 .gitconfig 文件

```config
# gitlab
[includeIf "gitdir:~/code/gitlab/**"]
	path = ~/.gitconfig_gitlab

# github
[includeIf "gitdir:~/code/github/**"]
	path = ~/.gitconfig_github

# gitee
[includeIf "gitdir:~/code/gitee/**"]
	path = ~/.gitconfig_gitee
```

gitdir 配置以 `"/"` 结尾，表示：`"/**"` 。

## 验证

分别在 `~/code/gitlab` 、 `~/code/github` 、 `~/code/gitlab` 下执行：

```bash
git config user.name
...
gitlabName
githubName
giteeName
```
