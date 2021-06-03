---
title: typeof vs instanceof
date: 2021-06-03 10:45:00
tags: 
  - == vs ===, typeof vs instanceof
categories: JavaScript
---
### 1. JS中的相等性判断
#### 1.1 严格相等===
1. 严格相等比较两个值是否相等时，不进行强制类型转换
2. 如果两个被比较的值类型相同，值也相同，并且都不是 number 类型时，两个值全等
3. 两个值都是number类型，都不是NaN但是数值相等时，或者+0和-0时，认为相等

#### 1.2 非严格相等==
相等操作符比较两个值是否相等，在比较前将两个被比较的值转换为相同类型。
###### 转化基本规则
1. 如果有一个操作数是布尔值，则在比较相等性之前现将其转换为数值-false转换为0，true转换为1
2. 如果有一个操作数是字符串，另一个操作数是数值，在比较相等性之前先将字符串转换为数值
3. 如果一个操作数是对象，另一个操作数不是，则调用对象valueOf( )方法，用得到的基本类型值按照前面的规则进行比较
4. 如果一个操作数是布尔值，另一个操作数是字符串，会把双方转换为数字，再进行比较

比较时遵循规则：
1. null和undefined相等
2. 比较相等性之前，不能把null和undefined转换成任何其他值
3. 如果有一个操作数是NaN，一定返回false，包括NaN==NaN
4. 如果两个操作数都是对象，则比较它们是不是同一个对象

简洁记忆：
1. null==undefined, null/undefined不进行隐式类型转换。
2. 进行隐式类型或转换时，优先转换成Number型

#### 1.3 同值相等
判定两个值是否是同值
```javascript
Object.is(0, -0);                 // false
Object.is(+0, -0);                // false
Object.is(-0, -0);                // true
Object.is(0n, -0n);               // true
```

### 2. typeof
typeof返回一个字符串表明数据的类型
| 数据类型     | type        |
| ------------ | ----------- |
| Undefined    | 'undefined' |
| Null         | 'object'    |
| 布尔值       | 'Boolean'   |
| 数值         | 'number'    |
| 字符串       | 'string'    |
| Symbol       | 'symbol'    |
| 函数对象     | 'function'  |
| 任何其他对象 | 'object'    |
typeof来判断数据类型其实并不准确。比如数组、正则、日期、对象的typeof返回值都是object，这就会造成一些误差。
所以在typeof判断类型的基础上，我们还需要利用Object.prototype.toString方法来进一步判断数据类型。

### 3. instancof
instanceof运算符可以用来判断某个构造函数的prototype属性是否存在于另外一个要检测对象的原型链上。instanceof 检测左侧的 __proto__ 原型链上，是否存在右侧的 prototype 原型。
```javascript
//假设instanceof运算符左边是L，右边是R
L instanceof R //instanceof运算时，通过判断L的原型链上是否存在R.prototype
L.__proto__.__proto__ ..... === R.prototype ？ //如果存在返回true 否则返回false
```
所有的构造器的 constructor 都指向 Function

Function 的 prototype 指向一个特殊匿名函数，而这个特殊匿名函数的 __proto__ 指向 Object.prototype
```javascript
Function.__proto__.__proto__ === Object.prototype;//true
Object.__proto__ === Function.prototype;//true
```