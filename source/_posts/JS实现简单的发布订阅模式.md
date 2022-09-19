---
title: JS实现发布订阅模式
date: 2022-09-17 21:08:46
tags: JS 
categories: 设计模式
---
## 发布订阅模式的基本概念

发布订阅模式（Publish/Subscribe）它定义了一种一对多的关系，让多个订阅者同时监听某一个发布者，这个发布者发生变化时就会通知所有的订阅者对象，使得它们能够触发 对应的回调。

发布订阅模式中有三个角色，发布者 Publisher ，事件调度中心 Event Channel ，订阅者 Subscribe。


![发布订阅模式](./images/PublishSubscribe.png)

发布订阅模式是一种我们生活中非常常见的模式列如微信订阅公众号，公众号就是发布者，而订阅者就是微信的部分用户

发布订阅模式在前端技术架构上也是一个非常常见的模式，在 vue 的源码中，为了让数据劫持和视图驱动解耦就是通过架设一层消息管理层实现的，而这一层消息管理层实现的原理就是发布订阅模式。再比如 Redux、Vuex 这些当下比较流行的库基本上都离不开发布订阅模式。


## 话不多说直接上代码
核心代码
```ts
class Pubsub {
  events: any
  constructor() {

    // 事件中心
    // 每种事件(任务)下存放其订阅者的回调函数
    this.events = {}
  }
  // 订阅方法
  subscribe(type: string, cb: Function) {
    if (!this.events[type]) {
      this.events[type] = [];
    }
    this.events[type].push(cb);
  }
  // 发布方法
  publish(type: string, arg: any) {
    if (this.events[type]) {
      this.events[type].forEach((cb: Function) => cb(arg))
    }
  }
}

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