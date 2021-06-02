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
### 3.强制类型转换
JS中有三种强制类型转换
```javascript
Boolean(value) // 当要转换的值是至少有一个字符的字符串、非 0 数字或对象时，Boolean() 函数将返回 true。如果该值是空字符串、数字 0、undefined 或 null，它将返回 false。
Number(value)  // 把给定的值转换成数字（可以是整数或浮点数）；Number() 函数的强制类型转换与 parseInt() 和 parseFloat() 方法的处理方式相似，只是它转换的是整个值，而不是部分值
String(value) // 把给定的值转换成字符串；强制转换成字符串和调用 toString() 方法的唯一不同之处在于，对 null 和 undefined 值强制类型转换可以生成字符串而不引发错误

```

### 4. 隐式类型转换
#### 4.1 隐式转换为布尔：“truthy”和“falsy”
当 JavaScript 需要一个布尔值时（例如：if 语句），任何值都可以被使用。 最终这些值将被转换为 true 或 false
undefined,Boolean值的false,+0,-0,NaN和空字符串会转换为false，其余会被转换为true。
#### 4.2 字符串的隐式转换
加运算符（+），就有这方面的问题，因为只要其中一个操作数是字符串，那么它就执行连接字符串的操作（而不是加法操作，即使它们是数字）
```javascript
> Boolean(false)
false
> String(false)
'false'
> Boolean('false')  // ！！
true
```
#### 4.3 对象的隐式转换
当某个对象出现在了需要原始类型才能进行操作的上下文时，JavaScript 会自动调用 ToPrimitive 函数将对象转化为原始类型。
当需要将对象转换成数字时，需要以下三个步骤：
1. 调用 valueOf()。如果结果是原始值（不是一个对象），则将其转换为一个数字。
2. 否则，调用 toString() 方法。如果结果是原始值，则将其转换为一个数字。
3. 否则，抛出一个类型错误。

如果把对象转换成字符串时，则转换操作的第一步和第二步的顺序会调换： 先尝试 toString() 进行转换，如果不是原始值，则再尝试使用 valueOf()。

```javascript
[]+{}
// "[object Object]"
{} +[]
// 0 
整体步骤：
1. +[]
2. Number([])
3. Number([].toString())  // [].valueOf() isn’t primitive
4. Number("")
5. 0
> {} + {}
'[object Object][object Object]'
> {} + []
'[object Object]'
```
第二种情况的原因是：JavaScript 把第一个 {} 解释成了一个空的代码块（code block）并忽略了它。 你在这里看到的 + 加号并不是二元运算符「加法」，而是一个一元运算符，作用是将它后面的操作数转换成数字，和 Number() 函数完全一样。
#### 4.4 其余特殊情况
```javascript
[1,2]+[3,4]
// '1,23,4'
```
```javascript
++[[]][+[]]+[+[]] = '10'
步骤：
1. +[]===0,所以 ++[[]][0]+[0]
2. [[][0]可以认为取出数组的第一个元素[]，++[]+[0]
3. ++[]替换为+([]+1)，+([]+1)+[0]
4. +([]+1)=>+(''+1)=>+('1')=>1
5. 1+[0]=> 1+'0'=> '10'
```
因为当一个对象同时包含 toString() 和 valueOf() 方法的时候，运算符 + 应该调用哪个方法并不明显(做字符串连接还是加法应该根据其参数类型，但是由于隐式类型转换的存在，类型并不显而易见。)，Javascript 会盲目的选择 valueOf() 方法而不是 toString() 来解决这个问题。
```javascript
var obj = {
    toString: function() { return "Object CustomObj"; },
    valueOf: function() { return 1; }
};

console.log("Object: " + obj);    // "Object: 1"
```
