---
title: 任务, 微任务, 队列和时间表
date: "2019-1-6"
layout: post
draft: false
path: "/posts/tasks-microtasks/"
category: "Javascript (translation)"
tags:
  - "首页"

description: "当我告诉我的同事Matt Gaunt我想写一篇关于微任务 队列和在浏览器事件循环中执行的文章时；  他说；我实话跟你说Jake我是不会去看的。 好吧；无论如何我已经写下来了；所以我们都要坐下来享受这篇文章； 好吗 ？"
---

当我告诉我的同事Matt Gaunt我想写一篇关于微任务 队列和在浏览器事件循环中执行的文章时，  他说；我实话跟你说Jake我是不会去看的。 好吧，无论如何我已经写下来了，所以我们都要坐下来享受这篇文章， 好吗 ？

如果你更喜欢视频，Philip Roberts在JSConf上关于事件循环有一个很棒的演讲 --不包括微任务， 但是很好的介绍了其他部分。不管怎样，精彩继续...

看看这段JavaScript代码：

```javascript

console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');

```
控制台上会以什么顺序出现呢？

##可以打开控制台试一下

正确答案是：
```javascript

script start
script end
promise1
promise2
setTimeout

```
但就浏览器支持而言，它是相当疯狂的

Microsofe Edge，Firefox 40，IOS Safari和桌面Safari 8.0.8在promise1和promise2之前打印setTimeout - 尽管这看起来像是一个竞争条件，这真的很奇怪，因为在Firefox39和Safari 8.0.7上始终是正确的

##为什么会这样？
