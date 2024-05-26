---
title: 前端风格统一化
toc: true
tags: [‘工具’, 'Clean Code']
cover: https://source.unsplash.com/f4R3w0w-jf4
categories: ['前端']
date: 2024-05-26 23:33:19
---

## 前言
如果你逛Github的话，就会发现很多项目根目录除了`.github`还有个`.husky`文件夹，我一直好奇这个文件夹的作用。今天正好创建了一个新项目，借此机会一探究竟。

<!-- more -->

### 项目地址
- [husky](https://github.com/typicode/husky)
- [prettier](https://github.com/prettier/prettier)
- [lint-staged](https://github.com/lint-staged/lint-staged)
- [eslint](https://github.com/eslint/eslint)
- [commitlint](https://github.com/conventional-changelog/commitlint)

## 了解Git Hooks
Git Hooks 是在 Git 执行特定事件（如commit、push、receive等）时触发运行的**脚本**（也就是Shell，Python等语言都可以），类似于“钩子函数”。跟钩子函数一样，Git Hooks可以起到一个承上启下的作用。

Git Hooks是Git提供的特性。钩子都被存储在 Git 目录下的 hooks 子目录中。 也即绝大部分项目中的 .git/hooks 。 当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。钩子又可以分类为客户端钩子和服务器端钩子。客户端钩子分为很多种。 下面把它们分为：提交工作流钩子、电子邮件工作流钩子和其它钩子。如果想了解更多关于Git Hooks的细节，请参考[Git官方文档（Git 钩子）](https://git-scm.com/book/zh/v2/自定义-Git-Git-钩子)

### Git Hooks存在的问题
由于Git Hooks默认是存在`.git`目录下的，无法进行版本控制。好在Git新版本中`core.hooksPath`的出现可以使Hooks存放的路径指向自定义的目录。

## husky
为什么需要在git hooks上再多husky这个上层建筑呢？简而言之，**它能够简化创建和修改Githooks的操作**。

### 安装husky
根据[husky官方文档](https://typicode.github.io/husky/getting-started.html)，
```sh
npx husky-init && npm install
```
它会：
1. 添加`prepare`脚本到`package.json`
2. 创建一个模板`pre-commit`钩子
3. 配置Git hooks路径到`.husky`

### 自定义钩子
以下命令会创建一个`pre-commit`git hook，也就是在每次commit之前都会执行`npm test`,如果执行失败呢，就不会`commit`成功。
```sh
npx husky add .husky/pre-commit "npm test"
```
（如果不清楚npm和npx的区别，请参考我的另一篇文章：[npm 和 npx 是什么](../npx_vs_npm)）
## prettier
> Prettier is an opinionated code formatter. It enforces a consistent style by parsing your code and re-printing it with its own rules that take the maximum line length into account, wrapping code when necessary.

简而言之，Prettier 是一个代码美化工具。支持JavaScript · TypeScript · Flow · JSX · JSON · CSS · SCSS · Less · HTML · Vue · Angular · GraphQL · Markdown · YAML等语言。

### 安装prettier
同样，参考自[官方文档](https://prettier.io/docs/en/install)。

```sh
npm install --save-dev --save-exact prettier
```
然后执行下列命令会在根目录新建一个空的`prettier`配置文件。
```sh
node --eval "fs.writeFileSync('.prettierrc','{}\n')"
```

### CLI使用
主要有两个操作：
- check（检查文件是否符合代码风格，但不执行格式化）
- write（格式化文件）

#### check
这里的`.`是指当前目录，可以自己指定想要格式化的目录。
```sh
npx prettier . --check
```

#### write
这里的`.`是指当前目录，可以自己指定想要格式化的目录。
```sh
npx prettier . --write
```

### 与编辑器集成
除了执行命令格式化之外，更常用的是在编辑器中执行操作，比如保存文件时触发格式化。本文主要记录与Husky的联动，不打算深究这部分，具体参考[Editor Integration](https://prettier.io/docs/en/editors)。

### eslint-config-prettier
注意⚠️：如果项目中同时使用了eslint和prettier，那么还需要[eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)，来避免eslint和prettier配置的冲突。它会关闭所有非必要或者会导致与prettier冲突的ESLint规则。

#### Prettier vs. Linters
> Linters have two categories of rules:
> 
> **Formatting rules**: eg: max-len, no-mixed-spaces-and-tabs, keyword-spacing, comma-style…
> 
> Prettier alleviates the need for this whole category of rules! Prettier is going to reprint the entire program from scratch in a consistent way, so it’s not possible for the programmer to make a mistake there anymore :)
> 
> **Code-quality rules**: eg no-unused-vars, no-extra-bind, no-implicit-globals, prefer-promise-reject-errors…
> 
> Prettier does nothing to help with those kind of rules. They are also the most important ones provided by linters as they are likely to catch real bugs with your code!
> 
> In other words, use Prettier for formatting and linters for catching bugs!

总儿颜值就是，使用`Prettier`来格式化代码，`linters`来抑制bug。

linters的一个代表就是eslint：eslint是一个按照规则执行代码检查的工具，它可以在编码阶段进行静态分析，给出检查报告。搭配一些插件，可以提前暴露问题，给出提示，并进行修复，大大减少执行过程中的bug。

## lint-staged
lint-staged是一个在**git暂存文件(staged)**上运行linters的工具。
官方的原文是：
> Run linters against staged git files and don't let 💩 slip into your code base!

它的好处是：
> But running a lint process on a whole project is slow, and linting results can be irrelevant. Ultimately you only want to lint files that will be committed.

### 安装 lint-staged
```sh
npm install --save-dev lint-staged
```

使用需要搭配`husky`和`Prettier`等工具，直接看[最佳实践部分]()。

## commitlint
从上文就可以得知，lint是一类按照规则执行检查的工具，commitlint就是一个`git commit`校验约束工具。它要求我们的提交记录符合[conventional-commits](https://github.com/conventional-commits/conventionalcommits.org)规范。

### 安装
```sh
npm install --save-dev @commitlint/{config-conventional,cli}
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```
然后在配置文件中配置详细的规则。

### 规则
详细规则参考：[官方文档](https://commitlint.js.org/#/reference-rules)。

#### angular的案例
不同的编程语言还贴心地给出了不同的提交规范，一般我只需要吧extends的类型修改一下即可。

```sh
npm install --save-dev @commitlint/config-angular @commitlint/cli
echo "module.exports = {extends: ['@commitlint/config-angular']};" > commitlint.config.js
```

### 与husky的结合
要在提交之前生效，需要添加如下钩子：
```sh
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

## 最佳实践
参考自：[Prettier #Git hooks](https://prettier.io/docs/en/install#git-hooks)

```sh
# prettier
npm install --save-dev --save-exact prettier
node --eval "fs.writeFileSync('.prettierrc','{}\n')"
# husky & lint-staged
npm install --save-dev husky lint-staged
npx husky install
npm pkg set scripts.prepare="husky install"
npx husky add .husky/pre-commit "npx lint-staged"
# commitlint
npm install --save-dev @commitlint/{config-conventional,cli}
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```
然后在`package.json`中加入：
```json
{
  "scripts": {
    ...
  },
  "lint-staged": {
    "**/*": "prettier --write --ignore-unknown"
  }
}
```
这样就会在每次提交代码之前对所有支持的文件类型执行格式化操作。

### 限制lint-staged的范围
```json
{
  "scripts": {
    ...
  },
  "lint-staged": {
    "src/**/!(*.min).js": [
      "prettier --check",
    ],
    "src/**/*.{ts,vue}": [
      "prettier --write",
    ],
    "src/**/*.{ts,js,vue,html,css,scss,sass,stylus}": [
      "prettier --write"
    ]
  },
}
```
这样就会对不同的文件夹下面的不同文件类型的文件执行不同的脚本。

