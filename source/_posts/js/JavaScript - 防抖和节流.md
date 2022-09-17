---
title: JavaScript - 防抖和节流

date: 2019-06-06

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/46889eea6d0674bb77690ab8656721804f555de9.jpg
---

```js

```

# 防抖和节流

## 事件高频触发的场景

在JavaScript编程中，如果某个事件在短时间内高频触发，可能就会出现对应的 **回调函数** 响应速度跟不上调用频率的问题，导致页面延迟，假死或卡顿的现象，
`函数防抖（debounce）` `函数节流（throttle）` 就是这类场景下常用的优化手段，来节约系统资源，改善用户体验。

## 原理

- 函数防抖（debounce）：一定时间内，只会执行最后一次函数回调
- 函数节流（throttle）：一定时间内，只执行一次函数回调

## 函数防抖的适用场景 —— 搜索查询

- 函数防抖：高频事件触发后，经过一定的时间间隔才会执行函数回调；如果事件在事件间隔内又被触发，则会重新计时，只会执行最后一次的回调函数

```js
/*
* fn [function] 需要防抖的函数
* delay [number] 毫秒，防抖期限值
*/
function debounce(fn, delay) {
    let timer = null //借助闭包
    return function () {
        console.log('timer', timer)
        if (timer) {
            clearTimeout(timer)
        }
        timer = setTimeout(fn, delay) // 简化写法
    }
}

function inputTap(e) {
    console.log(e.target.value)
}

document.addEventListener('input', debounce(inputTap, 1000))
```

## 函数节流的使用场景 —— 监听滚动条

- 函数节流：连续触发事件但在一定时间内只执行一次函数

### 节流的两种方式：时间戳和定时器

```js
// 时间戳版
const throttle = function (fn, delay) {
    let preTime = Date.now()
    return function () {
        let doTime = Date.now()
        if (delay <= doTime-preTime)
        {
            fn.apply(this, arguments)

            preTime = Date.now()
        }
    }

}
```

```js
// 定时器版
const throttle = function (fn, delay) {
    let timer = null
    return function () {
        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(this, arguments)
                clearTimeout(timer)
                timer = null
            }, delay)
        }
    }
}

```

- debounce

```js
function debounce() {
    var lastCallTime // 最后一次触发事件的事件
    var lastThis // 作用域
    var lastArgs //参数
    var timeId // 定时器对象
    wait = +wait || 0

    //启动定时器
    function startTimer(timerExpired, wait) {
        return setTimeout(timerExpired, wait)
    }

    // func 函数执行
    function invokeFunc() {
        timeId = undefined
        const args = lastArgs
        const thisArg = lastThis
        let result = func.apply(thisArg, args)

        lastArgs = lastThis = undefined
        return result
    }

    // 调用func函数的判定条件
    function shouldInvoke(time) {
        return lastCallTime !== undefined && (wait <= time - lastCallTime)
    }

    function remainingWait(time) {
        const timeSinceLastCall = time - lastCallTime
        const timeWaiting = wait - timeSinceLastCall
        return timeWaiting

    }
}

// 定时器的回调函数
function timerExpired() {
    // 在这里判断触发事件的时间差
    const time = Date.now()
    if (shouldInvoke(time)) {
        return invokeFunc()
    }
    timeId = startTimer((timerExpired, remainingWait(time))
}


/*要返回的函数*/

/* 
* 确定作用域和参数
* 更新触发时间的时间：lastCallTime
* 启动定时器 timerId
* */
function debounced(...args) {
    const time = Date.now()
    lastThis = this
    lastArgs = args
    lastCallTime = time

    if (timeId === undefined) {
        timeId = startTimer(timerExpired, wait)
    }

}

return debounced
}
```
