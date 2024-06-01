---
title: 前端代码规范工具
toc: true
cover: https://source.unsplash.com/vpOeXr5wmR4
tags: ['Angular', '工具', 'Clean Code']
categories: ['前端']
date: 2024-05-27 22:31:31
---

本篇文章介绍如何快速为Angular项目开启代码规范工具。

[The Importance of unified code style in large scaled applications](https://medium.com/carvago-development/the-importance-of-unified-code-style-in-large-scaled-applications-6c0487ebc12e)

`ESLint`, `typescript-eslint`, `Angular ESLint`, `prettier-eslint`, `eslint-config-prettier`, `eslint-plugin-prettier`, `prettier-eslint-cli`等名词太多啦，傻傻分不清？更别说还有`Prettier`, `lint-staged`, `husky`, `stylelint`, `commitlint`, `mrm`等工具。如果要再加上`VS Code`的插件的话，那就还有`prettier-vscode`, `vscode-eslint`, `vs-code-prettier-eslint`, `vscode-stylelint`, `vscode-commitlint`等等，数不胜数。

本篇博客从应用的角度，记录如何应用这些工具，实现代码的风格统一和代码质量。

<!-- more -->

# enables ESLint to lint Angular projects
## 安装
首先在项目中使用Angular的schematics特性快速添加和配置`ESLint`。
```sh
ng add @angular-eslint/schematics
```
让我们深入了解一下这个原理图实际做了些什么。首先，在`package.json`文件中添加了相关的依赖和指令。

```json
{
  "scripts": {
    "test": "ng test",
    "lint": "ng lint"
  },
  "devDependencies": {
    "@angular-eslint/builder": "17.5.1",
    "@angular-eslint/eslint-plugin": "17.5.1",
    "@angular-eslint/eslint-plugin-template": "17.5.1",
    "@angular-eslint/schematics": "17.5.1",
    "@angular-eslint/template-parser": "17.5.1",
    "@typescript-eslint/eslint-plugin": "7.10.0",
    "@typescript-eslint/parser": "7.10.0",
    "eslint": "^8.57.0",
  }
}
```

其次，在`angular.json`中添加了对应的配置。

```json
{
  "projects": {
    "architect": {
      "lint": {
        "builder": "@angular-eslint/builder:lint",
        "options": {
          "lintFilePatterns": [
            "src/**/*.ts",
            "src/**/*.html"
          ]
        }
      }
    }
  },
  "cli": {
    "schematicCollections": [
      "@angular-eslint/schematics"
    ]
  }
}
```
新建一个`.eslintrc.json`文件，进行`typescript`和`angular`风格的`eslint`的lint配置。
```json
{
  "root": true,
  "ignorePatterns": [
    "projects/**/*"
  ],
  "overrides": [
    {
      "files": [
        "*.ts"
      ],
      "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended",
        "plugin:@angular-eslint/recommended",
        "plugin:@angular-eslint/template/process-inline-templates"
      ],
      "rules": {
        "@angular-eslint/directive-selector": [
          "error",
          {
            "type": "attribute",
            "prefix": "app",
            "style": "camelCase"
          }
        ],
        "@angular-eslint/component-selector": [
          "error",
          {
            "type": "element",
            "prefix": "app",
            "style": "kebab-case"
          }
        ]
      }
    },
    {
      "files": [
        "*.html"
      ],
      "extends": [
        "plugin:@angular-eslint/template/recommended",
        "plugin:@angular-eslint/template/accessibility"
      ],
      "rules": {}
    }
  ]
}
```

## 使用
可以使用如下命令来尝试修复代码中的警告。
```sh
# --fix:  Fixes linting errors (may overwrite linted files).
ng lint --fix
# or
npm run lint --fix
```
如果只需要使用`ESLint`，那么到这里就结束了。如果还需要搭配`prettier`等工具一起使用，请继续往下看。

# enables Prettier
## 安装
```sh
npm install prettier --save-dev
```
然后我们需要在项目根目录新建两个文件：`.prettierrc.json`和`.prettierignore`。`.prettierignore`配置哪些文件不需要被格式化，可以大大缩短格式化时间。对应的文件配置都可以在互联网上查询到相关的配置。

这里我借鉴了网络上一位大佬的配置。

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "singleQuote": true,
  "semi": true,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "trailingComma": "es5",
  "bracketSameLine": true,
  "printWidth": 80,
  "endOfLine": "auto"
}
```

## 使用
可以通过以下命令简单地使用`prettier`:
```sh
npx prettier --write . --ignore
```
但如果需要把`linter`和`prettier`一起使用，有些规则会冲突，你会发现格式化完`eslint`又报错了，修复后又不符合`prettier`规范了。这些规则要自己修改的话很繁琐，还好`eslint-config-prettier`为我们提供了支持。

# enable eslint-config-prettier & eslint-plugin-prettier
## eslint-config-prettier & eslint-plugin-prettier & prettier-eslint 区别简述
- `eslint-config-prettier` 会配置如何处理eslint和prettier规则间的差异，对用户来说，差异不可见，没有差异
- `eslint-plugin-prettier` 使prettier作为eslint插件的形式，每次执行eslint会执行prettier检查
- `prettier-eslint` 是一个将 Prettier 和 ESLint 结合使用的工具。它先使用 Prettier 格式化代码，然后再通过 ESLint 进行修复

## 安装

`prettier-eslint` 是可选的，如果`npm install --save-dev prettier-eslint-cli`安装上了就可以添加如下脚本
```json
{
  "scripts": {
    "format": "prettier-eslint \"src/**/*.js\""
  }
}
```
进入正题：
```sh
npm install prettier-eslint-cli eslint-config-prettier eslint-plugin-prettier --save-dev
```

然后在`.eslintrc.json`中的每个需要的`extends`的**最后一行**添加：
```json
{
  "extends": ["plugin:prettier/recommended"]
}
```
因为还有一些额外的配置，所以可以直接参考下面的内容:
```json
{
  "root": true,
  "ignorePatterns": ["projects/**/*"],
  "overrides": [
    {
      "files": ["*.ts"],
      "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended",
        "plugin:@angular-eslint/recommended",
        "plugin:@angular-eslint/template/process-inline-templates",
        "plugin:prettier/recommended"
      ],
      "rules": {
        "@angular-eslint/directive-selector": [
          "error",
          {
            "type": "attribute",
            "prefix": "app",
            "style": "camelCase"
          }
        ],
        "@angular-eslint/component-selector": [
          "error",
          {
            "type": "element",
            "prefix": "app",
            "style": "kebab-case"
          }
        ]
      }
    },
    {
      "files": ["*.html"],
      "extends": [
        "plugin:@angular-eslint/template/recommended",
        "plugin:@angular-eslint/template/accessibility"
      ],
      "rules": {}
    },
    {
      "files": ["*.html"],
      "excludedFiles": ["*inline-template-*.component.html"],
      "extends": ["plugin:prettier/recommended"],
      "rules": {
        "prettier/prettier": ["error", { "parser": "angular" }]
      }
    }
  ]
}
```
这样，可以看到，当执行完`lint`之后，会继续执行`prettier`检查，并将报错提示在`eslint`的信息界面。

```sh
➜  frontend-formatter-demo git:(main) ✗ npm run lint

> frontend-formatter-demo@0.0.0 lint
> ng lint


Linting "frontend-formatter-demo"...

~/frontend-formatter-demo/src/app/app.component.ts
  12:10  error  Delete `·`                                prettier/prettier
  12:11  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

✖ 2 problems (2 errors, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.

Lint errors found in the listed files.
```

修复也很简单，最好还是挨个修复或者使用`prettier-eslint`。

# 与 VS Code 集成
前端编辑器，现在最流行的非VS Code莫属，特别是丰富的插件生态，更是让开发者事半功倍。有兴趣的话还可以继续研究一下怎么安装VS Code 插件进行风格化。

在根目录的`.vscode`目录下新建`settings.json`文件，加入如下配置：
```json
{
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "always"
    },
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "rvest.vs-code-prettier-eslint",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "always"
    },
    "editor.formatOnSave": true,
    "editor.suggest.snippetsPreventQuickSuggestions": false,
    "editor.inlineSuggest.enabled": true
  }
}
```

`extensions.json`可以列出推荐安装的插件，当打开插件标签页的时候，就会显示推荐的插件。属性值填写`Unique Identifier`，能在对应的[visualstudio marketplace](https://marketplace.visualstudio.com)上找到。
```json
{
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=827846
  "recommendations": [
    "angular.ng-template",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "rvest.vs-code-prettier-eslint"
  ]
}
```
这样工作区就会拥有这些配置，当右键选择格式化的时候就会以这些插件进行，具体的格式会扫描工作区中的对应配置文件，也就是上面写好的`.eslintrc.json`和`.prettierrc.json`等等。

到这里就完成了所有配置啦。明天继续记录和`lint-staged`, `husky`等工具的集成。

# Husky & lint-staged

书接上文，如果你逛Github的话，就会发现很多项目根目录除了`.github`还有个`.husky`文件夹，我一直好奇这个文件夹的作用。今天正好创建了一个新项目，借此机会一探究竟。

## 了解Git Hooks
Git Hooks 是在 Git 执行特定事件（如commit、push、receive等）时触发运行的**脚本**（也就是Shell，Python等语言都可以），类似于“钩子函数”。跟钩子函数一样，Git Hooks可以起到一个承上启下的作用。

Git Hooks是Git提供的特性。钩子都被存储在 Git 目录下的 hooks 子目录中。 也即绝大部分项目中的 .git/hooks 。 当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。钩子又可以分类为客户端钩子和服务器端钩子。客户端钩子分为很多种。 下面把它们分为：提交工作流钩子、电子邮件工作流钩子和其它钩子。如果想了解更多关于Git Hooks的细节，请参考[Git官方文档（Git 钩子）](https://git-scm.com/book/zh/v2/自定义-Git-Git-钩子)

### Git Hooks存在的问题
由于Git Hooks默认是存在`.git`目录下的，无法进行版本控制。好在Git新版本中`core.hooksPath`的出现可以使Hooks存放的路径指向自定义的目录。

## husky
为什么需要在git hooks上再多husky这个上层建筑呢？简而言之，**它能够简化创建和修改Githooks的操作**。

### 安装husky
根据[husky官方文档](https://typicode.github.io/husky/getting-started.html)，首先安装依赖
```sh
npm install --save-dev husky
```
然后执行：
```sh
npx husky init
```
它会：
1. 添加`prepare`脚本到`package.json`
2. 创建一个模板`pre-commit`钩子
3. 配置Git hooks路径到`.husky`

## lint-staged
lint-staged 是一个在 **git 暂存文件(staged)** 上运行 linters 的工具。

官方的原文是：
> Run linters against staged git files and don't let 💩 slip into your code base!

它的好处是：
> But running a lint process on a whole project is slow, and linting results can be irrelevant. Ultimately you only want to lint files that will be committed.

### 安装 lint-staged
```sh
npm install --save-dev lint-staged
```
然后配置`package.json`添加内容：

```json
"lint-staged": {
  "*.{js, jsx,ts,tsx}": [
    "eslint --fix"
  ],
  "*.{json,js,ts,jsx,tsx,html}": [
    "prettier --write --ignore-unknown"
  ]
},
```
（其实我感觉这里`prettier`也不是必须的，因为使用了`eslint-plugin-prettier`插件，实际上`prettier`是以`eslint`插件的形式存在的，每次执行lint就会执行prettier，下文`stylelint`也是同理。）

然后修改`.husky/pre-commit`

```sh
npx lint-staged
```
这样在每次commit之前都会执行`npx lint-staged`，执行`int-staged`并读取上面的`int-staged`配置，如果执行失败呢，就不会`commit`成功。（如果不清楚npm和npx的区别，请参考我的另一篇文章：[npm 和 npx 是什么](../npx_vs_npm)）

# commitlint

从上文就可以得知，lint是一类按照规则执行检查的工具，commitlint就是一个`git commit`校验约束工具。它要求我们的提交记录符合[conventional-commits](https://github.com/conventional-commits/conventionalcommits.org)规范。

## 安装
```sh
npm install --save-dev @commitlint/{config-conventional,cli}
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```
然后在配置文件中配置详细的规则。

### 规则
详细规则参考：[官方文档](https://commitlint.js.org/#/reference-rules)。

#### Angular的案例
不同的编程语言还贴心地给出了不同的提交规范，一般我只需要吧extends的类型修改一下即可。

```sh
npm install --save-dev @commitlint/config-angular @commitlint/cli
echo "module.exports = {extends: ['@commitlint/config-angular']};" > commitlint.config.js
```

### 与husky的结合

往`.husky/commit-msg`文件写入：
```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit "$1"
```
这样，如果提交不符合规范，就不能成功commit到git库了。

```txt
[COMPLETED] Cleaning up temporary files...
⧗   input: commit msg test
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky - commit-msg script failed (code 1)
```

# stylelint

按理说，eslint和stylelint一样，需要enable stylelint-config-prettier & stylelint-plugin-prettier等等， 但是
> As of Stylelint v15 all style-related rules have been deprecated. If you are using v15 or higher and are not making use of these deprecated rules, this plugin is no longer necessary.

所以只需要配置`stylelint-prettier`即可。

## 安装
```sh
npm i -D stylelint stylelint-config-standard-scss stylelint-prettier postcss stylelint-config-sass-guidelines
```

## 配置
新建`.stylelintrc.json`文件，写入如下内容：
```json
{
  "extends": [
    "stylelint-config-standard-scss",
    "stylelint-config-sass-guidelines",
    "stylelint-prettier/recommended"
  ]
}
```
并修改完善`packages.json`中的`lint-staged`配置如下：
```json
"lint-staged": {
  "*.{js,jsx,ts,tsx}": [
    "eslint --fix"
  ],
  "*.{json,js,ts,jsx,tsx,html}": [
    "prettier --write --ignore-unknown"
  ],
  "*.{css,scss}": "stylelint --fix"
},
```
即可。

# 最终测试
修改`app.component.ts`的内容如下：
```ts
export class AppComponent {
  title: any = 'frontend-formatter-demo';
}
```
然后提交出现如下报错：
```
✖ eslint --fix:

/Users/terry/Desktop/frontend-formatter-demo/src/app/app.component.ts
  12:10  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

✖ 1 problem (1 error, 0 warnings)
```
说明大功告成。

# TL;DR;
这是配置的源码仓库，每一节对应一个提交记录：
[a-fly-fly-bird/frontend-formatter-demo](https://github.com/a-fly-fly-bird/frontend-formatter-demo/)

# Reference
## Common
- [commitlint](https://github.com/conventional-changelog/commitlint)
- [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)
- [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier)
- [eslint](https://github.com/eslint/eslint)
- [lint-staged](https://github.com/lint-staged/lint-staged)
- [mrm](https://github.com/sapegin/mrm)
- [prettier-eslint](https://github.com/prettier/prettier-eslint)
- [prettier-eslint-cli](https://github.com/prettier/prettier-eslint-cli)
- [prettier](https://github.com/prettier/prettier)
- [stylelint](https://github.com/stylelint/stylelint)
- [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint)
- [stylelint-config-prettier](https://github.com/prettier/stylelint-config-prettier)
- [stylelint-prettier](https://github.com/prettier/stylelint-prettier)
- [stylelint-config-sass-guidelines](https://github.com/bjankord/stylelint-config-sass-guidelines/tree/main)

## VS Code Plugin
- [prettier-vscode](https://github.com/prettier/prettier-vscode)
- [vscode-eslint](https://github.com/Microsoft/vscode-eslint)
- [vs-code-prettier-eslint](https://github.com/idahogurl/vs-code-prettier-eslint)
- [vscode-stylelint](https://github.com/stylelint/vscode-stylelint)
- [vscode-commitlint](https://github.com/joshbolduc/vscode-commitlint)

## For Angular
- [angular-eslint](https://github.com/angular-eslint/angular-eslint)
- [NX](https://github.com/nrwl/nx)

## 本文参考
- [Configure Prettier and ESLint with Angular 🎨](https://justangular.com/blog/configure-prettier-and-eslint-with-angular)
- [Why and How to Lint like a PRO](https://medium.com/tbc-engineering/why-and-how-to-lint-like-a-pro-173fc4a73899)
