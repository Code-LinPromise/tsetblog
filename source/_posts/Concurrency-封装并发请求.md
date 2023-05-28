---
title: Concurrency-封装并发请求
date: 2023/05/18/ 22:22
tag: 
 - JavaScript
 - 并发请求
category:
 - 前端
---

## Concurrency
并发请求在平时做项目时经常出现，一般我们可以用Promise.all配合Promise.race可以解决这个问题，我们也可以自己封装一个class去解决这个并发请求。

## 封装需求
用class封装一个ConcurrencyRequest，将所有的请求task通过push函数添加至该class中的taskQueue，并且在new的同时，指定maxConcurrencyRequest，最后，我们可以使用ConcurrencyRequest.responses查看响应结果。

## 目录结构

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/964ad603374842ce9383db13055d0ce7~tplv-k3u1fbpfcp-watermark.image?)

### index.js
因为自己模拟了9个请求，所以代码会有些长并且有很多重复~可以跳过重复，查看核心代码
```js
import ConcurrencyRequest from "./ConcurrencyRequest.js";

const BASE_URL="http://localhost:5000/"

function getTest1(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test1').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest2(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test2').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest3(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test3').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest4(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test4').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest5(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test5').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest6(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test6').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest7(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test7').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest8(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test8').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

function getTest9(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            axios.get(BASE_URL+'test9').then((res)=>{
                resolve(res)
            }).catch(err=>{
                reject(err)
            })
        },2000)
    })
}

const taskQueue=[
    getTest1,
    getTest2,
    getTest3,
    getTest4,
    getTest5,
    getTest6,
    getTest7,
    getTest8,
    getTest9,
]

const concurrencyRequest =new ConcurrencyRequest(
    {
        maxConcurrencyRequest:2
    }
)
for (let task of taskQueue){
    concurrencyRequest.push(task);
}
console.log(concurrencyRequest.responses)
```
### ConcurrencyRequest.js
```js
export default class ConcurrencyRequest{
    constructor({
        maxConcurrencyRequest
    }){
        //获取到传入的最大并发量
        this.maxConcurrencyRequest=maxConcurrencyRequest;
        //声明任务队列
        this.taskQueue=[];
        //声明最后获得的信息对象
        this.responses={};

        //因为在new ConcurrencyRequest后要push task才可以执行并发，所以把执行并发的时机放到push task之后
        setTimeout(()=>{
            this._doRequest();
        },0)
    }

    push(task){
        //将请求任务push到任务队列中
        this.taskQueue.push(task);
    }

    _doRequest(){
        //如果任务队列中没任务时，就不做请求，直接return
        if(!this.taskQueue.length){
            return
        }
        //获取最小值
        const minConcurrencyRequest=getMinCount(
            this.maxConcurrencyRequest,
            this.taskQueue.length);
        
        for(let i=0;i<minConcurrencyRequest;i++){
            const task=this.taskQueue.shift();
            //因为我们需要保证同时最多只能有指定数请求所以我们在请求前需要-1
            this.maxConcurrencyRequest--;
            this._runTask(task);
        }
        

    }

    _runTask(task){
        task().then((res)=>{
            this.responses[task.name]={
                result:res,
                error:null
            }
        }).catch((error)=>{
            this.responses[task.name]={
                result:null,
                error:error
            }
        }).finally(()=>{
            //请求完成后+1
            this.maxConcurrencyRequest++;
            this._doRequest();
        })
    }

}

function getMinCount(count1,count2){
    return Math.min(count1,count2)
}
```
### server-index.js
用express启动的服务器后台，模拟了九个接口，代码有重复，可以直接跳过~
```js
const express =require("express");

const app=express();

app.all('*',(req,res,next)=>{
    //设置请求头，解决跨域
    res.header('Access-Control-Allow-Origin','*');
    //允许get方法访问
    res.header('Access-Control-Allow--Methods','GET');
    next();
})

app.get('/test1',(req,res)=>{
    res.send("test1")
})

app.get('/test2',(req,res)=>{
    res.send("test2")
})

app.get('/test3',(req,res)=>{
    res.send("test3")
})

app.get('/test4',(req,res)=>{
    res.send("test4")
})

app.get('/test5',(req,res)=>{
    res.send("test5")
})

app.get('/test6',(req,res)=>{
    res.send("test6")
})

app.get('/test7',(req,res)=>{
    res.send("test7")
})

app.get('/test8',(req,res)=>{
    res.send("test8")
})

app.get('/test9',(req,res)=>{
    res.send("test9")
})





app.listen(5000,()=>{
    console.log("服务器启动咯~")
})
```
### 执行结果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc401529e5904f6f961294b2dd283909~tplv-k3u1fbpfcp-watermark.image?)

## 总结
1. 首先需要注意我们并发执行的时机，因为需要所有的task push到taskQueue中才执行，所以我们需要将执行函数用setTimeout包裹（放入宏任务队列）。
2. 因为同时只能请求我们指定的请求数，所以我们需要在每一次请求前将maxConcurrencyRequest-1，当请求完成时+1。
