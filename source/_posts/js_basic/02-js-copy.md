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
#### 对象深拷贝
方式1：手动复制
```angular2html
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = { a: obj1.a, b: obj1.b, c: obj1.c };
obj2.b = 100;
console.log(obj1);
// { a: 10, b: 20, c: 30 } // 沒被改到
console.log(obj2);
// { a: 10, b: 100, c: 30 }
```
方式2：JSON做字符串转换
```angular2html
var obj1 = { body: { a: 10 } };
var obj2 = JSON.parse(JSON.stringify(obj1));
obj2.body.a = 20;
console.log(obj1);
// { body: { a: 10 } }
console.log(obj2);
// { body: { a: 20 } }
console.log(obj1 === obj2);
// false
console.log(obj1.body === obj2.body);
```
备注：这种方式简单易用，但是会抛弃对象的constructor，无论之前的构造函数是什么，拷贝后都是Object
这种方法能正确处理的对象只有Number, String, Boolean, Array, 扁平对象，即那些能够被json直接表示的数据结构。RegExp对象是无法通过这种方式深拷贝。
也就是说，只有可以转成JSON格式的对象才可以这样用，像function没办法转成JSON。
方式3：递归拷贝
```angular2html
function deepClone(initalObj, finalObj) {    
  var obj = finalObj || {};    
  for (var i in initalObj) {        
    var prop = initalObj[i];        // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if(prop === obj) {            
      continue;
    }        
    if (typeof prop === 'object') {
      obj[i] = (prop.constructor === Array) ? [] : {};            
      arguments.callee(prop, obj[i]);
    } else {
      obj[i] = prop;
    }
  }    
  return obj;
}
var str = {};
var obj = { a: {a: "hello", b: 21} };
deepClone(obj, str);
console.log(str.a);
```