---
title: JS实现发布订阅模式
date: 2022-09-17 21:08:46
tags: promise 
categories: ES6
---
## 手写Promise的常用api

实现一个api最重要的是要详细了解这个api的功能,所以在实现之前我们需要简单的回顾一下
### Promise.resolve
1. 返回值始终是一个Promise对象
2. 如果这个值是 thenable（即带有 "then" 方法），返回的 promise 会“跟随”这个 thenable 的对象

```ts
//非Promise对象，非thenable对象
let promise = Promise.resolve({a:1})
promise.then(console.log)//{a: 1}

//Promise对象成功状态
promise = Promise.resolve(Promise.resolve(2)) // 2
promise.then(console.log)//{a: 1}

////Promise对象失败状态
promise = Promise.resolve(Promise.reject(2)); // 2
console.log(promise) //状态rejected
promise.catch(console.error) //2
```
由此可以发布订阅模式其实就是触发相关的回调其实是一个很容易理解的设计模式
我门在vue，react中使用这个方式来实现组件emit事件也是可以行，不过为了维护等方面的问题，不建议在实际项目中使用

父组件
```ts
<template>
  <div>测试发布订阅实现简单的emit</div>
  <div>父组件 :{{ value }}</div>
  <child />
</template>

<script setup lang="ts">
import child from "./child.vue";
import { ref } from "vue";
import { pubsub } from "./index";
pubsub.subscribe("emit", (e: any) => {
  value.value = e;
});
const value = ref(1);
</script>
```

子组件
```ts
<template>
  <div>子组件:修改子组件值返回到父组件</div>
  <InputNumber
    id="inputNumber"
    v-model:value="value"
    :min="1"
    :max="10"
    @change="change"
  />
</template>

<script setup lang="ts">
import { InputNumber } from "ant-design-vue";
import { ref } from "vue";
import { pubsub } from "./index";
const value = ref(1);
const change = (e: any) => {
  pubsub.publish("emit", e);
};
</script>
```

其实看到这儿就很明显，流程其实和的emit事件方式几乎一模一样，父组件注册相关回调，子组件触发事件回调，ok今天就到这儿结束