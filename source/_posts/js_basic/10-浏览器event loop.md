---
title: 浏览器event loop
date: 2022-04-08 22:40:13
tags: 
  - event loop
categories: JavaScript
---
### 1. 作用
js是一门单线程的脚本语言，有很多异步的API来帮助开发者实现线程的阻塞问题。通过eventloop来实现不阻塞线程等待各种异步操作完成才继续执行操作。
> 1. event loop的规范是HTML5中规定的。
> 2. event loop是javascript**运行环境**的机制。
> 3. 浏览器实现的event loop和NodeJS实现的event loop是有差异的。

### 2. 基础概念
##### 2.1 宏任务和微任务
> **MacroTask(宏任务)**：dom对象生成、解析HTML、执行主线程js代码、更改当前URL、页面加载、输入、网咯时间和定时器事件。macroTask可以对浏览器而言，代表一些离散的独立的工作，完成一个task之后，可以继续其他的工作（重新渲染和垃圾回收）

> **MicroTask(微任务)**：promise的回调、dom的修改。这是一些更新应用程序状态的较小任务，Microtask的执行开销比执行一个新的Macrotask要小，因此应该以异步的方式尽快执行。Microtask可以在UI渲染之前执行某些任务，免得不必要的UI渲染，可能会导致现实的应用程序状态不一致。

#### 2.2 管理方式

浏览器通过至少一个MacroTask队列和一个MicroTask队列来实现对宏任务和微任务的队列，也有浏览器通过多个队列处理不同类型的宏任务和微任务

#### 2.3 基本原则
- 同一个时间点只能执行一个任务。
- 任务一直执行到完成，不允许被其他任务打断。

### 3. 流程
#### 3.1 基础流程
![event loop](https://doublewxx.github.io/images/event_loop_01.png)
1.javascript 引擎执行 javascript 是单线程的，因为只有一个 stack 里面有各种正在执行、等待执行的事件。
2.有一些 webAPI 将执行时产生的 callback 放入一个队列，即 “事件队列”。
3.在event loop 循环中不停的将“事件队列”里等待执行的事件，推入 javascript 执行栈

#### 3.2 详细流程
![event loop](https://doublewxx.github.io/images/event_loop_02.png)
1. 读取Macrotask queue中的任务
   - 任务队列空，继续向下执行
   - 任务队列不为空，最先进入的一个任务推入执行栈，向下执行。
2. 读取Microtask queue中的任务
   - 任务队列空，向下执行
   - 任务队恶劣不为空，将最先进入任务推入执行栈，再次重复此操作直到Microtask queue为空。就是将此任务队列按照入队先后顺序将所有任务推入js执行栈，向下执行。
3. 根据本次循环的耗时判断是否需要、是否可以更新UI
   - 不需要，回到第一步
   - 需要，继续向下
4. 更新UI，UI rendering，同时阻塞js执行
5. 回到第一步

#### 3.3 补充细节
- 两个任务队列都不在event loop内，将任务添加和任务处理行为分离。event loop内只负责执行任务并从队列里删除，而外部添加任务。否则event loop执行任务的时候，发生任何事件都会被忽略。
- 浏览器通常一秒尝试渲染页面60次，以达到60fps，这个帧速率被认为是平滑运动的选择。浏览器中update rendering的操作在事件循环中进行，是因为在呈现页面的时候，页面内容不能被另一个任务修改。也意味着如果想要平滑渲染，单个事件循环中，不能占据太多时间。

当浏览器完成页面渲染后，下次event loop循环迭代的时候，可能有三种情况
- event loop执行完成，耗时不到16ms，此时执行is rendering needed就不会明确要求渲染页面，更新UI是复杂操作还会阻塞js的执行，本次loop不执行UI渲染
- event loop执行完成，耗时大约16ms，此时执行is rendering needed就明确要求渲染页面，更新UI
- event loop本次循环的任务(one macrotask and all microtask)花费时间大大超过16ms，此时UI无法按照目标速度更新。如果超过几百毫秒，用户感觉不会太明显，但是如果过长，浏览器会显示“无响应脚本”。