---
title: node事件循环各个阶段
date: 2022-03-24 16:30:59
tags: 
  - 事件机制
  - event loop
categories: Node
---
### 1. 阶段
Event loop 包含一系列阶段 (phase)，每个阶段都是只执行属于自己的的任务 (task) 和微任务 (micro task)，这些阶段依次为：

- timers：执行到期的setTimeout / setInterval队列回调
- pending callbacks：执行一些系统调用错误，比如网络通信的错误回调
- idle, prepare：这个阶段仅在内部使用，对外不可见
- 轮询阶段poll：阶段基本上涵盖了剩下的所有的情况，你写的大部分回调（I/O，HTTP回调），如果不是上面两种（还要除掉 micro task，后面会讲），那基本上就是在 poll 阶段执行的，当没有要执行的setImediate同时也没有要执行的timers的时候，会阻塞在这个状态
- check：check 阶段和 timer 类似，当你使用 setImmediate() 函数的时候，传入的回调函数就是在 check 阶段执行。
- close callbacks：socket.close在这里

#### 1.1 timers阶段
 timers是事件循环的第一个阶段，node会去检查有无过期的timer，如果有则把它的回调压入timer的任务队列中等待执行，事实上，node并不能保证timer在预设时间到了就会立即执行，因为node对timer的过期检查不一定靠谱，它会受到机器上其他运行程序的影响，或者那个时间点主线程不空闲。比如下面的代码，setTimeout()和setImmediate()的执行顺序是不确定的。
 ```javascript
 setTimeout(()=>{
     console.log('settimeout');
 }, 0);
 setImmediate(()=>{
     console.log('setimmediate');
 },0)
 ```
 对于以上代码来说，setTimeout 可能执行在前，也可能执行在后。
首先 setTimeout(fn, 0) === setTimeout(fn, 1)，这是由源码决定的
进入事件循环也是需要成本的，如果在准备时候花费了大于 1ms 的时间，那么在 timer 阶段就会直接执行 setTimeout 回调
如果准备时间花费小于 1ms，那么就是 setImmediate 回调先执行了。但是把它们放到一个I/O回调里面，一定是setImmediate()先执行，因为poll阶段后面就是check阶段。

在 Node 中定时器指定的时间也不是准确时间，只能是尽快执行。

 #### 1.2 轮询阶段
 轮询阶段两个功能
1. 计算应该阻塞和轮询I/O的时间
2. 处理轮询队列里的事件

当事件循环进入 轮询 阶段且 没有被调度的计时器时 ，将发生以下两种情况之一：
    - 如果 轮询 队列 不是空的 ，事件循环将循环访问回调队列并同步执行它们，直到队列已用尽，或者达到了与系统相关的硬性限制。（就是避免timer被饿死的限制）
    - 如果 轮询 队列 是空的 ，还有两件事发生：
      - 如果脚本被 setImmediate() 调度，则事件循环将结束 轮询 阶段，并继续 检查 阶段以执行那些被调度的脚本。
      - 如果脚本 未被 setImmediate()调度，则事件循环将等待回调被添加到队列中，然后立即执行。
   
如果没有setImmediate()将会导致event loop阻塞在poll阶段，这样之前设置的timer岂不是执行不了了？因此在poll阶段event loop会有一个检查机制，检查timer队列是否为空，如果timer队列非空，event loop就开始下一轮事件循环，即重新进入到timer阶段。
为了防止 轮询 阶段饿死事件循环，libuv（实现 Node.js 事件循环和平台的所有异步行为的 C 函数库），在停止轮询以获得更多事件之前，还有一个硬性最大值（依赖于系统）

#### 1.3 check阶段
此阶段允许人员在轮询阶段完成后立即执行回调。如果轮询阶段变为空闲状态，并且脚本使用 setImmediate() 后被排列在队列中，则事件循环可能继续到 检查 阶段而不是等待。

setImmediate() 实际上是一个在事件循环的单独阶段运行的特殊计时器。它使用一个 libuv API 来安排回调在 轮询 阶段完成后执行。
setImmediate()的回调会被加入到check队列中，从event loop的阶段图可以知道，check阶段的执行顺序会在poll阶段之后。

#### 1.4 每个阶段的microtask阶段
每个阶段还有一个 microtask 的阶段。这个阶段就是我们下面主要要讲的 process.nextTick 和 Promise 的回调函数运行的地方。
那么每个阶段就包括以下操作：
1. 这个阶段的"同步" task queue
2. 这个阶段的 process.nextTick 的 queue
3. 这个阶段的 Promise queue
Process.nextTick
这是因为 process.nextTick() 从技术上讲不是事件循环的一部分。相反，它都将在当前操作完成后处理 nextTickQueue， 而不管事件循环的当前阶段如何。这里的一个操作被视作为一个从底层 C/C++ 处理器开始过渡，并且处理需要执行的 JavaScript 代码。
#### 1.4 小结
1. event loop的每个阶段都有一个任务队列。
2. event loop到达某个阶段时，将执行该阶段的任务队列，直到队列清空或执行的回调达到系统上限后，才会转入下一个阶段。
3. 当所有阶段被顺序执行一次后，称event loop完成了一个tick。
4. process.nextTick本质上属于Micro Task，但是它先于所有其他的Micro Task 执行；
```javascript
const fs = require('fs');
fs.readFile('app.js',()=>{
    console.log('readFile');
    setTimeout(()=>{
        console.log('settimeout');
    }, 0);
    setImmediate(()=>{
        console.log('setimmediate');
    },0)
})
// readFile
// setImmediate
// settimeout
```
- Node 端，microtask 在事件循环的各个阶段之间执行
- 浏览器端，microtask 在事件循环的 macrotask 执行完之后执行

```javascript
setTimeout(()=>{
    console.log('timer1')
    Promise.resolve().then(function() {
        console.log('promise1')
    })
}, 0)
setTimeout(()=>{
    console.log('timer2')
    Promise.resolve().then(function() {
        console.log('promise2')
    })
}, 0)
// node11之后的输出是这个，与浏览器一致
// timer1
// promise1
// timer2
// promise2
// node11之前是
// timer1
// timer2
// promise1
// promise2
```