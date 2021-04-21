---
title: 变量的解构赋值
date: 2021-03-09 17:25:30
tags: 
  - ES6
  - 变量解构
categories: ES6
---
## 1. 数组解构赋值
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
```javascript
let [a, b, c] = [1, 2, 3];
```
这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。
如果解构不成功，变量的值就等于undefined。
```javascript
let [foo] = []; // foo 为undefined
let [bar, foo] = [1]; // foo 为undefined
```
也可以进行不完全解构
如果等号右边不是可遍历解构，就会报错。
只要某种数据结构具有Iterator接口，都可以采用数组形式的解构赋值
```javascript
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```
解构允许指定默认值
```javascript
let [foo = true] = [];
foo // true
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```
ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，只有当一个数组成员严格等于undefined，默认值才会生效。

## 2. 对象解构赋值
数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。
如果解构失败，变量的值为undefined
对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。
如果变量名与属性名不一致，必须这么写
```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```
实际上，对象的解构赋值是下面形式的简写
```javascript
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
```
解构也适用于嵌套结果的对象
```javascript
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```
三次解构赋值，最后一次对line属性的解构赋值之中，只有line是变量，loc和start都是模式，不是变量。
对象的解构赋值可以取到继承的属性
```javascript
const obj1 = {};
const obj2 = { foo: 'bar' };
Object.setPrototypeOf(obj1, obj2);

const { foo } = obj1;
foo // "bar"
```

## 3. 对象的解构赋值
可以解构出各个字符，例如
```javascript
const [a, b, c, d, e] = 'hello';
let {length : len} = 'hello';
len // 5
```

## 4. 数值和布尔值的解构赋值
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。
```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```
解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。

## 5. 函数参数的解构赋值
函数的参数也可以使用解构赋值。
```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

## 6. 解构的用途
1. 交换变量的值
2. 从函数处返回多个值
3. 函数参数的定义
4. 提取JSON数据
5. 函数参数的默认值
6. 遍历map结构
7. 输入模块的指定方法