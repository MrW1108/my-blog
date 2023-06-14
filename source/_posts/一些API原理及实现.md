---
title: JS API原理及实现
top: true
cover: true
date: 2023-05-18
summary: 一些JS API原理及实现
categories: 前端
tags:
  - JavaScript
  - API
---

## &#x20;API

1. [new](#new)
2. [Promise.all](#Promis.all)
3. [Function.prototype.apply()](<#Function.prototype.apply()>)
4. [深拷贝](#深拷贝)

### new

- 以构造器的 `prototype` 属性为原型，创建新对象
- _将 this 和调用参数传给构造器执行_
- _如果构造器没有手动返回对象，则返回第一步的对象_

```javascript
function new() {
    // 获取函数调用时的第一个参数，即为构造函数 fn
    const fn = Array.prototype.shift.call(arguments)
    if(typeof constructor !== 'function'){
        throw new TypeError(fn + ' is not a constructor')
    }
    // 以构造器的原型为原型，创建新对象
    const obj = Object.create(fn.prototype)
    fn.apply(obj, arguments)
    return obj
}
```

### <span id='Promis.all'>Promis.all</span>

```javascript
function myPromisAll(iterable) {
  let result = [];
  let count = 0;
  let fulfilledCount = 0;
  return new Promise((resolve, reject) => {
    // 用for...of， 因为参数是 iterable 类型（Array, Set, Map )
    for (const prom of iterable) {
      const i = count;
      count++;
      Promise.resolve(prom).then(
        (data) => {
          result[i] = data;
          fulfilledCount++;
          if (fulfilledCount === count) {
            resolve(result);
          }
        },
        (reason) => {
          reject(reason);
        }
      );
    }
    if (count === 0) {
      resolve(result);
    }
  });
}
```

### <span id='Function.prototype.apply()'>Function.prototype.apply()</span>

```javascript
Function.prototype.myApply = function (targetObject, args = []) {
  // 判断调用对象是否为函数
  if (typeof this !== "function") {
    console.error("type error!");
  } // 判断绑定的对象
  if (targetObject === undefined || targetObject === null) {
    targetObject = window;
  } // 因为 apply(2)也有值，但是数字不能添加属性，所以用 Object包装
  targetObject = Object(targetObject); // 确保 targetFnKey唯一，防止覆盖掉原有的属性
  const targetFnkey = Symbol();
  targetObject.targetFnkey = this; // 将执行结果保存并返回
  let result = targetObject.targetFnkey(...args); // 删除暂时增加的属性
  delete targetObject.targetFnkey;
  return result;
};
```

### 深拷贝

**这里有三点需要注意：**

1.  用 `new obj.constructor ()` 构造函数新建一个空的对象，而不是使用{}或者\[],这样可以保持原形链的继承；
2.  用 `obj.hasOwnProperty(key)` ：判定自身是否含有这个属性，会忽略掉那些从原型链上继承到的属性，因为 for..in..也会遍历其原型链上的可枚举属性。
3.  上面的函数用到递归算法，在函数有名字，而且名字以后也不会变的情况下，这样定义没有问题。但问题是这个函数的执行与函数名 factorial 紧紧耦合在了一起。为了消除这种紧密耦合的现象，需要使用 `arguments.callee`。

```javascript
//方法一
function deepCopy(obj) {
  var newobj = obj.constructor === Array ? [] : {};
  if (typeof obj !== "object") {
    return obj;
  } else {
    for (var i in obj) {
      if (typeof obj[i] === "object") {
        //判断对象的这条属性是否为对象
        newobj[i] = deepCopy(obj[i]); //若是对象进行嵌套调用
      } else {
        newobj[i] = obj[i];
      }
    }
  }
  return newobj; //返回深度克隆后的对象
}

//方法二
var clone = function (obj) {
  if (obj === null) return null;
  if (typeof obj !== "object") return obj;
  if (obj.constructor === Date) return new Date(obj);
  if (obj.constructor === RegExp) return new RegExp(obj);
  var newObj = new obj.constructor(); //保持继承链
  for (var key in obj) {
    if (obj.hasOwnProperty(key)) {
      //不遍历其原型链上的属性
      var val = obj[key];
      // 使用arguments.callee解除与函数名的耦合
      newObj[key] = typeof val === "object" ? arguments.callee(val) : val;
    }
  }
  return newObj;
};
```
