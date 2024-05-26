---
title: Hexo + Github Actions 博客搭建
date: 2024-05-26 13:02:10
toc: true
# cover: https://source.unsplash.com/random
cover: https://source.unsplash.com/zqzmdrCTW9c
tags: ['博客']
categories: ['折腾']
---

静态博客生成器有很多选择：[Hugo](https://gohugo.io)，[Jekyll](https://jekyllrb.com)，[Hexo](https://hexo.io)...

[GitBook](https://www.gitbook.com)，[VuePress](https://vuepress.vuejs.org)，[docsify](https://docsify.js.org/#/)等文档生成器也可以作为博客平替。

深度体验下来，我最后选择Hexo作为生成器，有两个原因：
1. Hexo基于node.js，国人开发，生态丰富（如果你有前端基础，我推荐你把Hexo作为第一选择）
2. Hugo虽然构建快，但是我觉得V2EX里有一个评论正中下怀：
> hugo 的主题真的比 hexo 差了不少，不知道是不是因为用 hugo 的大多是后端程序员的原因。

作为视觉动物，我觉得Hexo在性能和生态上取得了最好的平衡，而且中文互联网上教程最多，上手难度最低。

以下内容参考[官方文档](https://hexo.io/zh-cn/docs/setup)以及我使用的[Hexo主题icarus配置文档](https://github.com/ppoffice/hexo-theme-icarus)。

<!-- more -->

# 安装
## 安装前提
- Node.js
- Git

如果你是Mac用户，可以通过`homebrew`安装。

## 安装 Hexo
```sh
npm install -g hexo-cli
```

# 建站
## 初始化
在想要放置博客文件夹的目录下，执行
```sh
hexo init <folder>
cd <folder>
npm install
```

新建完成后，可以继续敲入命令`code .`快速用`VS Code`打开，继续接下来的步骤。

在`VS Code`中打开终端，输入
```sh
hexo server
```
然后浏览器访问[http://localhost:4000](http://localhost:4000)即可访问默认博客。

## `_config.yml`配置站点信息

参考[Hexo的配置文件文档](https://hexo.io/zh-cn/docs/configuration)进行修改。

主要的需要修改的有：

```yml
title: Hexo
subtitle: 'Do something.'
description: '个人博客'
author: Terry Tan
language: zh-CN
timezone: 'Asia/Shanghai'
url: # 配置为在线访问的你的博客网站的目录
```
## 目录解析
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
1. `_config.yml`是网站的配置文件，可以在此配置大部分的参数。
2. `package.json`，这是npm依赖管理文件。
3. `scaffolds`是模板文件夹，当您新建文章时，Hexo 会根据 scaffold 来创建文件（下文介绍新建文档命令马上就会涉及）。
4. `themes`，主题文件夹。Hexo 会根据主题来生成静态页面（Hexo现已支持通过npm包依赖的形式引入，所以该文件夹不再是必需）。

## 安装主题

自带的博客不能满足大多数人的个性化需求，而Hexo拥有庞大的主题生态可以从中选择。

Hexo 4.3 已经支持了通过npm包依赖的形式引入主题，非常方便。可以在[Hexo Theme](https://hexo.io/themes/)或者[Awesome Hexo Theme](https://github.com/Ailln/awesome-hexo-theme)等地方挑选一个自己喜欢的主题进行个性化。

下文将以[hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus)主题为例。

```sh
npm install hexo-theme-icarus
hexo config theme icarus
```

`hexo config theme icarus`会新建一个`_config.icarus.yml`文件以及更改`_config.yml`文件中的`theme`属性为`icarus`。

## 主题配置

Icarus的默认主题配置文件为`_config.icarus.yml`。 此文件定义了站点全局的布局与样式设置，同时也控制了例如插件与挂件等外部功能的配置。

Icarus拥有许多的个性化配置选项，可以参考[Icarus用户指南 - 主题配置](http://ppoffice.github.io/hexo-theme-icarus/Configuration/icarus用户指南-主题配置/)进行配置。

当前的目录结构为：
```
.
├── _config.icarus.yml
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
`_config.icarus.yml`会覆盖`_config.yml`里的配置（如果两个配置文件中有相同的配置，`_config.icarus.yml`优先级更高）。

如果有其它的主题，那可以有另一个`_config.theme_name.yml`存在，并通过在`_config.yml`中切换要激活的主题。

> 总而言之，配置源的作用范围和优先级如下：
>
> - 对于某个文章或页面
>   - 文章/页面的front-matter覆盖所有下面的配置源。
>   - 布局配置文件覆盖所有下面的配置源。
>   - 站点配置文件中的theme_config选项覆盖所有下面的配置源。
>   - 主题配置文件覆盖所有下面的配置源。
>   - 站点配置文件。
>   - 对于所有的文章或页面
> - 布局配置文件覆盖所有下面的配置源。
>   - 站点配置文件中的theme_config选项覆盖所有下面的配置源。
>   - 主题配置文件覆盖所有下面的配置源。
>   - 站点配置文件。
>   - 对于所有的文章，页面，和索引页
> - 站点配置文件中的theme_config选项覆盖所有下面的配置源。
>   - 主题配置文件覆盖所有下面的配置源。
>   - 站点配置文件。

### 全局资源文件夹
需要强调的一点是，Hexo的全局资源文件夹是`source`目录，所以除了文章以外的所有文件，例如图片、CSS、JS 文件等，都可以放在`source/img`,`source/js`等目录下，在配置文件中访问时，`/img/favicon.ico`的形式访问。

# 部署

Github提供了免费强大的CI/CD功能和静态博客部署功能，下面将使用 GitHub Actions 部署至 GitHub Pages。

[在 GitHub Pages 上部署 Hexo步骤](https://hexo.io/zh-cn/docs/github-pages)：

> 1. 建立名为 <你的 GitHub 用户名>.github.io 的储存库，若之前已将 Hexo 上传至其他储存库，将该储存库重命名即可。
> 2. 将 Hexo 文件夹中的文件 push 到储存库的默认分支，默认分支通常名为 main，旧一点的储存库可能名为 master。
> 3. 将 main 分支 push 到 GitHub
> 4. 使用 node --version 指令检查你电脑上的 Node.js 版本，并记下该版本 (例如：v20.y.z)
> 5. 在储存库中前往 Settings > Pages > Source，并将 Source 改为 GitHub Actions。
> 6. 在储存库中建立 .github/workflows/pages.yml，并填入以下内容 (将 20 替换为上个步骤中记下的版本)：
> ```yml
> name: Pages
> 
> on:
>   push:
>     branches:
>       - main  # default branch
> 
> jobs:
>   build:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>         with:
>           token: ${{ secrets.GITHUB_TOKEN }}
>           # If your repository depends on submodule, please see: https://github.com/actions/checkout
>           submodules: recursive
>       - name: Use Node.js 20
>         uses: actions/setup-node@v4
>         with:
>           # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
>           # Ref: https://github.com/actions/setup-node#supported-version-syntax
>           node-version: '20'
>       - name: Cache NPM dependencies
>         uses: actions/cache@v4
>         with:
>           path: node_modules
>           key: ${{ runner.OS }}-npm-cache
>           restore-keys: |
>             ${{ runner.OS }}-npm-cache
>       - name: Install Dependencies
>         run: npm install
>       - name: Build
>         run: npm run build
>       - name: Upload Pages artifact
>         uses: actions/upload-pages-artifact@v3
>         with:
>           path: ./public
>   deploy:
>     needs: build
>     permissions:
>       pages: write
>       id-token: write
>     environment:
>       name: github-pages
>       url: ${{ steps.deployment.outputs.page_url }}
>     runs-on: ubuntu-latest
>     steps:
>       - name: Deploy to GitHub Pages
>         id: deployment
>         uses: actions/deploy-pages@v4
> ```
> 7. 部署完成后，前往 https://<你的 GitHub 用户名>.github.io 查看网站。

如果想部署在<你的 GitHub 用户名>.github.io 的子目录中，同样也是有方法的：

1. 建立名为 <repository 的名字> 的储存库，这样你的博客网址为 <你的 GitHub 用户名>.github.io/<repository 的名字>，repository 的名字可以任意，例如 blog 或 hexo。
2. 编辑你的 _config.yml，将 url: 更改为 <你的 GitHub 用户名>.github.io/<repository 的名字>。
3. 在储存库中前往 Settings > Pages > Source，并将 Source 改为 GitHub Actions。
4. 重复上述步骤6
5. Commit 并 push 到默认分支上。
6. 部署完成后，前往 https://<你的 GitHub 用户名>.github.io/<repository 的名字> 查看网站。

# 其它
## RSS 支持
> RSS 的全称是「简易内容聚合」（Really Simple Syndication），是一个能让你在一个地方订阅各种感兴趣网站的工具。
> RSS 的对立面是算法推荐，像微信公众号、知乎、微博、今日头条等平台。 且不说算法推送平台广告多，迁移麻烦的问题。算法推荐的特点是，你不需要刻意选择，算法会根据你的喜好，给你推送内容。这样一来，你几乎没有选择的余地，在不断被「喂饱」中逐渐失去判断的能力。更可怕的地方在于，它替你定义了你的画像，然后把你潜移默化中变成了它所认为的你。「大数据杀熟」的东窗事发绝非偶然，用算法窥视用户隐私是当今互联网公司的通配。
> 做信息的主人，而不是奴隶。RSS 是一种公开的协议，可自由更换平台与客户端。重要的一点是，获取信息的权力完全自治。RSS 相比算法推荐，拥有了可控性和安全感，隐私完全掌握在自己手里。

自从我开始使用RSS以来，受益良多，有许多优秀的大佬坚持个人博客的RSS更新，也激励了我。

Hexo有插件可以非常简单地开启RSS，何乐而不为呢？

### 安装
```sh
npm install hexo-generator-feed --save
```

### 配置
在` _config.yml`中，进行如下配置
```yml
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
  order_by: -date
```
这里是一些常用的配置选项：
- type: 生成 feed 的类型，可以是 atom 或 rss2。Atom 是默认值。
- path: 生成的 feed 文件的路径。默认是 atom.xml 或 rss2.xml。
- limit: 限制 feed 中的文章数量。默认是 20 篇。
- hub: 可选的 WebSub hub URL。
- content: 是否在 feed 中包含文章内容。默认为 true。
- content_limit: 文章内容的限制字符数，默认为无限制。
- content_limit_delim: 内容限制的分隔符，默认为空格。
- order_by: 排序方式，默认为 -date，即按日期降序排列。

### 访问

配置完成后，就可以通过`http://yourdomain.com/atom.xml`订阅内容啦。RSS阅读器我使用的是开源的[NetNewsWire](https://github.com/Ranchero-Software/NetNewsWire)

