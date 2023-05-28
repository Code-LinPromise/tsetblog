---
title: CommonJS模块化引入的实质与使用
date: 2023/03/01/ 20:43
tag: Node.js
category:
 - 后端
---

## 前言
**在Javascript的ES6没有推出时，在社区中推出了很多模块化的规范，其中有AMD、CMD、CommonJS，如今AMD、CMD规范的使用已经很少了，CommonJS与ES6模块化的使用非常常见，那么CommonJS与ES6模块化的引用实质具体是什么呢？**

## CommonJS的使用与实质
**Node.js中使用的就是CommonJS的模块化规范，下面，我将以Node.js的代码，演示如何使用CommonJS**

### CommonJS如何导出
**CommonJS的导出方法主要有两种，一种为exports={}，另一种为module.exports={}**

```js
const name="1in"

//可以直接对exports对象进行赋值
exports.name=name
```
**以上使用exports对象直接导出，下面我们来看看用module.exports对象导出的方法**
```js
const name="1in"

module.exports={
    name:name
}
//也可以使用ES6中的对象简写方法
module.exports={
    name
}
//也可以直接对module.exports对象进行赋值
module.exports.name=name
```
**根据上边的代码我们可以发现，为什么exports和module.exports的导出方法一模一样呢，那为什么还需要设置两种的导出方法。**

**实际上，Node.js在底层实现中，有 module.exports=exports 这行代码，Node.js在模块间实际导出的是module.exports，而exports的对象引用是赋值给module.exports的**

**所以，当我们用到module.exports导出时，不可以在用exports导出了，因为一旦使用module.exports={name:name} exports就不会将对象地址赋值给module.exports了。又因为module.exports才是真正导出的方法，所以，module.exports指向哪个地址，哪个地址的对象就会被导出**

**那么为什么会有一个exports导出方法呢，实际上，Node.js是为了符合CommonJS的规范，设定了一个exports的导出方法。**

### CommonJS的导入
```js

const { name }=require("./demo.js")

```

**当我们exports什么没有都没有导出时，exports就是一个空对象**

```js
exports={
    
}

console.log(exports)
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7af6aa3688e540c1900f7304859e3752~tplv-k3u1fbpfcp-watermark.image?)

**所以每一个模块中都会有一个exports全局对象，默认为空对象，exports指向一个对象，就意味，Node.js在内存里开辟一个新的空间，这个空间就是exports和module.exports指向的空间，当给exports和module.exports对象赋值是，就相当于给这个开辟的空间赋值，所以，另一个的模块能够获取引入别的模块导出的值，就是因为require也指向这个空间，并从里边取值。**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9909822cfcac45d3965a2c454e0658dc~tplv-k3u1fbpfcp-watermark.image?)


**所以，当一个模块导出用exports或者module.exports导出后，设置一个定时器，更改name的值，会导致require接收到的值改变吗？**

```js
//demo1.js

let name='1in'

exports.name=name

setTimeout(()=>{
    name='1lin-----'
},1000)
console.log(name)
```

```
//demo2.js

const {name} =require('./demo')
setTimeout(()=>{
    console.log("两秒后重新读取")
    console.log(name)
},2000)

console.log(name)
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2fc16c1aac94b35b0ba547aaa1f3cc1~tplv-k3u1fbpfcp-watermark.image?)
**那么我们将name设置成一个对象导出呢?**

```js
//demo1.js

let name= {
    name:'1in'
}

setTimeout(()=>{
    name.name='1lin-----'
},1000)

exports.name=name

console.log(name)
```

```js
//demo2.js

let {name} =require("./demo")

console.log(name.name)

setTimeout(()=>{
    console.log("两秒后重新读取")
    console.log(name.name)
},2000)
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cce9e6f06d60486cb95969dac14e6a0c~tplv-k3u1fbpfcp-watermark.image?)

**当我们导出的是一个对象时，导出后，可以利用地址找到这个对象并修改这个对象的属性，就导致了第二种结果，当导出的数据为基本数据类型时，我们没有对应的指针，就无法修改这个导出的值。**