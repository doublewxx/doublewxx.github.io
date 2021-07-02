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
