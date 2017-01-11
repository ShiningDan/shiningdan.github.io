---
title: Github上Hexo项目源码以及发布版管理
date: 2017-01-10 19:29:37
categories: coding
tags:
  - GitHub
  - Hexo
---

## 前言

在原来的电脑中，我的 hexo 备份在 GitHub上，除了一个 User Page 用来发布文章以外，还有一个 hexo-backup 的仓库，用来保存 hexo 的源代码。

在我换电脑以后，需要在新的电脑上重新部署 hexo 的环境，由于原来 hexo 的源代码及文稿与发布的仓库位于两个不同的仓库中，导致我需要下载一个仓库，但是要 hexo deploy 到另一个仓库中，这是一个很不合理的项目规划方案。所以，在网上搜索了一些教程以后，我重新规划了 GitHub 上面的代码仓库，用一个仓库同时存放 hexo 项目的发布版和源文件，但是这两个部分位于一个仓库的不同分支中。在以后需要重新部署 hexo 环境的时候，只用从一个仓库中下载代码，就可以获得源代码和发布版。

感谢[CrazyMilk](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo搭建博客/#more)的文章，为我提供了使用一个仓库同时管理发布版和源代码的解决方案！

<!--more-->

## 环境配置

### 创建 GitHub 仓库

首先点击 New Repository，创建一个新的代码仓库，在填写 Repository name 的时候，格式是 username.github.io (username 是你的 GitHub 账号用户名)。

### 本地安装环境配置

需要下载的软件有 Git、node.js 和 Hexo ，下载及安装方法不再赘述。

**注意，在配置 Git 的时候，最好使用 git config 进行配置**，这与 GitHub 在计算 contributions 次数的有关，也是就 GitHub 个人首页上绿色小点，用来统计每个人每天 commit 的次数有关。

在 GitHub 上统计代码仓库 commit 的次数时候，需要同时满足以下三点，才可以记为一次 commit：

* The email address used for the commits is associated with your GitHub account.
* The commits were made in a standalone repository, not a fork.
* The commits were made:
    * In the repository's default branch (usually master)
    * In the gh-pages branch (for repositories with Project Pages sites)

也就是你的本地 Git 账号需要和 GitHub 账号一致。所以需要通过 git config 来配置本地 Git 的账号：

```
git config --global user.name "username"
git config --global user.email "username@example.com"
```

### 本地博客搭建流程

本流程与[CrazyMilk](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo搭建博客/#more)的文章中的流程有所不同，如果是按照他的流程，在 `hexo init` 的时候，会删除本地的 .git 文件夹，所在流程上会略有不同。

以我的博客搭建流程为例：

1. 在 GitHub 上创建 shiningdan.github.io 代码仓库
2. 本地下载和安装 git、node.js，并且使用 git config 配置本地 git 账号
3. 在本地合适的位置创建 shiningdan.github.io 文件夹，作为本地博客根目录
4. 在本地 shiningdan.github.io 依次执行：`
npm install -g hexo
hexo init
npm install
npm install hexo-deployer-git
`
5. 使用 GitHub 提示，将本地文件夹与远程代码仓库连接：`
echo "# shiningdan.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/ShiningDan/shiningdan.github.io.git
git push -u origin master
`
6. 创建 master 和 hexo 分支。因为创建的代码仓库是 User Page，所以 master 分支是作为 hexo deploy 的分支，而 hexo 分支是保存源代码的分支。在 terminal 下执行命令：`
git branch hexo //创建 hexo 分支
git push origin hexo //将本地分支 PUSH 到远端
git checkout hexo //切换本地分支为 hexo
`
7. 修改_config.yml中的deploy参数，修改 deploy `type:git`、`repo:https://github.com/ShiningDan/shiningdan.github.io.git`、`branch: master`。注意，branch 一定要为 master，因为 User Page 的发布版必须位于 master 分支下。
8. 修改 GitHub 网页中的配置，将 hexo 分支设置为默认分支。
9. 依次执行`git add .`、`git commit -m “…”`、`git push origin hexo`提交网站相关的文件
10. 执行`hexo generate -d`生成网站并部署到GitHub上

### 日常发布以及备份

在备份源代码和源文件的时候，依次执行`git add .`、`git commit -m “…”`、`git push origin hexo`指令将改动推送到GitHub，此时源代码和源文件是被备份在 hexo 分支上。

再执行`hexo generate -d`发布网站到master分支上

### 重新部署

1. 使用`git clone https://github.com/ShiningDan/shiningdan.github.io.git`拷贝仓库（默认分支为hexo）
2. 在本地新拷贝的shiningdan.github.io文件夹下通过Git bash依次执行下列指令：`npm install -g hexo`、`npm install`、`npm install hexo-deployer-git`（记得，不需要hexo init这条指令）

