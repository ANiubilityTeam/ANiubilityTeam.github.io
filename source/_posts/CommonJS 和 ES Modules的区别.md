---
title: CommonJ 和 ES Modules的区别
author: showuu
tags: 
  - ES6
  - CommonJS
  - javascript
categories: 
  - ES6
description: 简单介绍CommonJS和ES Modules的区别，梳理前端模块化的知识点
date: 2021-07-10
---

# 主要区别

1. CommonJS输出值的拷贝，ES Modules输出值的引用
2. CommonJS运行时加载，ES Modules编译时输出接口
3. CommonJS的`require()`是同步加载模块，ES Modules的`import`命令是异步加载，有一个独立的模块依赖的解析阶段

## CommonJS输出拷贝 与 ES Modules输出引用

### CommonJS

CommonJS模块输出值以后，模块内部的**重新赋值**不会影响到对外输出的值

```javascript
// content.js
let num = 7;
let obj = { name: 'showuu' };
setTimeout(() => {
  num = 10;
  obj = { name: 'wfz' }
});
module.exports = { num, obj };

// index.js
const { num, obj } = require('./content.js');
console.log(num); // 7
console.log(obj); // { name: 'showuu' }
setTimeOut(() => {
  console.log(num); // 7
  console.log(obj); // { name: 'showuu' }
});
```

但注意**引用类型数据（对象、数据、函数等）**属性的变化会反映到外部输出值上，说明CommonJS的输出拷贝是浅层拷贝，详情请了解[浅拷贝与深拷贝]:To Write

```javascript
// content.js
let obj = { name: 'showuu' };
setTimeout(() => {
  obj.name = 'wfz'
});
module.exports = { obj };

// index.js
const { obj } = require('./content.js');
console.log(obj); // { name: 'showuu' }
setTimeOut(() => {
  console.log(obj); // { name: 'wfz' }
});
```

### ES Modules

ES Modules在编译阶段对外输出的是模块内部值的只读引用，当代码真正执行后，会根据引用去内部模块取对应的值，因此模块内部引用的变化，会反映到外部，属于动态引用

```javascript
// content.js
let num = 7;
let obj = { name: 'showuu' };
setTimeout(() => {
  num = 10;
  obj = { name: 'wfz' }
});
export { num, obj };

// index.js
import { num, obj } from './content.js';
console.log(num); // 7
console.log(obj); // { name: 'showuu' }
setTimeOut(() => {
  console.log(num); // 10
  console.log(obj); // { name: 'wfz' }
});
```

## CommonJS可读可写 与 ES Modules的read-only特性

### CommonJS

`require()`引入的变量是可读写的，允许被重新赋值

```javascript
// content.js
const num = 7;
const obj = { name: 'showuu' };
modulex.exports = { num, obj };

// index.js
const content = require('./content.js')
content.num = 10
content.obj = { name: 'wfz' }
console.log(num) // 10
console.log(obj) // { name: 'wfz' }
```

### ES Modules

`import`属性是只读的，不允许赋值

```javascript
// content.js
export const num = 7;
export const  obj = { name: 'showuu' };

// index.js
import * as content from './content.js'
content.num = 10 // Cannot assign to read only property 'num' of object '[object Module]'
content.obj = 10 // Cannot assign to read only property 'obj' of object '[object Module]'
```

但引用类型数据（对象、数组、函数等）的属性值是可以重新赋值的，因为没有改变引用类型数据指针指向的内存地址，也就没有违反`import`的只读特性

```javascript
import * as content from './content.js'
content.obj.name = 'wfz' // obj: { name: 'wfz' }
```

## 参考资料

[ECMAScript6入门 Module的加载实现 阮一峰](https://es6.ruanyifeng.com/#docs/module-loader)