---
title: Node.JS中的事件循环 Event Loop
date: 2023/03/08/ 22:18
tag: Node.js
category:
 - 后端
---


<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/262b606c572641ffbeaaf9014fab4276~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%" />

##  前言
**JavaScript中的事件循环（Event loop）在浏览器中和Node.JS中是有一定的不同，浏览器中的事件循环可以参考这篇[（JavaScript在浏览器中的事件循环）](https://juejin.cn/post/7207743145998860325)，那么在Node.JS中的事件循环具体是什么呢？我总结了以下不同。**

**Javascript执行时，同步按顺序执行，遇到异步函数则会放到异步队列中等待执行，所以Javascript中的同步代码最优先执行。在Node.JS中也是如此，所以同步函数永远优先执行。**

**Node的事件循环更复杂，它也分为微任务和宏任务**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9b481a3600c4e898e587cf13eadf284~tplv-k3u1fbpfcp-watermark.image?)

## 宏任务（MacroTask）
**在Node.js中，常见的宏任务事件循环有，setTimeout、setInterval、IO事件、setImmediate、close事件，但是，Node中的事件循环不止一个宏任务队列，它主要分为四种宏任务队列，并且宏任务队列也有优先级之分，四种队列按优先级排序，如下**

1. timer queque：setTimeout、setInterval；
2. poll queue：IO事件；
3. check queue：setImmediate；
4. close queue：close事件；

## 微任务（MicroTask）
**在Node.js中，常见的微任务事件循环有，Promise的then回调、process.nextTick、queueMicrotask，同样，Node中的事件循环不止一个微任务队列，它主要分为两种微任务队列，同样的，微任务队列也有优先级之分，两种队列按优先级排序，如下**

1.next tick queue：process.nextTick；
2.other queue：Promise的then回调、queueMicrotask；

## 执行顺序
**大局上的执行顺序与浏览器中的执行顺序相同，优先执行微任务，待微任务队列执行完成后，执行宏任务队列，只不过微任务队列与宏任务队列进行了细分以及优先级的细分。**

**同样，当微任务执行完时，才会执行宏任务，但每一次执行一个宏任务后，浏览器会查看微任务队列中是否存有未执行的任务，如果有，则会将微任务队列执行完，再去执行宏任务，反复如此，直至任务队列全部执行完成。**

## 举个🌰
**根据上边的总结，判断以下代码的输出结果**

```js
async function async1 () {
    console. log('async1 end')
    await async2()
    console.log('async1 end')
}

async function async2(){
    console.log('async2')
}
console.log('script start')
        
setTimeout (function () {
    console.log('setTimeout2')
},0)
setTimeout(function (){
    console.log('setTimeout2')
},300)
setImmediate ( () => console. log ('setImmediate'));
process.nextTick(() => console. log('nextTick1'));
async1();
process.nextTick(() => console. log ('nextTick2' ));
new Promise (function (resolve) {
    console.log('promise1')
    resolve();
    console.log('promise2')
}).then (function () {
    console. log ('promise3')
})

console. log ('script end')
```
**在上边的代码中我们遇到了async、await函数，在async函数中，执行await函数的下一行代码，我们可以当成一个Promise.then的回调，将它放入微任务中的other队列，因为setTimeout2有300ms的延迟入列，所以优先于setImmediate输出。**


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/753a3f12ac674045b0d923950e11533f~tplv-k3u1fbpfcp-watermark.image?)

**根据上述的总结，我们可以画出对应的队列一一入列，然后依照优先级依次执行，执行出的结果如图所示**


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/689e7b3051ba47f5b456c4617269f108~tplv-k3u1fbpfcp-watermark.image?)

**可见，我们按照总结推断出的console.log与输出完全一致。**

## 陷阱题目！
**那么，我们接下来看一下这个代码**
```js
setTimeout (function () {
    console.log('setTimeout')
},0)
setImmediate (
    () => console. log ('setImmediate'));
```
**我们先执行一次看一下结果，想必大家已经有了结果。**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/121a265a47e645bb879cfbc263480d65~tplv-k3u1fbpfcp-watermark.image?)
**那我们再次执行，看看会有什么结果呢？**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eaa7c1728644dea9c38fed2e2da29d3~tplv-k3u1fbpfcp-watermark.image?)
**为什么两次执行结果不一样呢？**

**因为当Node.js开始初始化加载事件循环时，如果初始化过快，setTimeout（timer）没有加载到任务队列，就会先执行setImmediate，执行完后，发现有timer微任务，就会再执行timer微任务队列，当初始化的时间小于setTimeout（timer）加载到任务队列的时间时，当Node.js开始执行事件循环时，setTimeout（timer）就已经加载到timer微任务队列了，所以就会根据优先级执行，所以就会产生这两种不同的结果。**

## 总结
1. Node中的事件循环不止一个宏任务队列，它主要分为四种宏任务队列
2. Node中的事件循环不止一个微任务队列，它主要分为两种微任务队列
3. 常见的宏任务事件循环有，setTimeout、setInterval、IO事件、setImmediate、close事件
4. 常见的微任务事件循环有，Promise的then回调、process.nextTick、queueMicrotask