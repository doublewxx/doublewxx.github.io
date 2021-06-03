---
title: JS执行机制
date: 2021-03-09 17:25:30
tags: 
  - 执行机制
  - 基础知识
categories: JavaScript
---
### 1. 单线程
javascript是一门单线程语言，在最新的HTML5中提出了Web-Worker，但javascript是单线程这一核心仍未改变。
### 2. JavaScript的事件循环
``` plantuml
start
:任务进入执行栈;
if (是否是同步任务) then (是)
    :主线程;
    :任务全部执行完毕;
else (否)
    :Event Table;
    :Event Queue;
endif
:读取任务队列中的结果，进入主线程执行;
@enduml
```
流程描述：
- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
- 主线程内的任务执行完毕为空，去Event Queue读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)。

### 3. setTimeOut
setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行。即便主线程为空，0毫秒实际上也是达不到的。根据HTML的标准，最低是4毫秒。
### 4. setInterval
setInterval会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。因为是每隔ms秒会有回调函数进入Event Queue，一旦setInterval的回调函数fn执行时间超过了延迟时间ms，就看不出时间差异了。
### 5. Promise与process.nextTick(callback)
- macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
- micro-task(微任务)：Promise，process.nextTick
不同任务进入不同的Event Queue
进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。

### 6. 备注
执行和运行有很大的区别，javascript在不同的环境下，比如node，浏览器，Ringo等等，执行方式是不同的。而运行大多指javascript解析引擎，是统一的。