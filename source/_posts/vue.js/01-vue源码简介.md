---
title: Vue源码简介
date: 2021-06-17 16:41:00
tags: 
  - Vue
categories: Vue
---
## 1. Flow
Flow 是 facebook 出品的 JavaScript 静态类型检查工具。Vue.js 的源码利用了 Flow 做了静态类型检查。
通常类型检查分成 2 种方式：
类型推断：通过变量的使用上下文来推断出变量类型，然后根据这些推断来检查类型。
类型注释：事先注释好我们期待的类型，Flow 会基于这些注释来判断。
## 2. Flow在Vue中的应用
- Flow 并不认识自定义类型或者第三方库，因此检查的时候会报错。为了解决这类问题，Flow 提出了一个 libdef 的概念，可以用来识别这些第三方库或者是自定义类型，而 Vue.js 也利用了这一特性。
- .flowconfig指定了flow的相关配置，主要包括8个section，内容如下
```
[include] // 除了.flowconfig之外其余外部需要watch的文件
[ignore]  // 忽略的文件
[untyped] // 不进行类型检查，直接认为是any。与ignore不同，配置部分会导致匹配的文件被模块解析器忽略，这本质上使它们未进行类型检查，也无法通过import或require进行解析。和declarations也不同，declarations只是提取函数或者类的签名，并不会进行类型检查。
[libs] // 自定义类型或者第三方库的目录
[lints]   // 规则
[options] // 配置
[version] // 指定flow的版本
[declarations] // declarations只是提取函数或者类的签名，并不会进行类型检查。
```
## 3. Vue源码目录
```
src
  ├── compiler        # 编译相关 
  ├── core            # 核心代码 
  ├── platforms       # 不同平台的支持
  ├── server          # 服务端渲染
  ├── sfc             # .vue 文件解析
  ├── shared          # 共享代码
```
**compiler**
compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。
**core**
核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。
**platforms**
Vue.js 是一个跨平台的 MVVM 框架，它可以跑在 web 上，也可以配合 weex 跑在 native 客户端上。platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。
**server**
服务端渲染主要的工作是把组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序
**sfc**
.vue文件解析
**shared**
浏览器端Vue.js和服务器端Vue.js共享
## 4. Vue构建过程
1. 从配置文件读取配置，再通过命令行参数对构建配置做过滤，这样就可以构建出不同用途的 Vue.js 。
2. 找到不同构建配置的入口文件
3. 拼接出入口文件的真实路径
4. 构建打包
## 5. Runtime Only VS Runtime + Compiler
**Runtime Only**
我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。

**Runtime + Compiler**
我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板

在 Vue.js 2.0 中，最终渲染都是通过 render 函数，如果写 template 属性，则需要编译成 render 函数，那么这个编译过程会发生运行时，所以需要带有编译器的版本。

