---
title: element-ui源码整体架构分析
categories: 学习笔记
date: 2018-06-13 21:29:24
tags:
---
## 为什么阅读element-ui源码
1. 之前没阅读过一个完整项目的源码,打算从中认识一个项目整体的构造。
2. 在我目前的技术栈中有用到该组件，也方便我更加灵活的运用。
3. 该组件使用了vue，可以见识其他人对vue的运用，加深我对vue的认识。

<!-- more -->
## 整体目录
```javascript
├── build            // 用于自动化生成文件的脚本，配合package.json设定的命令使用
├── examples         // element-ui官网代码
├── src              // 资源文件，由webpack打包后，放入lib文件夹
├── lib              // 发布到npm供用户下载使用
├── packages         // 每个element-ui组件的核心代码
├── test             // 测试文件
├── types            // typescript文件
├── package.json     // 项目配置文件

// 只列出部分重要文件

```
> 树状图的生成一开始我以为是markdown语法，没想到是系统的命令，由于windows系统参数太少，使用了[tree-cli](https://www.npmjs.com/package/tree-cli)插件,其他系统可以参考[这篇文章](https://wodewone.github.io/2017/02/19/node-tree-user/)。

## package.json
`package.json`文件相当于整个项目的入口文件，我们沿着此文件基本可以分析出项目的整体架构。
### "scripts"字段
主要分析一下各指令的作用（通过`npm run 指令`运行），以及过程，列出每个功能的进出口，在`vscode`中，输入快捷键`Ctrl+P`，键入文件地址便可快速查看，至于每个功能具体实现方法不作进一步分析，大家可以自行阅读源码。
#### "bootstrap" 
默认使用yarn安装包，其次是npm",(官方推荐yarn安装，否则会容易报错)
#### "build:file"
 通过执行`build/bin`中的以下4个文件，使用模板渲染生成相应文件。
- **iconInit.js**
作用：提供icon名称供`icon.md`渲染
过程：
    1.读取`packages/theme-chalk/src/icon.scss`
    2.通过`postcss`组件读取选择器，使用正则表达式筛选
    3.生成`examples/icon.json`
- **build-entry.js**
作用：建立webpack所需的入口文件。
过程：读取`components.json`，该文件包含了所`element-ui`所有组件名称，以及代码入口地址。脚本文件中自定义了模板，通过`json-templater/string`组件渲染生成`./src/index.js`。
- **i18n.js**
作用：通过模板生成多种语言，供官网使用。
过程：通过`examples/pages/template/`中的模板文件生成`examples/pages/`中的`.vue`文件。
- **version.js**
作用：处理版本信息。
过程：生成`examples/versions.json`

#### "build:theme"
建立主题所需文件。
- **运行build/bin/gen-cssfile**
作用：生成css的入口文件`packages\theme-chalk\src\index.scss`
过程：也是根据`/components.json`渲染
- **gulp执行任务并剪切**
作用：生成css代码，并使用`cp-cli`组件剪切到`lib/theme-chalk`
过程：gulp根据`packages/theme-chalk/gulpfile.js`脚本编译及压缩`./src/*.scss`中的代码。

#### "build:utils"
作用：使用`babel`编译`src`中除了`index.js`的其他工具代码，放入`lib`中。

#### "build:umd"
作用：编译`src/locale/lang`中的文件，使之符合umd规范，生成的文件放入`lib/umd/locale`

#### "clean"
通过`rimraf`组件删除指定文件。

#### "deploy"
1.通过`build/webpack.demo.js`打包文件
2.利用github Pages，`gh-pages`组件部署官网

#### "dev"和"dev:play"
`dev指令`利用`webpack-dev-server`组件进行本地浏览element网站，暂时还不知道`dev:play`的作用。

#### "dist"
打包文件，生成的文件在`lib`中,发布到npm供用户使用。

#### "lint"
在打包前规范特定文件的代码，如有不符合规则的代码，则抛出错误。

#### "pub" 
自动化部署，element维护人员用来发布新版本，其中涉及
- `.sh`:shell脚本，通过.sh文件来执行命令。
- `faas`:功能即服务，无服务器，有兴趣可以自行了解。
- `algolia`:站内搜索，部署时会更新搜索数据。

#### "test"和"test:watch" 
利用`Karma`组件对代码进行不同浏览器下的测试。
 
## 阅读源码收获
- 认识一个成熟项目的整体架构
- 各种小组件，例如去除html标签，文件删除，剪切等
- 正则表达式的运用
- 初步认识glup,postcss,algolia,karma,typescript
- 加深对webpack认识，合理配置

## 接下来要研究
- `/package`中`element-ui`的各个组件的核心代码，
- `/test`中测试文件的编写及使用，
- `typescript`的使用。