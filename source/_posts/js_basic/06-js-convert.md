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
- parseInt()从第一个字符开始依次查看是否是有效数字，直到遇到无效数字停止，将有效数字部分转换为数字。字符串中包含的数字字面量会被正确转换为数字，比如 "0xA" 会被正确转换为数字 10。parseInt()还有基模式，可以选择任何进制的字符串进行转换。
  ```javascript
  var iNum1 = parseInt("12345red");	//返回 12345
  var iNum1 = parseInt("0xA");	//返回 10
  var iNum1 = parseInt("56.9");	//返回 56
  var iNum1 = parseInt("red");	//返回 NaN
  var iNum1 = parseInt("AF", 16);	//返回 175
  var iNum1 = parseInt("010");	//返回 8
  var iNum2 = parseInt("010", 8);	//返回 8
  var iNum3 = parseInt("010", 10);	//返回 10
  ```
- parseFloat()与praseInt()类似，但是第一个出现的小数点是有效字符。另外，字符串必须以10进制来表示浮点数，不能用8进制或者16进制。
  ``` javascript
  var fNum1 = parseFloat("12345red");	//返回 12345
  var fNum2 = parseFloat("0xA");	//返回 0
  var fNum3 = parseFloat("11.2");	//返回 11.2
  var fNum4 = parseFloat("11.22.33");	//返回 11.22
  var fNum5 = parseFloat("0102");	//返回 102
  ```






