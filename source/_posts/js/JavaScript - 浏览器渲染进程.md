---
title: v8 的事件循环

date: 2020-12-25

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/311ce36d52f16c1f85ad648b214ab4b4f29dcfab.jpg
---

# 概念
- 事件循环：
  - js分为同步任务和异步任务
  - 所有同步任务都在主线程进行，形成一个执行栈
  - 同步任务进入主线程，异步任务进入 `event Table` 并注册回调函数；等事件完成（如：ajax请求响应返回 setTimeout延迟到指定时间）`event table` 会将这个回调函数移入 `Event Queue`
  - 栈中的代码执行完毕，执行栈任务为空时，主线程会先检查微任务队列中是否有任务，如果有：将微任务队列中的所有任务依次执行，直到队列为空，之后再检查宏任务队列是否有任务，如果有，取出第一个宏任务加入到执行栈中，之前再清空执行栈，检查微任务，以此循环，直到全部的任务都执行完成 
- 浏览器进程
- 插件进程
- GPU进程
- 浏览器渲染进程
    - 主线程
        - 解析、处理script
        - `与渲染线程互斥` 执行过程会导致渲染卡顿
        - `Web Worker`
    - GUI渲染线程
        - 解析`HTML` `CSS` 构建`DOM`，布局和绘制浏览器界面渲染任务
        - `与主线程互斥` （dom与ui的关系）
    - 事件触发线程
        - `事件循环`
        - 由主线程
        - wait时添加至 `event queue` 等待执行
    - http 请求线程
        - XMLHttpRequest 开启新线程请求
        - wait时添加至 `event queue` 等待执行
    - 定时器线程
        - `setTimeout`
        - `setInterval`
        - wait时添加至 `event queue` 等待执行

## 主线程

- 单线程 避免UI渲染时的竞争
- `UI 线程`
- 开启的所有子线程 由主线程统一调度，且不可操作DOM
- 所有同步任务 在主线程执行 形成 `执行栈`

## event queue

- `wait`
- （回调）`事件队列`
- 主线程空了，就会读取`事件`，进入`执行栈`
- macro 宏任务 script start、UI渲染、setTimeout、setInterval、setImmediately、requestAnimationFrame、IO
- micro 微任务: promise、`object.observe`、MutationObserver

```javascript
while (true) {
    if (macro.hasTask()) {
        doMacroTask()
    }
    // 每次取出一个宏任务 执行
    // 首次script 的初始化  等于 一次宏任务

    while (micro.hasTask()) { // 清空微任务
        doMicroTask()
    }

    if (isRepaintTime()) { // 是否应该触发重绘 16.7ms
        doAnimationTask()
    }
}
```







