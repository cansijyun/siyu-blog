# siyu-blog
python docker pycharm https://juejin.im/post/5d4d12bb518825053c7d5e12
js promise async await  https://www.jianshu.com/p/b16e7c9e1f9f


# Promise（async/await）和 observable/rxjs

### 特点

1. 对象的状态不受外界影响 （3种状态） 
   - Pending状态（进行中）
   - Fulfilled状态（已成功）
   - Rejected状态（已失败）
2. 一旦状态改变就不会再变 （两种状态改变：成功或失败） 
   - Pending -> Fulfilled
   - Pending -> Rejected

### 用法

#### 创建Promise实例



```jsx
var promise = new Promise(function(resolve, reject){
    // ... some code
    
    if (/* 异步操作成功 */) {
        resolve(value);
    } else {
        reject(error);
    }
})
```

  Promise构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由JavaScript引擎提供，不用自己部署。
   resolve作用是将Promise对象状态由“未完成”变为“成功”，也就是`Pending -> Fulfilled`，在异步操作成功时调用，并将异步操作的结果作为参数传递出去；而reject函数则是将Promise对象状态由“未完成”变为“失败”，也就是`Pending -> Rejected`，在异步操作失败时调用，并将异步操作的结果作为参数传递出去。

#### then

  Promise实例生成后，可用`then`方法分别指定两种状态回调参数。then 方法可以接受两个回调函数作为参数：

1. Promise对象状态改为Resolved时调用 （必选）
2. Promise对象状态改为Rejected时调用 （可选）

#### 基本用法示例



```jsx
function sleep(ms) {
    return new Promise(function(resolve, reject) {
        setTimeout(resolve, ms);
    })
}
sleep(500).then( ()=> console.log("finished"));
```

  这段代码定义了一个函数sleep，调用后，等待了指定参数(500)毫秒后执行then中的函数。值得注意的是，Promise新建后就会立即执行。

#### 执行顺序

  接下来我们探究一下它的执行顺序，看以下代码：



```jsx
let promise = new Promise(function(resolve, reject){
    console.log("AAA");
    resolve()
});
promise.then(() => console.log("BBB"));
console.log("CCC")

// AAA
// CCC
// BBB
```

  执行后，我们发现输出顺序总是 `AAA -> CCC -> BBB`。表明，在Promise新建后会立即执行，所以`首先输出 AAA`。然后，then方法指定的回调函数将在当前脚本所有同步任务执行完后才会执行，所以`BBB 最后输出`。

#### 与定时器混用

  首先看一个实例：



```jsx
let promise = new Promise(function(resolve, reject){
    console.log("1");
    resolve();
});
setTimeout(()=>console.log("2"), 0);
promise.then(() => console.log("3"));
console.log("4");

// 1
// 4
// 3
// 2
```

  可以看到，结果输出顺序总是：`1 -> 4 -> 3 -> 2`。1与4的顺序不必再说，而2与3先输出Promise的then，而后输出定时器任务。原因则是Promise属于JavaScript引擎内部任务，而setTimeout则是浏览器API，而引擎内部任务优先级高于浏览器API任务，所以有此结果。

### **拓展 async/await

#### async

  顾名思义，异步。async函数对 Generator 函数的改进，async 函数必定返回 Promise，我们把所有返回 Promise 的函数都可以认为是异步函数。特点体现在以下四点：

- 内置执行器
- 更好的语义
- 更广的适用性
- 返回值是 Promise

#### await

  顾名思义，等待。正常情况下，await命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。另一种情况是，await命令后面是一个thenable对象（即定义then方法的对象），那么await会将其等同于 Promise 对象。

#### 混合使用

  先看示例：



```jsx
function sleep(ms) {
    return new Promise(function(resolve, reject) {
        setTimeout(resolve,ms);
    })
}
async function handle(){
    console.log("AAA")
    await sleep(5000)
    console.log("BBB")
}

handle();

// AAA
// BBB (5000ms后)
```

  我们定义函数sleep，返回一个Promise。然后在handle函数前加上async关键词，这样就定义了一个async函数。在该函数中，利用await来等待一个Promise。

### Promise优缺点

|   优点   |          缺点          |
| :------: | :--------------------: |
| 解决回调 |    无法监测进行状态    |
| 链式调用 | 新建立即执行且无法取消 |
| 减少嵌套 |    内部错误无法抛出    |



# Promise vs Observable

## 如果看成状态机

- Promise 具有 3 个状态：pending、resolved、rejected（如果 Cancelable Promise 正式通过，那么还会增加一个状态）。

- Observable 有 N + 3 个状态：idle、pending、resolved_0、resolved_1 … resolved_N、completed 和 error。

  总结：相比于 Promise 这个有限状态机而言，Observable 既可能是有限状态机，也可能是无限状态机（N 为无穷）。

## 调用情况

- Observable 还具有可订阅性，对于 Cold Observable 而言，只有订阅后才开始起作用

- Promise 一经产生便开始起作用

  总结：在[视频](https://link.jianshu.com?t=https://egghead.io/lessons/rxjs-rxjs-observables-vs-promises)中有详细的介绍。

## 前言

1. promise解决了嵌套地狱的问题，Observable解决了promise只有一个结果，和不可以取消的问题。
2. 使用的是rxjs6版本。
3. 这篇文章是方便使用Observable的API替换Promise的API。

### 正常用法

```js
promise
.then(result => {})
.catch(error => {})
.finally(() => {});

observable.subscribe(
  result => {},
  error => {},
  ()=>{},  // finally
);
```

### then

```js
promise
.then(result => {})
.then(result => {})
.then(result => {})

import { concat } from 'rxjs';
concat(observable0,observable1,observable2).subscribe(
  result => {},
  error => {},
  ()=>{},  // finally
);

// promise
this.getOne().then(data => {
  // 这里返回另外一个Promise
  return this.getTwo(data);
}).then(data => {
  console.log(data);  // 这里打印第二个Promise的值
  return this.getThree(data);
}).then(data => {
  console.log(data); // 这里打印第三个Promise的值
});

// Observable
import { forkJoin, Observable,from,pipe } from 'rxjs';
import { retryWhen, map, mergeMap } from 'rxjs/operators';

from(this.getOne)
.pipe(
    mergeMap(oneData => {
        console.log(oneData)
        return from(this.getTwo)
    }),
    mergeMap(twoData => {
        console.log(twoData)
        return from(this.getThree)
    })
)
.subscribe(threeData => {
    console.log(threeData)
    ...
})
```

### Promise.all

```js
Promise.all([promise0, promise1]).then((result)=>{});

import { forkJoin } from 'rxjs';
forkJoin([observable0, observable1]).subscribe(result => {});
```

### Promise.race

```js
Promise.race([promise0, promise1]).then((result)=>{});

import { race } from 'rxjs/observable/race';
race([observable0, observable1]).subscribe(result => {});
```
