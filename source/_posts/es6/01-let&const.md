---
title: let 和const 命令
date: 2021-03-24 16:23:30
tags: 
  - ES6
  - 块级作用域 
categories: ES6
---
## 1. let和var的区别
1. let声明的变量只在它所在的代码块有效。
2. 循环内定义的变量，只在循环体内有效，循环体外引用会报错
```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```
var命名的变量是全局有效的，每次循环都会改变i的值，循环内被赋给数组a的函数内部的console.log(i)，里面的i指向的就是全局的i。也就是说，所有数组a的成员里面的i，指向的都是同一个i，导致运行时输出的是最后一轮的i的值，也就是 10。

```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```
变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。
3. for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```
4. 不存在变量提升现象
5. 在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。
6. “暂时性死区”也意味着typeof不再是一个百分之百安全的操作。
```javascript
typeof x; // ReferenceError
let x;
```
```javascript
typeof undeclared_variable // "undefined"
```
7. let不允许在相同作用域内，重复声明同一个变量。

## 2. 块级作用域
ES5 只有全局作用域和函数作用域，没有块级作用域，这样会导致一些问题
问题1： 内层变量可能会覆盖外层变量
```javascript
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); // undefined
```
本意f是希望输出全局变量tmp的，但是在函数内部又定义了tmp，var方式定义会导致变量提升，这里实际输出的是未定义的内部tmp
问题2：计数的循环变量泄露未全局变量
```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```
i本来只用于控制循环，但是在循环结束后并未消失
let命令为JavaScript新增了块级作用域。
```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```
ES6允许块级作用域任意嵌套。
块级作用域的出现，实际上使得获得广泛应用的匿名立即执行函数表达式（匿名 IIFE）不再必要了。
```javascript
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```
ES5规定函数只能做顶层作用域和函数作用域之中声明，不能在块级作用域声明。但是浏览器为了兼容旧代码，也支持。
考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。
```javascript
// 块级作用域内部的函数声明语句，建议不要使用
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 块级作用域内部，优先使用函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```
ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。
```javascript
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```
第一种写法没有大括号，所以不存在块级作用域，而let只能出现在当前作用域的顶层，所以报错。第二种写法有大括号，所以块级作用域成立。

## 3. const 命令
1. const 声明一个只读常量，一旦声明，常量的值就不会改变。
2. const 的作用域与let命令相同：只在声明所在的块级作用域内有效。
3. const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。
4. const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。
```javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```
如果真的想将对象冻结，应该使用Object.freeze方法。
```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
5. ES5 只有两种声明变量的方法：var命令和function命令。ES6 除了添加let和const命令，还有import命令和class命令。因此ES6一共有6种声明变量的方式。

## 4. 顶层对象的属性
顶层对象，在浏览器中指的是window对象，在Node中指的是global对象。ES5中，顶层对象的属性与全局变量等价。
顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。
这样的设计带来了几个很大的问题：
1、没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；
2、程序员很容易不知不觉地就创建了全局变量（比如打字出错）；
3、顶层对象的属性是到处可以读写的，这非常不利于模块化编程。
4、window对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。
ES6规定var和function命令声明的全局变量，依旧是顶层对象的属性。let、const和class命令声明的全局变量，不属于顶层对象的属性

## 5. globalThis对象
JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的。
- 浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
- 浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
- Node 里面，顶层对象是global，但其他环境都不支持。
- 全局环境中，this会返回顶层对象。但是，Node.js 模块中this返回的是当前模块，ES6 模块中this返回的是undefined。
- 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
```javascript
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```