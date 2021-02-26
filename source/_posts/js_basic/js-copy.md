---
title: JS数组的深拷贝与浅拷贝
---
### js数据类型
js数据类型包含基本数据类型(String,Number,Boolean,Null,Undefined)和引用数据类型(Array,Object,Function)
### 堆内存和栈内存
1、栈内存中存放的是存储对象的地址，而堆内存中存放的是存储对象的具体内容。
2、基本数据类型的地址和具体值都存在于栈内存中，引用数据类型的地址存在于栈内存中，而具体内容存在在堆内存中。
3、栈内存运行效率比堆内存高，空间相对堆内存来说较小。所以将构造简单的原始类型值放在栈内存中，将构造复杂的引用类型值放在堆中而不影响栈的效率。
### 浅拷贝与深拷贝
浅拷贝：复制一份引用，所有引用对象都指向一份数据，且都可以修改这份数据。
深拷贝：复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。
### 浅拷贝
#### 数组的浅拷贝
```angular2html
let arr1 = [1,2,3];
let arr2 = arr1;
arr1[0] = 2;
console.log(arr1[0]);  //2
```
#### 对象的浅拷贝
方式1：
```
let srcObj = {'name': 'lilei', 'age': '20'};
let copyObj = srcObj;
copyObj.age = '22';
console.log('srcObj', srcObj);   // srcObj { name: 'lilei', age: '22' }
console.log('copyObj', copyObj);  // copyObj { name: 'lilei', age: '22' }
```
方式2：Object.assign
可以处理对象属性为简单类型的深拷贝
```angular2html
let srcObj = {'name': 'lilei', 'age': '20'};
let copyObj2 = Object.assign({}, srcObj, {'age': '21'});
copyObj2.age = '23';
console.log('srcObj', srcObj); //{ name: 'lilei', age: '20' }
```
不能处理对象属性为引用类型，则那对于这个对象而言其实是浅拷贝的
```
var obj = { a: {a: "hello", b: 21} };
var initalObj = Object.assign({}, obj);
initalObj.a.a = "changed";
console.log(obj.a.a); // "changed"
```
### 深拷贝
#### 数组深拷贝
方式1：直接遍历赋值
```angular2html
var arr1 = [1,3,5];
var arr2 = [];
arr1.forEach(function(value,index){
arr2[index] = value;
})
console.log(arr2);
//这个时候改变arr1[0]  = 3;那么输出arr2[0]还是等于1
```
方式2：使用slice方法
```angular2html
var arr1 = [1,2,3];
var arr2 = arr1.slice(0);
console.log(arr2); //[1,2,3]
```
方式3：使用concat方法
```angular2html
var arr1 = [1,2,3];
var arr2 = arr1.concat();
console.log(arr2); //[1,2,3]
```
方式4：使用map方法
```
var arr1 = [2,3,4];
var arr2 = arr1.map(function(value){
return value;  
})
console.log(arr2);  //[2,3,4]
```
方式5：ES6扩展运算符
```angular2html
  var arr = [1,2,3,4,5];
  var [ ... arr2 ] = arr;
  console.log(arr); //[1,2,3,4,5]
  console.log(arr2); //[1,2,3,4,5]
```
方式6：for-in连原型链也一并复制的方法
```angular2html
var arr = [1,2,3,4,5];
arr.prototype = 5;
var arr2 = [];
for(var a in arr) {
  arr2[a] = arr[a];
}
console.log(arr2); // [1,2,3,4,5]
console.log(arr2.prototype); // 5
//之前的方法中新数组的prototype都是undefined
```
多维数组的深拷贝
```angular2html
var arr = [[1,2],3,4,[5,6]];
var arr3 = JSON.parse(JSON.stringify(arr));  
console.log(arr3) // [[1,2],3,4,[5,6]]
```