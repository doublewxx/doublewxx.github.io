---
title: JS类型转换
date: 2021-04-21 20:46:00
tags: 
  - 类型转换
categories: JavaScript
---
### 1. 转换为字符串
- ECMAScript 的 Boolean 值、数字和字符串的原始值是伪对象，实际上具有属性和方法，都有toString()方法
- Boolean的toString()方法
  ``` javascript
  var b=false;
  console.log(b.toString());  // "false"
  ```
- Number的toString()方法，包含默认模式和基模式。在默认模式中，无论最初采用什么表示法声明数字，Number 类型的 toString() 方法返回的都是数字的十进制表示。采用 Number 类型的 toString() 方法的基模式，可以用不同的基输出数字，例如二进制的基是 2，八进制的基是 8，十六进制的基是 16。
  ```javascript
  var iNum = 10;
  alert(iNum.toString(2));	//输出 "1010"
  alert(iNum.toString(8));	//输出 "12"
  alert(iNum.toString(16));	//输出 "A"
  ```
### 2. 转换为数字
- ECMAScript 提供了两种把非数字的原始值转换成数字的方法，即 parseInt() 和 parseFloat()。输入参数只能为数字或者string，其余类型都会返回NaN。
- 






