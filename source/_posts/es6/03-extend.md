---
title: 各类扩展
date: 2021-04-15 16:23:30
tags: 
  - ES6
  - 各类扩展
categories: ES6
---
## 1. 字符串的扩展
1. 字符的Unicode表示法： 加强了对 Unicode 的支持，允许采用\uxxxx形式表示一个字符，其中xxxx表示字符的 Unicode 码点。
2. 字符串的遍历器接口
3. ES2019 允许 JavaScript 字符串直接输入 U+2028（行分隔符）和 U+2029（段分隔符）
4. 根据标准，JSON 数据必须是 UTF-8 编码。但是，现在的JSON.stringify()方法有可能返回不符合 UTF-8 标准的字符串。如果遇到0xD800到0xDFFF之间的单个码点，或者不存在的配对形式，它会返回转义字符串，留给应用自己决定下一步的处理。
5. 模板字符串
6. 模板编译
## 2. 字符串新增方法
```javascript
String.fromCodePoint() // 用于从 Unicode 码点返回对应字符，可以识别码点大于OXFFFF的字符
String.raw() // ES6 还为原生的 String 对象，提供了一个raw()方法。该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法
// 实例方法：
a.codePointAt() // 正确处理 4 个字节储存的字符，返回一个字符的码点。
a.codePointAt() // 方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}
includes()
startsWith()
endsWith()
repeat()
'x'.repeat(3) // "xxx"
padStart()
padEnd()
'x'.padStart(5, 'ab') // 'ababx'
'x'.padEnd(5, 'ab') // 'xabab'
trimStart() // 过滤前面空格
trimEnd() // 过滤后面空格
matchAll() // matchAll()方法返回一个正则表达式在当前字符串的所有匹配
replaceAll() // 全量替换
```


