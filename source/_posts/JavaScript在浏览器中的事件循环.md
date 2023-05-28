---
title: JavaScript在浏览器中的事件循环
date: 2023/03/07/ 21:08
tag: JavaScript
category:
 - 前端
---
**由于Javascript是单线程执行，所以有同步和异步的概念，正因为有同步和异步的概念，所以需要单线程逐步执行同步和异步的事件。**

**Javascript执行时，同步按顺序执行，遇到异步函数则会放到异步队列中等待执行，所以Javascript中的同步代码最优先执行。**


![1678190653124.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31a201bb483b4d4ab01395443645b26e~tplv-k3u1fbpfcp-watermark.image?)

**javascript每一次执行函数都是一个入栈和出栈的过程，执行时入栈，执行完出栈。**

## 浏览器中的事件循环
**在浏览器中，JavaScript的任务队列实际上分为宏任务和微任务队列，浏览器优先执行微任务，当微任务执行完时，才会执行宏任务，但每一次执行一个宏任务后，浏览器会查看微任务队列中是否存有未执行的任务，如果有，则会将微任务队列执行完，再去执行宏任务，反复如此，直至任务队列全部执行完成。**

## 宏任务（MacroTask）
**常见的宏任务有Event（监听事件）、setTimeout、setInterval、（ajax callback），在每一次执行完一个宏任务后，浏览器就会检查微任务队列是否有任务。也就是宏任务执行之前，必须保证微任务队列是空的；如果不为空，那么就先执行微任务队列中的任务（回调）。**

## 微任务（MicroTask）
**常见的微任务有Promise.then（）的回调、Mutation Observer API、queueMicrotask（），微任务优先执行。**

## 举个🌰
```js
setTimeout(function (){
    console.log('set1');
    new Promise(function (resolve){
        resolve();
    }).then(function (){
        new Promise(function (resolve){
            resolve();
        }).then(function (){
            console.log('then4');
        });
        console.log('then2');
    });
});
new Promise(function (resolve){
    console.log('pr1');
    resolve();
}).then(function (){
    console.log('then1');
});
setTimeout(function (){
    console.log('set2');
});
console.log(2);
queueMicrotask(()=>{
    console.log('queueMicrotask1')
});
new Promise(function (resolve){
    resolve();
}).then(function (){
    console.log('then3');
});
```
**根据浏览器事件循环的逻辑，以上代码的输出顺序是怎么样的呢？**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89ffc12f09b0499785110a2c4ff03d2e~tplv-k3u1fbpfcp-watermark.image?)

**首先，我们画一个宏任务队列（MacroTaskQueue）和一个微任务队列（MircroTaskQueue），根据上述的任务队列执行顺序可以推断出上述的结果。**
**当我们遇到第一个setTimeout时，将它先放入宏任务队列，遇到Promise函数时，函数体内的函数照常同步执行，所以第一个输出"pr1",因为第一个Primose执行了resolve函数，所以resolve的回调需要执行，所以将第一个Promise.then的回调放入微任务队列，同样遇到第二个setTimeout时，直接放入宏任务队列，遇到console.log（2）时，直接执行，因为这是同步的，所以输出2，遇到queueMicrotask（）函数时，直接放入微任务队列，接下来遇到最后一个Promise函数，同样，将Promise.then的回调函数放入微任务队列中，随后我们看微任务队列，执行微任务队列里边的函数，首先输出"then1"(第一个Promise.then的回调输出)，然后执行queueMicrotask（）函数（输出"queueMicrotask1"），随后输出"then3"（第二个Promise.then的回调输出），这时微任务队列执行完成，开始执行宏任务队列，执行第一个setTimeout函数，输出"set1",同样遇到Promise.then的回调函数时，放入微任务队列，分别先后输出"then2"和"then4"，最后，执行最后一个setTimeout函数，输出"set2",到此，全部输出完毕。**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ff5c6a4247c4d1ead61412814ba12ca~tplv-k3u1fbpfcp-watermark.image?)
**如图，浏览器中的执行结果跟我们输出的结果一致。**

## 结论
1. 浏览器的任务队列分为微任务队列和宏任务队列
2. 微任务队列优先于宏任务队列执行
3. 在每一次执行完一个宏任务后，浏览器就会检查微任务队列是否有任务
4. 宏任务执行之前，必须保证微任务队列是空的；如果不为空，那么就先执行微任务队列中的任务（回调）
5. 常见的微任务有Promise.then（）的回调、Mutation Observer API、queueMicrotask（）
6. 常见的宏任务有Event（监听事件）、setTimeout、setInterval、（ajax callback）
