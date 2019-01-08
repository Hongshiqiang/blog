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

理解这一点你要知道事件循环是如何处理任务和微任务，当你第一次遇到它的时候，这可能会让你头脑清醒很多。深呼吸...

每个线程都有自己的事件循环，所以每个工作线程都有它自己的事件循环，它可以独立执行，而同一起点上的所有窗口共享一个事件循环，因为它们可以同步通讯，事件循环持续运行，执行任何队列里的任务，一个事件循环有多个任务源，这些任务源保证了该源内的执行顺序(比如IndexedDB这样的规范定义它们自己的任务源)，但是浏览器可以在每次循环中选择从那个源执行任务。这允许浏览器优先选择性能敏感的任务，比如用户输入。

任务被放到任务源，这样浏览器就可以从内部访问Javascript/Dom，并确保这些操作按顺序进行。任务之间，浏览器可能会重新渲染，从鼠标点击事件到事件回调需要调度一个任务，解析HTML也是一样的，在上面的例子中setTimeout也一样。

setTimeout等待给定的延迟，然后为其回调安排一个新的任务，这就是为什么setTimeout在script end之后打印，scripe end在第一个任务内，setTimeout被记录在单独的一个任务中，好了，我们快讲完了，但是我需要你为剩下这一点坚持下...