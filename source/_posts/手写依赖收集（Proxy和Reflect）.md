---
title: 手写依赖收集（Proxy和Reflect）
date: 2023/03/27 22-18
tag: JavaScript
sticky: 100
category:
 - 前端
---


## 响应式是什么？🙄
在手写依赖收集之前，我们需要明白响应式到底是什么，假设，我们有一个对象info，info对象内有name、age两个属性元素，代码如下
```js
const info={
    name:"1in",
    age:20
}
```
此时我们有一个watchFn函数，依赖于info.name执行，代码如下
```js
console.log(info.name)
```
每当我们info中的name改变时，watchFn函数因为依赖了info.name,所以watchFn函数就自动执行，这就是响应式，那么我们怎么监听info.name的变动呢？这时我们需要用到Proxy，在Vue2中使用的是Object.defineProperty，而在Vue3中则使用的就是Proxy。

## Proxy的基本使用
Proxy的汉译就是代理，具体基本用法如下
```js
//声明一个对象info
const info={
    name:"1in",
    age:20
}

//将对象info与Proxy绑定关系
const infoProxy=new Proxy(info,{
    get(target, key, receiver) {
      
       return Reflect.get(target,key,receiver)
},
    set(target, key, newValue, receiver) {
    
        Reflect.set(target,key,newValue,receiver)
}
```
此时，我们就可以在get，set方法中具体监听info对象，[Proxy具体用法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，[Reflect具体用法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)。

## 依赖收集（响应式原理）
手写源码如下
```js
//保存当前响应式函数
let activeReactiveFn=null
class Depend{
    constructor() {
        this.reactiveFns=new Set()
    }
    addDepend(reactiveFn){
        this.reactiveFns.add(reactiveFn)
    }
    depend(){
        if(activeReactiveFn){
            this.reactiveFns.add(activeReactiveFn)
        }
    }
    notify(){
        this.reactiveFns.forEach(fn=>{
            fn()
        })
    }
}

//封装一个响应式函数
function watchFn(fn){
    activeReactiveFn=fn
    fn()
    activeReactiveFn=null
}

//封装一个获取depend函数
const targetMap=new WeakMap()
function getDepend(target,key){
    //根据target获取map
    let map=targetMap.get(target)
    if(!map){
        map = new Map()
        targetMap.set(target,map)
    }
    //根据key获取depend
    let depend=map.get(key)
    if(!depend){
        depend = new Depend()
        map.set(key,depend)
    }
    return depend
}

//Vue3响应式原理Proxy
function reactive(obj){
    return new Proxy(obj,{
        get(target, key, receiver) {
            //依赖收集
            //根据target，key获取对应的depend
            const depend=getDepend(target,key)
            //给depend添加函数
            // depend.addDepend(activeReactiveFn)
            depend.depend()

            return Reflect.get(target,key,receiver)
        },
        set(target, key, newValue, receiver) {
            Reflect.set(target,key,newValue,receiver)
            // depend.notify()
            const depend=getDepend(target,key)
            depend.notify()
        }
    })
}

const info={
    name:"1in",
    age:20
}
const infoProxy=reactive(info)
watchFn(()=>{
    console.log(infoProxy.name)
})
infoProxy.name="1in."
```
首先，我们需要声明一个Set数据结构reactiveFns来存储watchFn函数（可以用数组来存储，但是数组的值可以重复，可能会导致收集的依赖函数重复，所以这里我们用Set数据结构，防止重复。），因为watchFn函数不止一个，并且我们需要依次执行所有的watchFn函数，所以我们需要创建一个class来声明这些方法。
```js
//创建一个class
class Depend{
    constructor() {
        //利用Set数据结构防止收集的依赖函数重复
        this.reactiveFns=new Set()
    }
    addDepend(reactiveFn){
        //将依赖函数添加到Set数据结构中
        this.reactiveFns.add(reactiveFn)
    }
    
    notify(){
        //利用forEach遍历Set数据结构，依次执行watchFn函数
        this.reactiveFns.forEach(fn=>{
            fn()
        })
    }
}
```
随后我们就可以创建一个reactive函数，传入一个obj对象，返回一个响应式（Proxy）的对象，自动绑定为响应式。
```js
function reactive(obj){
    return new Proxy(obj,{
        get(target, key, receiver) {
            
            return Reflect.get(target,key,receiver)
        },
        set(target, key, newValue, receiver) {
            Reflect.set(target,key,newValue,receiver)
            
        }
    })
}
```
接下来，我们就可以为对象中的每一个元素属性进行依赖收集，在这里我们用到了WeakMap以及Map数据结构，因为每一个元素的依赖都不一样，所以每一个元素都要new一个Depend对象存储在Map数据结构中（一个Map对应着一个对象），最终用WeakMap将所有的Map存储起来，这里用WeakMap的原因是WeakMap的key是弱引用，当存储的Map指针指向为null时，这个Map就会被GC垃圾回收，可以防止内存泄漏。

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42643cc767a64613a11b1889b39c1eb2~tplv-k3u1fbpfcp-watermark.image?" alt="c42897359ca89d8cf08d9383051236a.jpg" width="100%" />

所以接下来我们需要构造一个getDepend函数。
```js
//创建一个WeakMap的数据结构
const targetMap=new WeakMap()
function getDepend(target,key){
    //根据target获取map
    let map=targetMap.get(target)
    //第一次get肯定没有map，所以需要new
    if(!map){
        map = new Map()
        targetMap.set(target,map)
    }
    //根据key获取depend
    let depend=map.get(key)
    //第一次get肯定没有depend，所以需要new
    if(!depend){
        depend = new Depend()
        map.set(key,depend)
    }
    return depend
}
```
当我们构造了一个getDepend函数之后，我们就可以在Proxy中添加getDepend函数，完成响应式。


在此之前我们需要构造watchFn函数。
```
function watchFn(fn){
    fn()
}
```

```js
new Proxy(obj,{
    get(target, key, receiver) {
        //依赖收集
        //根据target，key获取对应的depend
        const depend=getDepend(target,key)
        //给depend添加函数
        depend.addDepend(activeReactiveFn)

        return Reflect.get(target,key,receiver)
    },
    set(target, key, newValue, receiver) {
        Reflect.set(target,key,newValue,receiver)
        // depend.notify()
        const depend=getDepend(target,key)
        depend.notify()
    }
```
在set方法中，调用notify（）方法执行所有的watchFn函数，在get方法中，收集所有的依赖函数，因为依赖函数依赖对象中的元素属性，所以当依赖函数调用时，会调用Proxy中的get方法，所以我们可以在get方法中进行依赖函数的收集，所以，我们需要在全局声明一个activeReactiveFn=null,当依赖函数调用之前，使activeReactiveFn指向这个依赖函数，当依赖函数调用get方法时，将这个依赖函数添加到reactiveFns中。具体代码如下
```js
let activeReactiveFn=null

function watchFn(fn){
    activeReactiveFn=fn
    fn()
    activeReactiveFn=null
}

```
当然，我们也可以对addDepend进行优化，也就是说我们直接可以调用depend.depend()函数，自动添加依赖函数到reactiveFns中，不用传参，所以我们需要在class中添加一个depend（）方法。
```
class Depend{
    constructor() {
        this.reactiveFns=new Set()
    }
    addDepend(reactiveFn){
        this.reactiveFns.add(reactiveFn)
    }
    depend(){
        if(activeReactiveFn){
            this.reactiveFns.add(activeReactiveFn)
        }
    }
    notify(){
        this.reactiveFns.forEach(fn=>{
            fn()
        })
    }
}
```
随后，我们就可以在get方法中这样调用。

```
get(target, key, receiver) {
    //依赖收集
    //根据target，key获取对应的depend
    const depend=getDepend(target,key)
    //给depend添加函数
    // depend.addDepend(activeReactiveFn)
    depend.depend()

    return Reflect.get(target,key,receiver)
},
```

至此，Proxy的依赖收集，手写结束！🥳🥳🥳。

## 总结 🥂
1. 对象中的每一个元素属性都有对应的一个Depend对象（reactiveFns）。
2. 使用WeakMap存储对象（Map），使用Map存储Depend对象。
3. 使用Set数据结构来存储reactiveFns，防止收集依赖函数重复。
4. 使用Proxy中的set方法来执行reactiveFns。
5. 使用Proxy中的get方法来收集依赖函数。



