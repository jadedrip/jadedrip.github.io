---
title: '简单说下 Vue3 里 ref, reactive 的区别'
layout: post
subtitle: ''
tags:
  - Vue	
category: 
---

编写 vue 的网页时，很多人对 `ref` 和 `reactive` 的区别感到困惑，这里简单介绍一下。

ref 和 reactive 都是 Vue3 提供的响应式 API，用于创建响应式数据。它们的主要区别在于使用方式和适用场景。

ref 用于数据本身会被更改的场合，代码中需要通过 .value 来访问和修改数据，所以基本数据最好使用 ref。reactive 用于数据本身不会更改的场合，代码中可以直接访问和修改数据，reactive 最大的好处是代理里可以省略 .value 。

```javascript
import { ref, reactive } from 'vue'

const a = ref({ count: 0 })
const b = reactive({ count: 0 })

console.log(a.value.count) // 需要通过 .value 来访问
console.log(b.count) // 可以直接访问

a.value = { newCount: 1 }  // 可以修改 count 对象本身
b.count = 55  // 只能修改 state 对象内 count 的值，state 对象本身没有被更改

b = { newCount: 1 } // 会破坏对象导致报错，reactive 对象不能被重新赋值

```