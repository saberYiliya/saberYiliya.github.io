---
title: JS实现发布订阅模式
date: 2022-09-17 21:08:46
tags: promise
categories: ES6
---

## 手写 Promise 的常用 api

实现一个 api 最重要的是要详细了解这个 api 的功能,所以在实现之前我们需要简单的回顾一下

### Promise.resolve

1. 返回值始终是一个 Promise 对象
2. 如果这个值是 thenable（即带有 "then" 方法），返回的 promise 会“跟随”这个 thenable 的对象

```ts
//非Promise对象，非thenable对象
const a = Promise.resolve(1);
console.log(a); //状态fulfilled

//非Promise对象，thenable对象
const b = Promise.resolve({ then: () => {} });
console.log(b); //状态pending,因为then方法没有改变状态()

//Promise对象成功状态
const c = Promise.resolve(Promise.resolve(1));
console.log(c); //状态fulfilled

//Promise对象失败状态
const e = Promise.resolve(Promise.reject(1));
console.log(e); //状态rejected
```

其实根据 resolve 的结果我们很明显观察出 Promise.resolve 的入参如果不是 promise 直接包了一层，如果是直接返回该 promise,了解了 Promise.resolve 的作用那么实现也是很简单的了

```ts
//参考vue源码。判断是否为promise
function isDef(v) {
  return v !== undefined && v !== null;
}

function isPromise(val) {
  return (
    isDef(val) &&
    typeof val.then === "function" &&
    typeof val.catch === "function"
  );
}

Promise.myResolve = (e) => {
  if (isPromise(e)) return e;
  return new Promise((resolve) => {
    resolve(value);
  });
};
```

测试

```ts
const q = Promise.myResolve(1);
console.log(q);
const w = Promise.myResolve({ then: () => {} });
console.log(w);
const r = Promise.myResolve(Promise.resolve(1));
console.log(r);
const t = Promise.myResolve(Promise.reject(1));
console.log(t);
```

看到这里可能还有个疑惑就是 thenable 的处理。(这个应该都是 promise 处理的。因为 myResolve 的实现也基于 promise。。)

### Promise.reject

这个相对于 resolve 更简单，返回一个状态 rejected 的 Preomise 即可

```ts
Promise.myReject = (e) => {
  return new Promise((_, reject) => {
    reject(e);
  });
};
```

### Promise.all

1. 方法接收一个 promise 的 iterable 类型（注：Array，Map，Set 都属于 ES6 的 iterable 类型）的输入,并且只返回一个 Promise 实例
2. 输入的所有 promise 的 resolve 回调的结果是一个数组
3. 只要任何一个输入的 promise 的 reject 回调执行或者输入不合法的 promise 就会立即抛出错误，并且 reject 的是第一个抛出的错误信息。

```ts
//将上面三个promise传入
const all1 = Promise.all([a, b, c]);
console.log(all1); //pending
all1.then((e) => console.log(e)); // 未执行 因为b的promise then方法未修改状态
//修改b为
const b = Promise.resolve({ then: (resolve) => resolve(1) });
//再执行  [1, 1, 1]

const all2 = Promise.all([a, e, c]);
all2.then((e) => console.log(e)).catch((e) => console.log(e));
//可以看见执行到catch 
```
