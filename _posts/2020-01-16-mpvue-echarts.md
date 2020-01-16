---
title: '解决小程序里的 echarts 图表滑动时有时不显示的问题'
layout: post
subtitle: ''
tags:
  - Vue	
category: 
---

微信小程序的坑也是蛮多的，一个问题已经困惑我很久了，我们的微信小程序用了 mpvue 框架，然后通过 mpvue-echarts 引入了 echarts 组件来画图表，然后就发现了一个奇怪的问题，手指在图表上滑动的时候，如果划出边界，就有一定概率导致整个图表不显示。

折腾了半天 echarts 的配置，各种折腾，一直无效果，拖了好久。

最近看了 mpvue-echarts 的源码，也没找到问题，不过看到一个配置 throttleTouch ，随手设置了一下：

```js
<mpvue-echarts v-if="isOnLoad" :echarts="echarts" :onInit="onInit" :canvasId="canvasId" throttleTouch="true" />
```

然后就神奇的发现这个问题解决了！

源码里这个配置的作用是限制 mousemove 的输入频率

```js
  touchMove(e) {
  ...

      if (throttleTouch) {
        const currMoveTime = Date.now();
        if (currMoveTime - lastMoveTime < 240) return;
        this.lastMoveTime = currMoveTime;
      }
  ...
  }
```

怀疑是触摸输入太频繁导致 echarts 或者微信画布内部多线程冲突。

不管怎么说，这问题解决了就好，哎，真浪费了我好多时间啊！