---
title: 迭代器（iterator）、可迭代对象、生成器（generator）到底是什么
date: 2023/04/03 20-20
tag: JavaScript
sticky: 100
category:
 - 前端
---

##  前言
在JS中，我们常常需要使用async以及await语法来发送网络请求等操作，但是这个语法的背后实现原理到底是什么呢？，我们就得需要了解迭代器（iterator）、可迭代对象、生成器（generator）这些知识点也在各种开源库中，广泛应用。

## 什么是迭代器（iterator）
在JS中，迭代器是一个对象，这个对象中有一个next（）方法，每次调用会retrun出一个对象，这个对象中有两个参数，done（bool类型）、value（具体的属性值），当迭代未完全完成时，done的值则会一直为false，当value的值为最后一个时，下一次的next（）调用返回的done则为true，表明迭代完全完成，此时的value值为undefined。

下面则是一个简单的迭代器实现
```js
const obj=['1','i','n']

const iterator=(obj)=>{
    let index=0
    return {
        next(){
            if(index<obj.length){
                return {done:false,value:obj[index++]}
            }
            else{
                return {done:true,value:undefined}
            }
        }
    }
}

const iteratorObj=iterator(obj)

console.log(iteratorObj.next())
console.log(iteratorObj.next())
console.log(iteratorObj.next())
console.log(iteratorObj.next())
```
调用四次next（）方法输出结果如下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7683be7fded5469187808cbefae332b0~tplv-k3u1fbpfcp-watermark.image?)
每一次调用next（）方法就会返回迭代结果。

## 什么是可迭代对象
一种很简单的说法就是，具有迭代器的对象就是可迭代对象，但是这个迭代器的名称为[Symbol.iterator]，
它和迭代器是不同的概念，当一个对象实现了iterable protocol协议时，他就是一个可迭代对象，这个对象的要求是必须实现@@iterator方法，在代码中我们使用Symbol.iterator访问该属性。

以下为简单的可迭代对象代码实现
```js
const iteratorObj={
    obj:['1','i','n'],
    [Symbol.iterator]:function (){
        let index=0
        return{
            next:()=>{
                if(index<this.obj.length){
                    return {done:false,value:this.obj[index++]}
                }
                else{
                    return {done:true,value:undefined}
                }
            } 
        }
    }
}
```
那么可迭代对象有什么用呢？，我们都知道，for of 遍历的对象都需要是有迭代器的对象，比如Set、Map、Array，所以我们简单实现的这个可迭代对象是可以用 for of 遍历的。

所以我们用for of 遍历输出一些结果，结果如下

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5523e567d5974c42b47172f1d986a49d~tplv-k3u1fbpfcp-watermark.image?)
**在这里，我们的next（）方法一定要用箭头函数，将this指针绑定为interatorObj**

## 可迭代对象的应用场景
1. Javascript语法中，for of、展开语法（spread syntax）、yield*（yield*返回的必须是一个可迭代对象）、数组的解构赋值。
2. 创建一些对象时：new Map([Iterable])、new WeakMap([Iterable])、new Set([Iterable])、new WeakSet([Iterable])。
3. 一些方法的调用：Promise.all（Iterable）、Promise.race（Iterable）、Array.from（Iterable）。

### 注意！！！

```js
const newObj={...obj}
```
像这个展开语法，没有用到可迭代对象，而是在ES9（ES2018）中新增的一个特性。

```js
const {done,value}=obj
```
这种对象的解构赋值也是ES9新增的特性

## 什么是生成器
生成器是ES6中新增的一种函数控制、使用的方案、它可以让我们更加灵活的控制函数什么时候继续执行、暂停执行等。

平时我们会编写很多的函数，这些函数终止的条件通常是返回值或者发生了异常。

在我们了解生成器之前，我们需要了解生成器函数。

## 生成器函数
生成器函数也是一个函数，但是和普通的函数有一些区别
1. 生成器函数需要在function的后面加一个符号：*。
2. 生成器函数可以通过yield关键字来控制函数的执行流程。
3. 生成器函数的返回值是一个Generator（生成器）。

生成器事实上是一种特殊的迭代器。
MDN：Instead,they return a special type of iterator,called a **Generator**

下面是一个简单的生成器函数代码
```js
function* foo(){
    const value1='1'
    console.log(value1)
    yield
    const value2='i'
    console.log(value2)
    yield
    const value3='n'
    console.log(value3)
    yield
}
const generator=foo()
generator.next()
generator.next()
generator.next()
```
执行结果如下

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e17e01defcb94ea0b7fb4a7478bf9dea~tplv-k3u1fbpfcp-watermark.image?)
生成器函数通过yield关键字控制函数的执行，当我们每一次调用next（）方法时，就会调用yield对应顺序的以下代码。

### yield关键字传出参数
当然，我们也可以通过在yield后边跟参数，将参数传出，并在next（）方法调用时接收

```js
function* foo(){
    const value1='1'
    console.log(value1)
    yield 'export 1'
    const value2='i'
    console.log(value2)
    yield 'export i'
    const value3='n'
    console.log(value3)
    yield 'export n'
}
const generator=foo()
const word1=generator.next()
const word2=generator.next()
const word3=generator.next()
console.log(word1)
console.log(word2)
console.log(word3)
```
输出结果如下

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c510e993c4a42138fa923a81b0af9ce~tplv-k3u1fbpfcp-watermark.image?)

### next（）方法调用传入参数
当然，我们也可以在next（）方法调用时，传入参数，并用每一次yield执行时接收参数。
```js
function* foo(){
    const value1='1'
    console.log(value1)
    const s1=yield 'export 1'
    console.log(s1)
    const value2='i'
    console.log(value2)
    const s2=yield 'export i'
    console.log(s2)
    const value3='n'
    console.log(value3)
    const s3= yield 'export n'
    console.log(s3)
}
const generator=foo()
const word1=generator.next()
const word2=generator.next('incoming 1')
const word3=generator.next('incoming i')
const word4=generator.next('incoming n')
console.log(word1)
console.log(word2)
console.log(word3)
console.log(word4)
```
执行结果如下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0b82171c6b44685a5a52d383cf17c75~tplv-k3u1fbpfcp-watermark.image?)

### return方法以及throw方法
通过调用return（）方法可以直接return出函数，不会再执行后边的函数，也无法用yield控制函数执行的过程，next（）方法也无法执行，throw方法则可以抛出异常。通过try catch方法去捕获错误。

## yield*
在生成器函数中 yield可以传出参数，但是当我们用yield* 传出参数时，传出的参数必须是可迭代对象，async和await语法糖也是用到了yield*实现。

## 总结
1. 迭代器是一个对象，这个对象中有一个next（）方法，每次调用会retrun出一个对象，这个对象中有两个参数，done（bool类型）、value（具体的属性值）
2. 当一个对象实现了iterable protocol协议时，他就是一个可迭代对象，这个对象的要求是必须实现@@iterator方法，在代码中我们使用Symbol.iterator访问该属性。
3. 生成器是ES6中新增的一种函数控制、使用的方案、它可以让我们更加灵活的控制函数什么时候继续执行、暂停执行等。
4. 生成器事实上是一种特殊的迭代器。
5. yield关键字传出参数、next（）方法调用传入参数。
