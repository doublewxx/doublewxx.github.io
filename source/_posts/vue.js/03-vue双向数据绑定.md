---
title: 双向数据绑定
date: 2022-03-08 11:34:00
tags: 
  - Vue
categories: Vue
---
## 1. MVC和MVVM
MVC：Model绑定到View，当我们用JavaScript代码更新Model时，View就会自动更新。即Model，View和Controller之间，形成单向数据流动。Model的更新会更新View，View的更新通过Controller再更新到Model。
MVVM：MVVM模式就是Model–View–ViewModel模式。它实现了View的变动，自动反映在 ViewModel，反之亦然。对于双向绑定的理解，就是用户更新了View，Model的数据也自动被更新了，这种情况就是双向绑定。

## 2. 实现原理
### 2.1 原理
Vue数据双向绑定是通过数据劫持结合发布者-订阅者模式的方式来实现的。Vue2.X的版本，是通过 ES5 提供的 Object.defineProperty() 方法来劫持（监听）各属性的 getter、setter，并在当监听的属性发生变动时通知订阅者，是否需要更新，若更新就会执行对应的更新函数。
Object.defineProperty()函数定义：Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。
语法：Object.defineProperty(obj, prop, descriptor)
参数：
obj：要定义属性的对象。
prop：要定义或修改的属性的名称或 Symbol 。
descriptor：要定义或修改的属性描述符。
属性描述符可以包含的属性：

```javascript
{
  value: undefined,   // 属性的值
  get: undefined,     // 获取属性值时触发的方法
  set: undefined,     // 设置属性值时触发的方法
  writable: false,    // 属性值是否可修改，false不可改
  enumerable: false,  // 属性是否可以用for...in 和 Object.keys()枚举
  configurable: false // 该属性是否可以用delete删除，false不可删除，为false时也不能再修改该参数
}
```

```javascript
var obj = {    
    name:'tset'
}

Object.defineProperty(obj, "name",{
    get(){        
        console.log("get方法被触发");
        dep.depend(); // 这里进行依赖收集
        return value;
    },
    set(val){        
        console.log("set方法被触发");
        value = newValue;
        // self.render();
        dep.notify();  // 这里进行virtualDom更新，通知需要更新的组件render
    }
})

var str = obj.name;  // get方法被触发
obj.name = "test2";  // set方法被触发
```

### 2.2 实现流程

1. 实现一个监听器Observer，用来劫持并监听所有属性，如果有变动的，就通知订阅者。
2. 实现一个订阅者Watcher，每一个Watcher都绑定一个更新函数，watcher可以收到属性的变化通知并执行相应的函数，从而更新视图。
3. 实现一个解析器Compile，可以扫描和解析每个节点的相关指令，如果节点存在{% raw %}v-model，v-on{% raw %}等指令，则解析器Compile初始化这类节点的模板数据，使之可以显示在视图上，然后初始化相应的订阅者（Watcher）。

#### 2.2.1 实现Observer
就是将组件的data的所有属性通过Object.defineProperty绑定上去，在数据的get时添加订阅者，set的时候通知订阅者。

#### 2.2.2 实现Watcher
Watcher的getter函数中，Dep.target指向了自己，也就是Watcher对象。在getter函数中把watcher添加到了订阅器中。

#### 2.2.3 实现compiler
Compile主要的作用是把new SelfVue 绑定的dom节点，（也就是el标签绑定的id）遍历该节点的所有子节点，找出其中所有的v-指令和" {{}} ".

（1）如果子节点含有v-指令，即是元素节点，则对这个元素添加监听事件。（如果是v-on，则node.addEventListener('click'），如果是v-model，则node.addEventListener('input'))。接着初始化模板元素，创建一个Watcher绑定这个元素节点。
（2）如果子节点是文本节点，即" {{ data }} ",则用正则表达式取出" {{ data }} "中的data，然后var initText = this.vm[exp]，用initText去替代其中的data。xp是node节点的v-model或v-on：click等指令的属性值。


1. Object.defineProperty - get ，用于 依赖收集
2. Object.defineProperty - set，用于 依赖更新
3. 每个 data 声明的属性，都拥有一个的专属依赖收集器 Dep.subs
4. 依赖收集器 subs 保存的依赖是 watcher
5. watcher可用于 进行视图更新