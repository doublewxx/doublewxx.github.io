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
- Node以8KB为界限区分Buffer是大对象还是小对象
- 小对象在分配的时候，中间变量为pool，然后一次分配8KB的slab，used为0，分配给一个小对象。等再次申请小对象的时候，如果剩余空间足够，继续分配。不够则申请新的8KB进行分配。一块slab只有所有的小对象Buffer对象在作用域释放可回收时，才会被回收
- 大对象在分配的时候，直接分配一个大对象的长度为slab单元
- Buffer对象是JS层面的，能被V8标记收回，但是内部是指向C++的实现，内存在堆外
- Buffer类型与string类型是不一样的，在转换的时候支持多种编码方式的转换，但是要注意宽字节的转换因为截断会出现乱码的问题。
