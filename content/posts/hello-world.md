---
title: Hello World - Hexo and Next Guild
tags: [博客, Hexo]
categories: 云的旅途
date: 2020-07-05T13:36:38+08:00
lastmod: 2020-07-10T08:37:18+08:00
---

![](https://cdn.jsdelivr.net/gh/hotsnow-sean/image-host/img/20200707210446.png)
第一篇博文，没什么内容，依旧充满情怀地对这个世界问声好，Hello world!  
其实是由于 hexo 搭建博客必须保证至少有一篇博文才能正常使用，233，顺便简单记录一下搭建博客的过程

<!--more-->

# 使用 Hexo 搭建个人博客流程记录

## 准备工作

必要安装的软件程序：nodejs、git
需要的账号：github 账户

## 搭建流程

1. 在 github 上创建一个仓库，仓库名为 `GitHub用户名.github.io`，搭建完成后将用这个链接访问博客
2. 运行 `npm install -g hexo-cli` 安装 `hexo`
3. 在本地新建文件夹用于存储相关内容，然后进入此文件夹，命令行下运行 `hexo init`，等待运行完毕，脚手架工具将在此文件夹下创建好所需的所有文件
4. 到这一步，其实已经可以在本地查看博客页面了，运行 `hexo serve` 或 `hexo s`，即可打开本地服务器，打开浏览器访问 `localhost:4000`，啪唧，博客出现了![](https://cdn.jsdelivr.net/gh/hotsnow-sean/image-host/img/20200706203100.png)
5. 当然，我们还需要将其部署到远端 GitHub 上，也就是第一步创建的仓库。打开本地博客文件夹下的 `_config.yml` 文件，翻到文件底部`deploy`配置项，将`type`设为`git`，将`repo`设为仓库的 GitHub 链接地址，将`branch`设为`master`。
6. 配置好后，运行 `hexo d` 即可部署博客到仓库了（过程中可能需要登录 git 账户）
7. OK，万事俱备，已经可以通过第一步中的链接正常访问博客喽！！！接下来就让无处安放的才华在这里生根发芽吧，冲冲冲 👊👊👊

## 后续更新

为了备份 hexo 源码到仓库，实现多终端无缝衔接撰写博客，作以下步骤：

1. 在博客部署所在的仓库新建分支 `hexo`，并设为默认
2. 在本地将仓库克隆下来
3. 将克隆到的文件夹里除 `.git` 文件夹之外全部删除
4. 将原博客文件夹里除 `.deploy_git` 和 `public` 之外的所有文件文件夹移动到新克隆下的文件夹里
5. 到此，就可以使用 `git push` 等相关操作将源码文件上传到 hexo 分支了，当更换设备后可以直接克隆下来运行 `npm install` 后继续编写
   注意点：

- 将主题文件夹里面的 `.git` 删除，因为仓库无法嵌套上传
- 使用 `.gitignore` 文件，将 `node_modules` 文件夹以及第 4 步提到的文件夹排除在上传之外

## 最新更新 - 重构源文件结构 + 分离主题配置

由于主题文件经常会有更新，如果直接在主题内部配置，当你更新主题时你的配置项将全部消失需要重新配置。因此采用 Next 主题 推荐的方式进行配置，完全不改动主题文件夹，这样就可以使用 `git pull` 来直接拉取最新主题并且保留原本的配置了。具体做法如下：

1. 选择一种配置方式，Next 主题提供两种方式：

- 一是直接在 Hexo 项目的配置文件 `_config.yml` 中加入 `theme_config` 选项，有关主题的配置都写在这个配置项下面；
- 二是在项目的 `source/_data` 文件夹下（没有就创建），创建 `next.yml` 文件，主题配置可直接写入这个文件
  ✨ 这里我选择第二种方式，因为第一种方式在修改后必须重启 `hexo serve` 才能看到效果（如果正在`serve`），并且和项目配置混在一个文件，还需要多缩进一格，感觉不是很爽就对了。【注：经实测，第二种方式的修改也不会像直接修改主题内部配置那样立刻应用到页面上，但是也不需要重启 `serve`，多保存几次或者等一会儿都可以】

2. 将需要改变的主题配置项复制到 `next.yml` 文件，修改保存即可
3. 除配置项外，样式文件也可以分离修改，在主题配置文件中有 `custom_file_path` 这样一项配置，将其中的 `variable`、`style` 选项打开，并按照其路径名称在 `source/_data` 文件夹下建立相应文件，之后，主题中 `source/css/_variables` 文件夹里所有需要修改的样式都可以放入对应新建立的文件里，而其它需要修改的样式则可以放入新建立的另一个文件里。

---

默认 Hexo 文档哦 👇

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

```bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

```bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

```bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

```bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
