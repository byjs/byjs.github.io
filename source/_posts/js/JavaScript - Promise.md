---
title: JavaScript - Promise

date: 2019-05-27

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/a50c68a33bd7333fd112ddb575f1b9adc5788d88.jpg
---

# JS的四种异步解决方案：回调函数、Promise、Generator、async/await

## 异步

当前代码的执行不影响后面代码执行。当程序运行到异步的代码时，会将该异步代码作为任务放进任务队列，而不是推入主线程的调用栈。等住线程执行完之后，再去任务队列里执行对应的任务。
异步操作的优点：不会阻塞后续代码执行

### 异步应用场景

- 定时任务：setTimeout、setInterval
- 网络请求：ajax请求、动态创建img标签的加载
- 事件监听器：addEventListener

## 为了解决回调地狱的问题，提出了Promise、async/await、generator

## Promise

- 特点
    - 对象的状态不受外界影响（3种状态）
        - Pending（进行中）
        - Fulfilled（已成功）：resolve()表示
        - Rejected（已失败）：reject()表示
    - 一旦状态改变就不会再变（两种状态改变：成功或失败）
        - Pending -> Fulfilled
        - Pending -> Rejected

```js
var promise = new Promise(function (resolve, reject) {
    //....
    if (/* 异步操作成功 */) {
        resolve(value)
    } else {
        reject(error)
    }

})
```

- 执行顺序
    - Promise会在新建后立即执行
    - then 方法的回调函数在当前所有同步任务执行完后才执行

```js
let promise = new Promise(function (resolve, reject) {
    console.log('AAA')
    resolve()
})
promise.then(() => console.log('BBB'))
console.log('CCC')
// AAA -> CCC -> BBB
```

- 与定时器混用
    - Promise 属于js引擎内部任务，而setTimeout是浏览器API，引擎内部任务高于浏览器API任务：3比2先执行

```js
let promise = new Promise(function (resolve, reject) {
    console.log('1')
    resolve()
})
setTimeout(() => console.log('2'), 0)
promise.then(() => console.log('3'))
console.log('4')

// 1 -> 4 -> 3 -> 2
```

## Generator

- 一个返回值为 `iterator`对象的函数
- 可以在执行过程中多次返回（yield多次执行）

### 特点

- `function`*函数名
- 函数体内部使用yield 语句，定义不同的内部状态

### iterator

- `iterator`：迭代器。它为js中各种不同的数据结构（Object、Array、Set、Map）提供统一的访问机制
- 任何数据结构只要部署了 `Iterator` 接口，就可以完成遍历操作
- 也是一个对象，相比普通对象，它有专为迭代设计的接口

#### 作用

- 为各种数据结构提供统一的、简便的访问接口
- 使数据结构的成员能够按某种次序排列
- Iterator接口主要供`for...of`消费

#### 结构

- next方法：返回一个包含`value`（迭代的值）和`done`（迭代是否完成的标志：完成true）两属性的对象（假设：result）
- 内部有指向迭代位置的指针，每次调用next，自动移动指针并返回相应的result
- 原生具备`Iterator`接口的数据结果：Array、Map、Set、String、TypedArray、函数里的arguments对象、NodeList对象
    - 这些结构都有 `Symbol.iterator` 属性，可以直接通过这个属性直接创建一个迭代器，但这个属性只是一个用来创建迭代器的接口，而不是一个迭代器，它不含遍历的部分

- 使用Symbol.iterator接口生成iterator迭代器的过程：`for...of`循环内部实现机制就是iterator
    - 先调用被遍历对象的 `Symbol.iterator` 方法，返回一个迭代器对象
    - 在for...of每次循环中，都调用该迭代器对象的.next方法
    - `for i of`中i就是调用.next方法后得到对象的value属性

## async / await

- 异步，是Generator的语法糖

- 改进
    - async函数自带执行器，会自动执行。`Generator`函数必须依靠执行器，因为不能一次性执行完
    - async函数 await后面没有限制。`Generator`函数 yield命令后只能是Thunk函数或Promise对象


























