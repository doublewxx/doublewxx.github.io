---
title: Symbol
date: 2021-05-24 20:55:00
tags: 
  - ES6
  - Symbol
categories: ES6
---
## 1. Symbol概述
- Symbol是一种新的原始数据类型，用于防止属性名的冲突，通过Symbol函数生成。
- 对象的属性名可以有两种类型，Symbol和原来的字符串。
- Symbol函数前不能使用new命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。
- Symbol接受字符串或者对象为参数，调用对象的toString方法，将其转为字符串，然后再生成一个Symbol值。
- Symbol函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的。
- Symbol 值不能与其他类型的值进行运算，但是可以显式转化为字符串，也可以转化为布尔值，但是不能转化为数字。
## 2. Symbol.prototype.description
```javascript
const sym = Symbol('foo');
sym.description // "foo"
```
## 3. Symbol做属性名
- 对象使用Symbol值定义属性时，必须放在方括号中
- Symbol 值作为对象属性名时，不能用点运算符，点运算符后面只能是字符串
- Symbol 值作为属性名时，该属性还是公开属性，不是私有属性
## 4. 消除魔术字符串
一些对象中的值的定义，如果只是为了不能和对象其他属性值冲突，可以用Symbol
## 5. 属性名的遍历
- Symbol 作为属性名，遍历对象的时候，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。
- 它也不是私有属性，有一个Object.getOwnPropertySymbols()方法，可以获取指定对象的所有 Symbol 属性名。该方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。由于以 Symbol 值作为键名，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。
## 6. Symbol.for()，Symbol.keyFor()
- Symbol.for()接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局。
- Symbol.keyFor()方法返回一个已登记的 Symbol 类型值的key。
- Symbol.for()为 Symbol 值登记的名字，是全局环境的，不管有没有在全局环境运行。
## 7. 模块的 Singleton 模式
- Singleton 模式指的是调用一个类，任何时候返回的都是同一个实例。