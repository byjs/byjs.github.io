---
title: JavaScript - 微前端

date: 2021-02-06

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/034228206951a9f937222fe57296930f78008cb1.jpg
---

# 微前端

## 概念

- 微前端是一种类似于微服务的架构，将微服务的理念应用于浏览器端，将web应用由单一的单体应用转变为**多个小型前端应用聚合为一的应用**，各个前端应用独立运行、独立开发、独立部署。

- 当前前端应用通常以spa的形态（一个应用所有的相关页面都在一个项目）出现。随着项目迭代，维护成本越来越高。微前端的意义在于 **拆分项目**：细化为若干个可以单独部署的子项目，子项目互相独立，单独部署
- 微前端结构下，不同技术栈的子项目可以共存
- 可以低成本整合已有项目。因为子项目是独立的，不需要太大的工作量就可迁至新项目

## qiankun（乾坤）

- `qiankun` 是一个基于`single-spa` 的微前端库（二次开发）

- `single-spa`：是一个用于微服务化的js前端解决方案
    - 将子应用看作一个spa的页面，主应用通过路由匹配的方式，加载不同的子应用，子应用会经过下载（loaded）、初始化（initialized）、被挂载（unmounted）、被移除(unloaded)等过程

### 主应用

- 安装

```js
 `yarn add qiankun` // `npm i qiankun -S`
```

- 注册微应用

```js
import {registerMicroApps, start} from 'qiankun'

registerMicroApps([
    {
        name: 'react-app',
        entry: '//localhost:7100',
        container: '#yourContainer',
        activeRule: '/yourActiveRule'
    },
    {
        name: 'vue-app',
        entry: {scripts: ['//localhost:7100/main.js']},
        container: '#yourContainer',
        activeRule: '/yourActiveRule'
    }
])

start()

// 浏览器的url变化，触发qiankun的匹配逻辑，所有activeRule规则匹配上的微应用就会被插入到指定的container中，同时依次调用微应用暴露出的生命周期钩子
```

### 微应用

- 导出对应的生命周期钩子（通常是配置的webpack 的 entry）

```js
/* bootstrap 
* 只会在微应用初始化的时候调用一次，下次微应用重新进入会直接调用mount钩子，不会再重复触发bootstrap
* 在这里可以做一些全局变量的初始化，比如不会在 unmount阶段被销毁的应用级别的缓存等
* */
export async function bootstrap() {
    console.log('react app bootstraped')
}

/*
* 应用每次进入都会调用mount 方法，通常在这里触发应用的渲染方法
*/
export async function mount(props) {
    ReactDOM.render(<APP/>, props.container ? props.container.querySelector('#root') : document.getElementById('root'))
}

/*
* 应用每次 切出/卸载 会调用，通常在这里卸载应用的应用实例
* */
export async function unmount(props) {
    ReactDOM.unmountComponentAtNode(props.contains ? props.container.querySelector('#root') : document.getElementById('root'))
}

/*
* 可选生命周期钩子，仅使用 loadMicroApp 方式加载微应用时生效
* */
export async function update(props) {
    console.log('update props', props);
}
```

### 配置微应用的打包工具

- 为了让主应用能正确识别微应用暴露出来的一些信息

```js
const packageName = require('./package.json').name
module.export = {
    output: {
        library: `${packageName}-[name]`,
        libraryTarget: 'umd',
        jsonpFunction: `webpackJsonp_${packageName}`
    }
}
```

### 通信

- 官方提供 `Actions` 通信：适合业务划分清晰，通信较少的微前端应用场景
    - 内部提供了 `initGlobalState` 方法用于注册 `MicroAppStateActions` 实现用于通信，该实例有三个方法
    - `setGlobalState` 设置 `globalState`
    - `onGlobalStateChange`：注册`观察者`函数，响应 `globalState` 变化，在 `globalState` 发生改变时触发该观察者函数
    - `offGlobalStateChange`: 取消`观察者`函数， 该实例不再响应 `globalState` 变化

```js
// 主应用
import {initGlobalState} from 'qiankun'

const initialState = {
    // 初始化数据
}

// 初始化state
const actions = initGlobalState(initialState)

actions.onGlobalStateChange((state, prev) => {
    // 监听公共状态的变化
    console.log('主应用变更前', prev)
    console.log('主应用变更后', state)
})
export default actions

// 组件中
import actions from './actions'

sendMsg()
{
    actions.setGlobalState(this.msg)
}
```

- `localStorage、sessionStorage` 实现：适合跟踪通信状态，类似用户登陆信息


