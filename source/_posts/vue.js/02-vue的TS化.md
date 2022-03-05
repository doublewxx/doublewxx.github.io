---
title: VueTS化
date: 2022-03-05 11:34:00
tags: 
  - Vue
categories: Vue
---
## 1. TS化的好处
- 本身后端开发使用的TS语言，日常开发中经常需要前后端共享枚举状态值、返回码等信息。当后端修改枚举值时，需要同步前端手动修改。
- TS语言更熟悉，整体TS化有利于提高后端开发的积极性，培养全栈能力。
- TS通过引入静态类型声明，减少不必要的类型判断和文档注释，有利于提高整体代码质量，及早发现错误，在静态类型检查或者编译时就发现问题。

## 2. 方案选择
### 2.1 Vue组件的TS化
**vue-property-decorator**

Github：https://github.com/kaorun343/vue-property-decorator

介绍：依赖vue-class-component库，Vue官方提供的使用Class风格语法编写Vue组件的库，可以使vue组件可以使用继承、混入等高级特性。

### 2.2 Vuex的TS化
**vuex-module-decoratoers**
Github：https://github.com/championswimmer/vuex-module-decorators
官方文档：https://championswimmer.in/vuex-module-decorators/

介绍：vuex-module-decorator可以让开发者以TS语法来编写vuex状态管理逻辑，提供js-ts的语法元素映射，帮助讲vuex的js语法概念映射到ts的语法概念。

## 3.实施过程
1. 全局安装vue cli
2. vue add typescript
3. 修改tsconfig配置支持路径自动匹配
4. 修改vue.config.js支持vuex decorators，加入transpileDependencies配置
5. 修改script标签后面加上lang='ts'
6. 修改js到ts
    - 引入decorater，使用component装饰器
    - 将data修改为成员变量
    - 将props修改为@Props装饰器
    - 迁移methods部分的代码
    - 迁移computed为get method，对于有mapState和mapActiond，需要在计算属性上面加上重定义的mapAction，定义在组件类的外面，然后在里面进行使用
    - 修改watch为@Watch
    - 验证迁移结果
备注：
vue也有一定的重构辅助工具，vscode就有插件Vue Language Features，但实际上只是将组件内用到的父组件的所有东西都作为属性穿进去，自动生成的代码，可维护性并不够好