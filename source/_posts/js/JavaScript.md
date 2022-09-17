---
title: JavaScript

date: 2019-05-10

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/52e117cbb8fca1cf66355eb73a060de6c1952bf7.jpg
---

# 闭包

- 函数的嵌套
- 函数内部可以引用外部的变量和参数
- 变量和参数不会被垃圾回收机制回收

# 跨域

## jsonp

- 只能实现get一种请求

## `document.domain` ➕ `iframe`

- 主域相同，子域不同
- 两个都是通过js 强制设置`document.domain`为基础主域，就成同域

## `location.hash` ➕ `iframe`

- a域与b域相互通信，通过中间页c来实现。三个页面不同域之间利用iframe的location.hash传值，相同域之间直接js访问通信

## `window.name`➕`iframe`

- 跨域数据由`iframe` 的 `window.name` 从外域传到本地域

## postMessage

- 页面和打开的新窗口的数据传递
- 多窗口之间消息传递
- 页面与嵌套 `iframe` 消息传递

## CORS

- 前端设置是否带  `cookie xhr.withCredentials = true`

## nginx反向代理

- 通过nginx 配置代理服务器

## nodejs中间件代理

## webSocket 协议

- 非vue：`node + express + http-proxy-middleware` 搭建proxy 服务器
- vue：`node + webpack + webpack-dev-server`

# 事件

## 冒泡：从里到外

```js
// 阻止 
window.event.cancelBubble = true;
||
event.stopPropagation();
```

## 捕获：从外到里

## 委托（代理）：利用事件冒泡

# ajax

- get

```js
// 创建异步对象
var ajax = new XMLHttpRequest()
// 设置请求参数（"请求的类型","请求url"）
ajax.open('get', 'getStar.php?name= ' + name)
// 发送请求
ajax.send()
// 注册事件 onreadystatechange 状态改变就会调用
ajax.onreadystatechange = function () {
    if (ajax.reagyState == 4 && ajax.status == 200) {
        console.log(ajax.responseText)
    }
}
```

- post

```js
// 创建异步对象
var ajax = new XMLHttpRequest()
// post 请求必须添加请求头
xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
// 设置请求参数（"请求的类型","请求url"）
ajax.open('get', 'getStar.php')
// 发送请求
ajax.send('name=liu')
// 注册事件 onreadystatechange 状态改变就会调用
ajax.onreadystatechange = function () {
    if (ajax.reagyState == 4 && ajax.status == 200) {
        console.log(ajax.responseText)
    }
}
```

# 重绘和回流

- 回流必引起重绘，重绘不一定引起回流
- 避免操作dom、table布局、多层内联样式

## 回流的产生

- 页面首次渲染，开销最大的回流
- 浏览器窗口尺寸、元素位置、文件数量、图片尺寸、元素字体大小改变

## 重绘的产生

- 只是外观改变不影响布局

# 浏览器存储

## sessionStorage

- 临时存储，不支持跨页面，关闭小时并回收

## localStorage

- 长期存储，手动清除，最大存储5M

## cookie

- 服务端 `set cookie` 4k, 不适合放重要数据

## 安全

## xss























































