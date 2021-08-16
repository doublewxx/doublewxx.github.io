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
