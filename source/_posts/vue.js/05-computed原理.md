---
title: computed原理
date: 2022-03-16 09:21:00
tags: 
  - Vue
categories: Vue
---
## 1. 使用方式
- 函数声明
- 对象声明
## 2. 工作流程
入口文件
```javascript
// 源码位置：/src/core/instance/index.js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

```
init.js
```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```
**initState**
```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  // 这里初始化computed计算属性
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
**initComputed**
```javascript
const computedWatcherOptions = { lazy: true }

function initComputed (vm: Component, computed: Object) {
  // $flow-disable-line
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()

  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      // 注册内部的订阅者
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      } else if (vm.$options.methods && key in vm.$options.methods) {
        warn(`The computed property "${key}" is already defined as a method.`, vm)
      }
    }
  }
}

```
1. 实例上定义 _computedWatchers 对象，用于存储“计算属性Watcher”
2. 获取计算属性的 getter，需要判断是函数声明还是对象声明
3. 创建“计算属性Watcher”，getter 作为参数传入，它会在依赖属性更新时进行调用，并对计算属性重新取值。需要注意 Watcher 的 lazy 配置，这是实现缓存的标识
4. defineComputed 对计算属性进行数据劫持

**defineComputed**
```javascript
const noop = function(){}
// 1
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  // 判断是否服务端渲染
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    // 2
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : createGetterInvoker(userDef)
    sharedPropertyDefinition.set = noop
  } else {
    // 3
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : createGetterInvoker(userDef.get)
      : noop
    sharedPropertyDefinition.set = userDef.set || noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  // 4
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```
1. sharedPropertyDefinition 是计算属性初始的属性描述对象
2. 计算属性使用函数声明时，设置属性描述对象的 get 和 set
3. 计算属性使用对象声明时，设置属性描述对象的 get 和 set
4. 对计算属性进行数据劫持，sharedPropertyDefinition 作为第三个给参数传入

客户端渲染使用 createComputedGetter 创建 get，服务端渲染使用 createGetterInvoker 创建 get。它们两者有很大的不同，服务端渲染不会对计算属性缓存，而是直接求值：
```javascript
function createGetterInvoker(fn) {
  return function computedGetter () {
    return fn.call(this, this)
  }
}
```
客户端渲染
```javascript
function createComputedGetter (key) {
  return function computedGetter () {
    // 1
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      // 2
      if (watcher.dirty) {
        watcher.evaluate()
      }
      // 3
      if (Dep.target) {
        watcher.depend()
      }
      // 4
      return watcher.value
    }
  }
}
```
1. 在上面的 initComputed 函数中，“计算属性Watcher”就存储在实例的_computedWatchers上，这里取出对应的“计算属性Watcher”
2. watcher.dirty 是实现计算属性缓存的触发点，watcher.evaluate 对计算属性重新求值
3. 依赖属性收集“渲染Watcher”
4. 计算属性求值后会将值存储在 value 中，get 返回计算属性的值

**Watcher**
```javascript
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          const info = `callback for watcher "${this.expression}"`
          invokeWithErrorHandling(this.cb, this.vm, [value, oldValue], this.vm, info)
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  /**
   * Depend on all deps collected by this watcher.
   */
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }
}
```
创建“计算属性Watcher”，配置的 lazy 为 true。dirty 的初始值等同于 lazy。所以在初始化页面渲染，对计算属性进行取值时，会执行一次 watcher.evaluate。
```javascript
evaluate() {
  this.value = this.get()
  this.dirty = false
}
```
求值后将值赋给 this.value，上面 createComputedGetter 内的  watcher.value 就是在这里更新。接着 dirty 置为 false，如果依赖属性没有变化，下一次取值时，是不会执行 watcher.evaluate 的， 而是直接就返回 watcher.value，这样就实现了缓存机制。

**更新逻辑**
依赖属性在更新时，会调用 dep.notify:
```javascript
notify() {
  this.subs.forEach(watcher => watcher.update())
}
```
然后执行watcher的update函数
```javascript
update() {
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    this.run()
  } else {
    queueWatcher(this)
  }
}
```
由于“计算属性Watcher”的 lazy 为 true，这里 dirty 会置为 true。等到页面渲染对计算属性取值时，符合触发点条件，执行 watcher.evaluate 重新求值，计算属性随之更新。

**依赖属性收集依赖**
初始化时，页面渲染会将“渲染Watcher”入栈，并挂载到Dep.target
在页面渲染过程中遇到计算属性，对其取值，因此执行 watcher.evaluate 的逻辑，接着调用 this.get:
```javascript
get () {
  // 1
  pushTarget(this)
  let value
  const vm = this.vm
  try {
    // 2
    value = this.getter.call(vm, vm) // 计算属性求值
  } catch (e) {
    if (this.user) {
      handleError(e, vm, `getter for watcher "${this.expression}"`)
    } else {
      throw e
    }
  } finally {
    popTarget()
    this.cleanupDeps()
  }
  return value
}
```
```javascript
Dep.target = null
let stack = []  // 存储 watcher 的栈

export function pushTarget(watcher) {
  stack.push(watcher)
  Dep.target = watcher
} 

export function popTarget(){
  stack.pop()
  Dep.target = stack[stack.length - 1]
}
```
1. pushTarget 轮到“计算属性Watcher”入栈，并挂载到Dep.target，此时栈中为 [渲染Watcher, 计算属性Watcher]
2. this.getter 对计算属性求值，在获取依赖属性时，触发依赖属性的 数据劫持get，执行 dep.depend 收集依赖（“计算属性Watcher”）

收集渲染Watcher
this.getter 求值完成后popTarget，“计算属性Watcher”出栈，Dep.target 设置为“渲染Watcher”，此时的 Dep.target 是“渲染Watcher”。
```javascript
if (Dep.target) {
  watcher.depend()
}
```
watcher.depend收集依赖：
```javascript
depend() {
  let i = this.deps.length
  while (i--) {
    this.deps[i].depend()
  }
}
```
deps 内存储的是依赖属性的 dep，这一步是依赖属性收集依赖（“渲染Watcher”）
经过上面两次收集依赖后，依赖属性的 subs 存储两个 Watcher，[计算属性Watcher，渲染Watcher]

Dep.target就是当前正在计算其表达式的订阅者，是 Watcher 的一个实例，且是全局唯一的。当某个 Watcher 调用它的get方法计算表达式时，就会将它自己设置为Dep.target，并将之前的Dep.target压入栈中。等到这个 Watcher 计算完表达式之后，会从栈中取出之前的Dep.target以重新作为当前的Dep.target


