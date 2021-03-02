---
title: 基础知识
---

## 1、0.1+0.2为什么不等于0.3
JavaScript使用Number类型表示数字（整数和浮点数)，遵循 IEEE 754 标准，通过64位来表示一个数字。
JS中1位符号位，11位的指数e,剩余52位是尾数。
js最大安全数是 Number.MAX_SAFE_INTEGER == Math.pow(2,53) - 1。本来应该是Math.pow(2,52)-1，但是二进制表示的有效数字总是1.xxx。
计算机无法计算10进制，因此会先按照IEEE 754转换成二进制进行运算。
浮点数运算的五个步骤：对阶、尾数运算、规格化、舍入处理、溢出判断
1、0.1和0.2在转化为二进制后，都是无限循环小数，只能保留52位，会有精度损失
2、在运算时，需要对阶，一般是小阶对大阶，这一步也可能丢失进度，
3、对阶后进行尾数运算，相加后得到可结果，如果结果的整数部分超过1，需要进行规格化
4、规格化之后如果小数部分超出52位，需要再次摄入处理
5、0.1+0.2不涉及溢出判断，直接将结果转为10进制

## 2、[1]==[1]为什么为false
引用类型的比较，是数组堆内存的地址的比较，两个数组在堆内存中地址是不同的。

## 3、如何将值类型的变量以引用的方式传递
```angular2
var str = "abcd";        //基本类型
var obj = {"str":str};   //引用类型
var boj2 = obj;          //复制引用地址
console.log(boj2.str);   //abcd
obj.str = "bcd";
console.log(boj2.str);   //bcd
```
## 4、const定义的Array中间元素是否可以被修改？如果可以，那么const修饰对象有什么意义
const定义的Array中间元素可以被修改，const定义的变量保存的是Array的地址，这个变量本身不可以修改，而存在于堆内存的Array本身是可以修改的。
对于const声明，只是它的赋值操作被冻结了，而值不会因为const而不变。主要是预防在coding过程中的coder因为疏忽对变量的意外修改。
## 5、null与undefined的区别
null表示"没有对象"，此处不该有值

典型用法
 - 作为函数的参数，表示该函数的参数不是对象
 - 作为对象原型链的终点

undefined表示"缺少值"，此处该有值，但是还未定义
 - 变量被声明了，但是没有赋值，就是undefined
 - 调用函数时，应该提供的参数没有提供，该参数等于undefined
 - 对象没有赋值的属性，该属性的值为undefined
 - 函数没有返回值时，默认返回undefined 

要比较相等性之前，不能将 null 和 undefined 转换成其他任何值，它们就是相等的，在ECMAScript规范中也是这样定义的。
## 6. ==和===的操作流程
这两个操作符都会先转换操作，再比较它们的相等性
转化基本规则
1. 如果有一个操作数是布尔值，则在比较相等性之前现将其转换为数值-false转换为0，true转换为1
2. 如果有一个操作数是字符串，另一个操作数是数值，在比较相等性之前先将字符串转换为数值
3. 如果一个操作数是对象，另一个操作数不是，则调用对象valueOf( )方法，用得到的基本类型值按照前面的规则进行比较
4. 如果一个操作数是布尔值，另一个操作数是字符串，会把双方转换为数字，再进行比较

比较时遵循规则：
1. null和undefined相等
2. 比较相等性之前，不能把null和undefined转换成任何其他值
3. 如果有一个操作数是NaN，一定返回false，包括NaN==NaN
4. 如果两个操作数都是对象，则比较它们是不是同一个对象