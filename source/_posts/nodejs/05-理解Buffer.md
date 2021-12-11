---
title: 理解Buffer
date: 2021-12-05 11:17:21
tags: 
  - Buffer类型
categories: Node
---
### 1. Buffer结构
- Buffer是一个JS与C++结合的模块，性能部分用C++实现，非性能部分用JS实现
- Buffer所占用的内存不是由V8分配，属于堆外内存，由Node在C++层面实现内存的申请。因为处理大量的字节数据不能采用需要一点内存就跟操作系统申请一点内存的方式，在JS中分配内存
- Node在进程启动时就加载了Buffer并放在全局对象上，无需require
- Buffer对象也有length属性，初始化Buffer的时候，每一位的值都是随机的0-255
- slab分配机制是动态内存管理机制。slab就是一块申请好的固定大小的内存区域，有三种状态，full完全分配状态，partial部分分配状态，empty没有被分配的状态。
- 