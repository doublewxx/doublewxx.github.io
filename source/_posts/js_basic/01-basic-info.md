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
```javascript
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

## 7. JavaScript中不同类型以及不同环境下变量的内存都是何时释放
引用类型是在没有引用之后，通过V8的GC自动回收。
值类型如果处于闭包的情况下，要等闭包没有引用才会被GC回收，非闭包的情况下，等待V8的新生代(new space)切换的时候回收。
如何爆掉JS的内存
数组循环赋值
```javascript
let arr = [];
while(true) {
    arr.push(1);
}
```
随着死循环会查出数组的最大长度，报错。
FATAL ERROR: invalid array length Allocation failed - JavaScript heap out of memory

数组循环赋空值
```javascript
let arr = [];
while(true){
    arr.push();
}
```
死循环，不会爆掉V8内存，但是导致进程阻塞。

push Buffer类型
```javascript
let arr = [];
while(true){
    arr.push(new Buffer(1000));
}
```
内存泄露的速度比直接push Number类型的慢很多。
因为ES定义的Number类型，实质上是遵循IEEE-2008的64位存储，也就是说，Number类型的1与buffer类型的1，完全不一样，前者在编译器中是00000000000000000000000000（63个0）1，占了64位；但是后者就只是1，只占了1位。当push buffer崩溃时，也会崩溃，只不过比Number类型的慢一些。
当push buffer崩溃时...不存在的，push buffer不会崩溃，当内存顶到爆，也就是即将到达100%时，会自动垃圾回收，直观地讲，就是会瞬间降低内存，push工作继续，很是神奇。

## 8. JS中typeof的类型有哪些
```javascript
    console.log(typeof undefined);  //undefined
    console.log(typeof 123);  //number
    console.log(typeof '123');  //string
    console.log(typeof true); //boolean
    console.log(typeof [1,2,3]);  //object
    console.log(typeof {"id": 11}); //object
    console.log(typeof null); //object
    console.log(typeof console.log);  //function
```
## 9. koa 的中间件机制
1. app.use函数，在application中将调用use的参数都添加到this.middleware数组中
2. 在push时，对generator语法进行兼容，调用convert函数将其转换为async/await语法
3. 源码中，通过 compose() 这个方法，就能将我们传入的中间件数组转换并级联执行，最后 callback() 返回this.handleRequest()的执行结果。
compose的函数源码
```javascript
function compose (middleware) {
    if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
    for (const fn of middleware) {
        if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
    }

    /**
     * @param {Object} context
     * @return {Promise}
     * @api public
     */

    return function (context, next) {
        // last called middleware #
        let index = -1
        return dispatch(0)
        function dispatch (i) {
            if (i <= index) return Promise.reject(new Error('next() called multiple times'))
            index = i
            let fn = middleware[i]
            if (i === middleware.length) fn = next
            if (!fn) return Promise.resolve()
            try {
                return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
            } catch (err) {
                return Promise.reject(err)
            }
        }
    }
}
```
1. compose()返回一个匿名函数的结果，匿名函数自执行了dispatch() 这个函数，并传入了0作为参数。
2. 上面的代码执行了中间件 fn(context, next)，并传递了 context 和 next 函数两个参数。context 就是 koa 中的上下文对象 context。至于 next 函数则是返回一个 dispatch(i+1) 的执行结果。值得一提的是 i+1 这个参数，传递这个参数就相当于执行了下一个中间件，从而形成递归调用。
3. 只有执行了 next 函数，才能正确得执行下一个中间件。
   所以当我们用 async 的语法写中间件的时候，执行流程大致如下：

    1. 先执行第一个中间件（因为compose 会默认执行 dispatch(0)），该中间件返回 Promise，然后被 Koa 监听，执行对应的逻辑（成功或失败）
    2. 在执行第一个中间件的逻辑时，遇到 await next()时，会继续执行 dispatch(i+1)，也就是执行 dispatch(1)，会手动触发执行第二个中间件。这时候，第一个中间件 await next() 后面的代码就会被 pending，等待 await next() 返回 Promise，才会继续执行第一个中间件 await next() 后面的代码。
    3. 同样的在执行第二个中间件的时候，遇到 await next() 的时候，会手动执行第三个中间件，await next() 后面的代码依然被 pending，等待 await 下一个中间件的 Promise.resolve。只有在接收到第三个中间件的 resolve 后才会执行后面的代码，然后第二个中间会返回 Promise，被第一个中间件的 await 捕获，这时候才会执行第一个中间件的后续代码，然后再返回 Promise
    4. 以此类推，如果有多个中间件的时候，会依照上面的逻辑不断执行，先执行第一个中间件，在 await next() 出 pending，继续执行第二个中间件，继续在 await next() 出 pending，继续执行第三个中间，直到最后一个中间件执行完，然后返回 Promise，然后倒数第二个中间件才执行后续的代码并返回Promise，然后是倒数第三个中间件，接着一直以这种方式执行直到第一个中间件执行完，并返回 Promise，从而实现文章开头那张图的执行顺序。
