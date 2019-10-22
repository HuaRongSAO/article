# vue-next 源码阅读笔记： 项目结构 monorepo 风格

![img](./asset/lerna.png)

## vue-next 采用的 monorepo 的风格

Monorepo(monolithic repository) 开源项目代码管理的一种方式，他解决了传统的 multirepo(one-repository-per-module)的一些痛点。能更好的管理大型的前端开源项目。

## monorepo VS multirepo

multirepo 管理方式，也就是传统做法，按项目大模块拆分为多个代码库，然后在主仓库再去依赖其他子仓库，每当子模块有更新，就需要通知主模块更新。  
于是就出现很多问题：

- issue 管理混乱，经常有在 core repo 提 module 问题的。就是我有问题，但是我不知道在那个仓库提 issue
- changelog 不好管理，每当有更新，依赖之间的修改复杂。

monorepo 管理方式：将一个项目的核心模块全部都放在一个仓库下，每个 module 单独发布，但使用与该 repo 统一的版本号（例如 React, Angular, Babel, Google）。  
优点：

- issue 和 PR 都集中到该 repo，就不会出现有问题找不到提 issue 的仓库
- changelog 可以之间从整个仓库中梳理清楚，提炼出来
- 模块之间的联系更清晰，调试开发更方便，不需要`npm link`

缺点:

- 项目体积太大
- 对构建工具和项目的组织能力有更高的要求

总结：  
multirepo 对子仓库的管控更灵活，又可以有多样化的结构，不同的打包工具，不同的风格，依赖，版本管理，测试。但从趋势来说，google，babel，facebook 都选择 monorepo，就足以说明一切了。monorepo 有更好的工作流，更好的沟通方式，更好的版本控制，带来的效益是远远超过了缺点的。

## 典型的 monorepo

每个 module 都在 packages 下有自己的目录仓库， 都有自己的依赖项（package.json），能够作为独立的 npm package 发布，所有源码放在一起维护

```shell
vue
├── packages # 项目分包
│   ├── template-explorer
│   │   ├── index.html
│   │   ├── package.json
│   │   ├── README.md
│   │   ├── src
│   │   │   ├── index.ts
│   └── vue
│       ├── api-extractor.json
│       ├── dist
│       │   └── vue.global.js
│       ├── index.js
│       ├── package.json
│       ├── README.md
│       ├── src
│       │   └── index.ts
│       └── __tests__
│           └── index.spec.ts
├── tsconfig.json
├── .eslintrc
├── node_modules
└── package.json
```

## lerna: monorepo 的生成管理工具

> Lerna 是 monorepo 的流程工具，方便于生成，开发，发布，更新开源仓库的方案

怎么使用？

```shell
mkdir lerna-repo &&  cd  lerna-repo
npm install -D lerna
npx lerna init
tree .
# lerna-repo/
#    ├── lerna.json  # lerna 配置文件
#    ├── packages  # 模块
#    └──  package.json
npx lerna create @vue/core # 创建子模块 core
npx lerna create @vue/runtime # 创建子模块 runtime
tree .
# lerna-repo/
#     ├── lerna.json
#     ├── package.json
#     └── packages
#         ├── core
#         │   ├── lib
#         │   │   └── core.js
#         │   ├── package.json
#         │   ├── README.md
#         │   └── __tests__
#         │       └── core.test.js
#         └── runtime
#             ├── lib
#             │   └── runtime.js
#             ├── package.json
#             ├── README.md
#             └── __tests__
#                 └── runtime.test.js
npx lerna version # 打版本
npx lerna public # 一键发布
```

支持 yarn 的 workspaces 模式：  
结合 yarn 的 workspaces 你可以更友好的管理你的项目依赖关系
[yarn workspaces](https://yarnpkg.com/zh-Hans/docs/workspaces)

## 分析 vue-next 的目录结构

```shell
.
├── packages # 源码目录
│   ├── compiler-core # 编译器核心
│   ├── compiler-dom # dom编译器
│   ├── reactivity # 响应
│   ├── runtime-core # runtime 核心
│   ├── runtime-dom # runtime dom
│   ├── runtime-test # runtime 测试
│   ├── server-renderer # 服务器渲染
│   ├── shared # 工具包
│   ├── template-explorer # 模板浏览器
│   │   ├── index.html
│   │   ├── package.json
│   │   ├── README.md
│   │   ├── src
│   │   │   ├── index.ts
│   │   │   └── options.ts
│   │   └── style.css
│   └── vue # 项目主文件
│       ├── api-extractor.json
│       ├── dist
│       │   └── vue.global.js
│       ├── index.js
│       ├── package.json
│       ├── README.md
│       ├── src
│       │   └── index.ts
│       └── __tests__
│           └── index.spec.ts
├── scripts # 构建脚本目录
│   ├── bootstrap.js
│   ├── build.js
│   ├── dev.js
│   ├── utils.js
│   └── verifyCommit.js
│
├── api-extractor.json # 	TypeScript 的API提取和分析工具 api-extractor 的配置文件
├── jest.config.js # JavaScript 测试框架 jest 的配置文件
├── lerna.json # 	JavaScript 多 package 项目管理工具 lerna 的配置文件
├── package.json # npm 配置文件
├── rollup.config.js # JavaScript 模块打包器 rollup 的配置文件
├── README.md # 项目介绍
├── tsconfig.json # 	TypeScript 配置文件
└── yarn.lock
```

## 如何开发运行

```shell
# 安装monorepo管理工具
npm install -g lerna

# 下载项目
git clone https://github.com/vuejs/vue-next

cd vue-next
# 输出包结构
lerna ls
# 输出目录结构
tree -I "*.ts" -L 1 -C packages
# 创建 packages 的符号链接
lerna bootstrap
# 启动开发
yarn dev
# 编译
yarn build
# 查看文件大小
yarn size-runtime
yarn size-compiler
yarn size
# 代码规范
yarn lint
# 单元测试
yarn test

```
