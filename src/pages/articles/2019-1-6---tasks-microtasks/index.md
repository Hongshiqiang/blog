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

微任务通常是为当前执行脚本结束后应该立即发生的事情，例如对一批动作作出反应，或者在不承担整个新任务的代价下进行异步，只要没有其他javascript在执行中，微任务队列就会在回调之后处理，在每个任务结束的时候。在微任务期间排队的任何其他微任务都被添加到队列的末尾并进行处理。微任务包括`mutation observer callbacks`，上面的例子中的`promise`的`callbacks`。

一个settles状态的promise，或者已经变成settled状态，它会将callback放到微任务队列里。这确保promise的回调是异步的即使promise已经变成settled状态。因此对已settled状态的promise调用`.then()`会立即把一个微任务添加到微任务队列。这就是为什么`script end`结束后打印`promise1`和`promise2`，因为当前运行的脚本必须在处理微任务之前完成。在setTimeout之前打印promise1和promise2，因为微任务总是在下一个任务之前发生。

okay 一步一步运行

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

```

没错上面的解释是对的，我创建一个step-by-step动画图解(`这个原文有个动画可以一步一步点击运行js代码，可以点下方原文地址进行访问操作`)。你星期六过得怎么样？和你的朋友一起出去玩？我没有。(`反正我的休息日除了学习就是看电影`)如果我的UI设计不够清晰，点击上方箭头运行代码。

##有些浏览器有什么不同之处？

有些浏览器打印`script start`，`script end`，`setTimeout`，`promise1`，`promise2`。他们在setTimeout之后运行promise回调。很可能他们调用promise回调是作为新任务的一部分而不是作为微任务。

这多少情有可原，