---
title: CI/CD - Jenkins 使用
toc: true
cover: https://source.unsplash.com/rYQlRntSU0E
tags: ['CI/CD']
categories: ['折腾']
date: 2024-05-28 23:44:31
---

入职之后，一直对公司的CI/CD流程实现细节一知半解，最近正好需要打包部署一个应用，借此机会一口气把所有黑盒给打开。

<!-- more -->

# CI
``
## Jenkinsfile

Jenkinsfile语法规则是基于Groovy语言的DSL。

官方教程：[Jenkins 入门指南](https://www.jenkins.io/zh/doc/pipeline/tour/getting-started/)

### 扩展共享库
项目Jenkinsfile中的第一行基本都是:
```groovy
@Library('my-shared-library') _
```
一直很好奇这句话的意思，没想到背后的东西挺多，一直回顾到Groovy语言。但其实只要看官方教程地址：[扩展共享库](https://www.jenkins.io/zh/doc/book/pipeline/shared-libraries/)就能很快地了解这是什么。（我之前一直不知道怎么搜索这个语法的原理）

在 Jenkinsfile 中使用 `@Library('my-shared-library') _` 这种语法是 Jenkins 的一种特殊功能，允许你加载共享库以便在流水线中使用库中的步骤、变量和类。这里的共享库是指在 Jenkins 实例中配置的、可以被多个 Jenkins 项目复用的代码库。

解释一下这个语法：

- `@Library('my-shared-library')` 是一个指示器（directive），用于告诉 Jenkins 要加载命名为 `my-shared-library` 的共享库。这个名称对应于 Jenkins 配置中的共享库名称。
- `_` 是一个特殊的记号，用于加载库中定义的全局变量。当你使用这个记号时，它会自动加载共享库中名为 `vars` 目录下的所有 Groovy 脚本作为全局变量。这意味着这些脚本中定义的方法可以在 Jenkinsfile 中直接调用，而不需要任何 `import` 语句。

这个语法允许在 Jenkinsfile 中使用共享库提供的资源，而无需额外的导入步骤。实际上，这是一种 DSL（领域特定语言）层面的集成，它是 Jenkins 流水线插件提供的功能，不同于标准的 Groovy 语法。

在共享库中，你可以定义步骤、全局变量或类。步骤通常定义在 `vars` 目录下，全局变量则可以在 `vars` 下直接定义，或者通过类定义在 `src` 目录下的包中。如果你想在 Jenkinsfile 中使用 `src` 目录下的类，你可能需要使用 `import` 语句，除非你使用了全局变量来包装这些类的功能。

简而言之，`@Library('my-shared-library') _` 这种写法是 Jenkins 流水线语法的一部分，它允许开发者以一种简洁的方式在 Jenkinsfile 中加载和使用共享库资源。

