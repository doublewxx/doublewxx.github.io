---
title: AST语法树
date: 2021-08-16 16:23:11
tags: 
  - 基础知识
categories: JavaScript
---
### 1. 什么是AST语法树
AST是抽象语法树（Abstract Syntax Tree），是源代码的抽象语法结构的树状表现形式。webpack、eslint 等很多工具库的核心都是通过抽象语法书这个概念来实现对代码的检查、分析等操作
常用的浏览器就是通过将 js 代码转化为抽象语法树来进行下一步的分析等其他操作。
以下方代码为例子
```javascript
var a=7;
```
上述代码转换为AST之后，就是
```javascript
{
  "type": "Program", // 顶级type属性
  "body": [ // 数组，每个元素是一个对象，里面包含了所有的对于该语句的描述信息
    {
      "type": "VariableDeclaration", // 描述该语句的类型  --> 变量声明的语句
      "declarations": [ // 声明内容的数组，里面每一项也是一个对象
        {
          "type": "VariableDeclarator", // 描述该语句的类型
          "id": { // 描述变量名称的对象
            "type": "Identifier", // 定义
            "name": "AST" // 名字
          },
          "init": { // 初始化变量值的对象
            "type": "Literal", // 类型
            "value": "is Tree", // 值不带引号
            "raw": "'is Tree'"  // 值带引号
          }
        }
      ],
      "kind": "var" // 变量声明的关键字  --> var
    }
  ],
  "sourceType": "script"
}
```
### 2. 语法分析和词法分析
JavaScript是解释型语言，一般通过 词法分析 -> 语法分析 -> 语法树，就可以开始解释执行了
**词法分析**：也叫扫描，是将字符流转换为记号流(tokens)，它会读取我们的代码然后按照一定的规则合成一个个的标识
比如说：var a = 2 ，这段代码通常会被分解成 var、a、=、2
```javascript
;[
  { type: 'Keyword', value: 'var' },
  { type: 'Identifier', value: 'a' },
  { type: 'Punctuator', value: '=' },
  { type: 'Numeric', value: '2' },
]
```
当词法分析源代码的时候，它会一个一个字符的读取代码，所以很形象地称之为扫描 - scans。当它遇到空格、操作符，或者特殊符号的时候，它会认为一个话已经完成了。
**语法分析**：也称解析器，将词法分析出来的数组转换成树的形式，同时验证语法。语法如果有错的话，抛出语法错误。
### 3. AST用途
- 语法检查、代码风格检查、格式化代码、语法高亮、错误提示、自动补全等
- 代码混淆压缩
- 优化变更代码，改变代码结构等

### 4. AST解析流程
```javascript
const esprima = require('esprima')
const estraverse = require('estraverse')
const code = `function getUser() {}`
// 生成 AST
const ast = esprima.parseScript(code)
// 转换 AST，只会遍历 type 属性
// traverse 方法中有进入和离开两个钩子函数
estraverse.traverse(ast, {
  enter(node) {
    console.log('enter -> node.type', node.type)
  },
  leave(node) {
    console.log('leave -> node.type', node.type)
  },
})
```
经过钩子函数可以确认，AST遍历的流程是深度优先
escodegen可以将一棵AST转换为代码，所以可以直接修改函数名
```javascript
// 转换树
estraverse.traverse(ast, {
  // 进入离开修改都是可以的
  enter(node) {
    console.log('enter -> node.type', node.type)
    if (node.type === 'Identifier') {
      node.name = 'hello'
    }
  },
  leave(node) {
    console.log('leave -> node.type', node.type)
  },
})
// 生成新的代码
const result = escodegen.generate(ast)
console.log(result)
// function hello() {}
```
### 5. babel使用AST进行语法编译
自从 Es6 开始大规模使用以来，babel 就出现了，它主要解决了就是一些浏览器不兼容 Es6 新特性的问题，其实就把 Es6 代码转换为 Es5 的代码，兼容所有浏览器，babel 转换代码其实就是用了 AST，babel 与 AST 就有着很一种特别的关系。babel 核心包并不会去转换代码，核心包只提供一些核心 API，真正的代码转换工作由插件或者预设来完成，用到哪个插件就用哪个插件，但是你无法预设你会用到哪些插件，因此会使用presets做预设。presets 是 plugins 的集合，一个 presets 内部包含了很多 plugin。
可以使用babel将一个箭头函数转为普通函数
```javascript
const babel = require('@babel/core')
const t = require('@babel/types')
const code = `const fn = (a, b) => a + b` // const fn = function(a, b) { return a + b } 简写模式
const arrowFnPlugin = {
  // 访问者模式
  visitor: {
    // 当访问到某个路径的时候进行匹配
    ArrowFunctionExpression(path) {
      // 拿到节点然后替换节点
      const node = path.node
      // console.log("ArrowFunctionExpression -> node", node)
      // 拿到函数的参数
      const params = node.params
      let body = node.body
      console.log(body)
      // 判断是不是 blockStatement，不是的话让他变成 blockStatement
      if (!t.isBlockStatement(body)) {
        let returnStatement = t.ReturnStatement(body) // 简写模式，因此需要做一次转换，转成带return的BlockStatement
        body = t.blockStatement([returnStatement])
      }
      const functionExpression = t.functionExpression(null, params, body)
      // 替换原来的函数
      path.replaceWith(functionExpression)
    },
  },
}
const r = babel.transform(code, {
  plugins: [arrowFnPlugin],
})
console.log(r.code) 
```
Babel执行过程
1. 解析：词法分析和语法分析。词法分析：词法分析阶段把字符串形式的代码转换为令牌（tokens） 流。你可以把令牌看作是一个扁平的语法片段数组。语法分析：语法分析阶段会把一个令牌流转换成 AST 的形式。
2. 转换（transform）：接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程，同时也是插件将要介入工作的部分。
3. 生成（generate）：代码生成步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建源码映射（source maps）。


