---
title: React Hooks-useState逻辑实现
date: 2023/05/09 20-16
tag: React
category:
 - 前端
---


##  前言

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/517040d5c49445ac955147e113e61ee1~tplv-k3u1fbpfcp-watermark.image?)
在React中，我们经常使用useState这个Hooks，那么这个Hooks的实现逻辑是怎样的呢，下边我总结了一下这个Hooks的实现逻辑。

## 关于useState
useState是在函数式组件中用来管理状态的一个Hooks，弥补了函数式组件没有状态这一缺点。

在React中，state是以链表的形式存在，并在每一次的更新时，实现state批量更新，也就是说，在React中，当state变化时，会先暂存在更新最后统一实现。

**注意**useState不可以在if这样的条件语句中使用，否则会出现批量更新混淆错误。这也是React官方严厉禁止的。

## useState实现逻辑
首先，我们要明白，useState是通过return出一个数组，这个数组有两个参数，第一个参数是维护的状态值，第二个参数是改变状态值的函数。

### 为什么useState不返回一个对象而返回一个数组呢？
因为返回值为数组时，结构时可以直接起别名，而用对象解构时，起别名的过程需要:（冒号）,这样就会繁琐。

### useState的结构
通过React源码，我们可以发现，useState中是用链表的形式存储着state，所以当我们实现逻辑复现的时候，就得需要通过数组去存储state值和setState函数（简单的逻辑复现暂时用数组）。

## 源码
```js
const MyReact=(
    ()=>{
        const states =[ ]
        const stateSetters=[]

        let stateIndex=0

        function createState(initialState,stateIndex){
            return states[stateIndex]!==undefined ? states[stateIndex] : initialState
        }

        function createStateSetters(stateIndex){
            return function (newState){
                if(typeof newState==="function"){
                    states[stateIndex]=newState(states[stateIndex])
                }
                else{
                    states[stateIndex]=newState
                }

                render()
            }
        }
        function useState(initialState){
            const  _state=createState(initialState,stateIndex)

            if(!stateSetters[stateIndex]){
                stateSetters.push(createStateSetters(stateIndex))
            }
            const _setState=stateSetters[stateIndex]

            stateIndex++
            return [_state,_setState]
        }

        function render(){
            stateIndex=0
            ReactDOM.render(
                <App />,
                document.querySelector('#app')
            )
        }

        return {
            useState
        }
    }

)()


const {useState}=MyReact

export default function App(){
    const [count,setCount]=useState(0)

    return (
        <div>
            <h1>{count}</h1>
            <button onClick={()=>{setCount(count+1)}}>ADD +1</button>
        </div>
    )
}
```
接下来，我们逐步分析源码。

我们都知道，使用useState时需要传入一个init值，所以我们需要接收一个init值，通过createState这个函数，将init值保存在states数组中。在这里，需要做一个判断，如果不等于undefined时，将值设置为**states[stateIndex]**（防止重复初始化，因为render函数调用后，重新执行useState，我们需要保留render调用前的状态值）。

在createStateSetters函数中，根据传入的newState值，返回顶一个setState。

useState的第二个参数，有两种传参形式，一种是传入一个值，一种是传入一个函数，所以我们得需要typeof判断一下是否传入的参数是函数，如果是函数，我们就调用这个函数，并且将这个函数的返回值赋值给states[stateIndex]，如果不是函数，则直接将newState值赋值给states[stateIndex]。

当我们执行一次useState函数值后，将stateIndex+1，确保下一次的useState函数可以正常的将state和setState存入到正确的数组位置。

最后，我们需要在每一次的render函数调用时，将stateIndex重置为0，因为render函数调用后会重新执行useState。
