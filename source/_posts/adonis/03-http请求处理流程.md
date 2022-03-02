---
title: http请求处理流程
date: 2022-03-01 22:33:58
tags: 
  - adonis
categories: Node
---
### 1. 调用server的handle函数
handle函数执行流程：
1. 找到对应的上下文信息
2. 运行所有before的hooks，静态文件处理就在hook里面处理
3. 最后再执行requestHandle里面的handle，解析url、method和host，找到对应的路由。
4. 通过commit调用到所有的middleware。
5. middleware执行完成所有的中间件之后，finalHandler就是route里面绑定的该路由实际的处理函数，处理函数执行完成后，再回调middleware，从而形成洋葱模型。

### 2. 静态文件处理
静态文件是挂在在hook上面，加载了一个staticHook，无论是请求函数还是静态文件，都会先走hook，找到了文件的时候，修改hasLazyBody这个属性，就在执行后返回true，直接调用executeAfter函数，触发提前返回。static目录配置在rcFile里，可以进行修改。
如果一个静态文件的命名和路由冲突，会直接返回静态文件，相应的路由并不会被调用。

### 3. middleware洋葱模型
1. handleRequest函数，每次通过runner()函数返回一个新的Runnable，Runnable里面的executor，绑定了所有要执行的middleware，再把finalHandle(也就是路由匹配找到的处理业务逻辑的controller)绑定上去，就可以调用Runnalbe上面的run函数了。
2. run函数讲绑定了新的this指向的invoke打进参数里面，然后执行invoke。这里在执行的时候，params里面的参数包括了invoke，在handle里面作为next函数传入进去。这个getInjections会将组合的的args传入过去作为handle的参数。因为this被修改为Runnable当前的执行环境，每一个middleware在执行的时候，一旦调用next，也就是重新调用invoke函数，则会取出下一个middleware继续执行，上一个等待，等到所有的middleware都执行的前序部分操作后，交由finalHandle执行业务逻辑，执行完成后再一层一层向上进行回调，实现洋葱模型。