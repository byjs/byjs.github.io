---
title: React - Api

date: 2019-05-05

tag: React

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/1.jpeg
---

# React 顶层 API

## 组件

- React.Component
- React.PureComponent
    - 仅在props 和 state 简单时使用，在深层数据结构发生变化时调用 forceUpdate() 来确保组件被正确地更新。使用`immutable 对象`加速嵌套数据的比较

- React.memo
    - 只适用函数组件，不适用class组件
    - 仅检查props变化，memo包裹组件并且视线中拥有 useState 或 useContext的hook，发生变化时，仍会重新渲染

```js
  // 默认情况只对复杂对象浅对比
function MyComponent(props) {
    /* 使用 props 渲染 */
}

function areEqual(prevProps, nextProps) {
    /*
    props相等返回true,不相等返回false
    与 shouldComponentUpdate 返回值相反（不等于返回true）
    */
}

export default React.memo(MyComponent, areEqual);
```

## 创建 React 元素

- createElement()

```js
React.createElement(
    type, // 标签字符串（'div'）、React组件类型、Fragment
    [props],
    [...children]
)
```

- createFactory()

```js
// 用于生成指定类型React元素的函数， 类型参数与createElement相似
React.createFactory(type)
```

## 操作元素

- cloneElement()

```js
React.cloneElement(
    element,
    [props],
    [...children]
)
// 几乎等同于

< element.type
{...
    element.props
}
{...
    props
}
>
{
    children
}
</element.type>
```

- isValidElement()

```js
// 是否为React元素
React.isValidElement(object)
```

- React.Children
    - React.Children.map
    - React.Children.forEach
    - React.Children.count
    - React.Children.only
    - React.Children.toArray

## Fragments

- React.Fragment（ <></> ）

## Refs

- React.createRef
- React.forwardRef

## Suspense

- React.lazy
- React.Suspense（懒加载组件）

# React.Component

- 组件的生命周期
    - 挂载
        - **constructor()**
        - static getDerivedStateFromProps()
        - **render()**
        - **componentDidMount()**
    - 更新
        - static getDerivedStateFromProps()
        - **shouldComponentUpdate()**
        - **render()**
        - getSnapshotBeforeUpdate()
        - **componentDidUpdate()**
    - 卸载
        - componentWillUnmount()
    - 错误处理
        - static getDerivedStateFromError()
        - componentDidCatch()
- 其他 APIS
    - 额外API
        - setState()

```js
setState(updater, [callback])
// callback在 setState 完成合并并重新渲染组件后执行，建议使用componentDidUpdate()代替
```

    - forceUpdate()
      - render() 依赖其他数据，调用此方法强制组件重新渲染

```js
component.forceUpdate(callback)
// 调用此方法使组件调用 render() 方法，跳过该组件的shouldComponentUpdate()，其子组件会触发正常生命周期
```

- class 属性
    - defaultProps
    - displayName
- 实例属性
    - props
        - ⚠️：this.props.children 是特殊的prop，通常由 JSX表达式中的子组件组成
    - state

- render()
    - class组件中唯一必须实现的方法
    - 应该为纯函数，不修改组件state，每次调用返回相同的结果，它不会直接与浏览器交互
    - 如需与浏览器交互，在componentDidMount()或其他生命周期中执行
    - ⚠️ shouldComponentUpdate()返回 false，则不会调用render()
    - render被调用时，检查 `this.props` `this.state`的变化并返回类型：
        - React元素（DOM节点｜自定义组件）
        - 数组或fragment（多个元素）
        - portal
        - 字符串或数值类型（文本节点）
        - 布尔类型或null（什么都不渲染）

- constructor()
    - 不初始化state 或 不绑定方法，不需要使用
    - 在为React.Component子类实现构造函数时，应在其他语句之前调用 `super(props)`（this.props在构造函数中定义）
    - 用于两种情况
        - 通过给 `this.state` 赋值对象来初始化内部state
        - 为事件处理函数绑定实例
    - ⚠️ constructor()函数中不要调用setState()方法
    - ⚠️ 避免将 props 的值 赋值给 state

```js
  constructor(props)
{
    super(props);
    // 不要这样做
    this.state = {color: props.color};
}
// 更新prop中的color，并不会影响state
```

- componentDidMount()
    - 组件挂载后（插入DOM树中）立即调用，依赖DOM节点初始化应该放在这
    - 适合添加订阅

- componentDidUpdate()
    - 首次渲染不会执行，更新后被立即调用，可在此处对DOM进行操作
    - 调用setState() 必须被条件语句包裹，否则会死循环
    - ⚠️ shouldComponentUpdate()返回 false，componentDidUpdate()

- componentWillUnmount()
    - 组件卸载、销毁之前立即调用，执行必要的清理操作（清除timer，取消订阅）a
    - 不应调用setState()，该组件永远不会重新渲染。
    - 组件实例卸载后，永远不会在挂载它

- 不常用的生命周期

    - shouldComponentUpdate()
        - state每次发生变化组件都会重新渲染，返回值默认为true，
        - 仅作为性能优化的方式存在，不要靠它来阻止渲染：考虑使用内置 `PUreComponent`组件
        - 不建议深层比较或使用 `JSON.stringify()` 非常影响效率，且会损害性能
        - 返回false，则不会调用之后的生命周期

    - static getDerivedStateFromProps()
        - render方法之前调用，返回一个对象更新state
        - 适用：state的值在任何时候都取决于props
        - ⚠️：不管什么原因，都会在每次渲染前触发此方法

    - getSnapshotBeforeUpdate()
        - 在render之后调用，组件能在发生改变之前从DOM获取信息，任何返回值作为参数传递给 `componentDidUpdate()`
        - 可能出现在UI处理中（处理滚动位置）

    - Error boundaries（错误边界详解）
        - React组件，会在子组件树中的任何位置捕获js错误并记录，展示降级UI而不是崩溃的组件书
        - ⚠️：无法捕获自身错误

    - static getDerivedStateFromError()
        - 会在渲染阶段调用

    - componentDidCatch()
        - 提交 阶段被调用，用于记录错误

# ReactDom

- 通过`<script>` 标签引入 React，所有顶层API都能在全局ReactDom上调用

## 可在应用顶层使用的DOM方法

- render()
    - 首次调用，容器节点中所有DOM元素都会替换，后续使用diff算法更新
    - 不会修改容器节点，只会修改容器的子节点

```js
ReactDOM.render(element, container[, callback
])
```

- hydrate()
  - 

```js
ReactDOM.hydrate(element, container[, callback
])
```

- unmountComponentAtNode()
    - 从DOM中卸载组件，事件处理器和state一并清除

```js
// 没有指定容器，函数什么也不做
ReactDOM.unmountComponentAtNode(container)
```

- findDOMNode()
    - 大多数情况可以绑定ref到DOM节点，可以避免使用此方法
    - ⚠️：一个访问底层DOM节点的应急方案，大多数情况不推荐使用。严格模式下该方法已弃用
    - 只在已放置在DOM中的组件可用，如调用未挂载组件，会异常
    - 不能用于函数组件

```js
ReactDOM.findDOMNode(component)
```

- createPortal()

```js
// 创建portal
ReactDOM.createPortal(child, container)

```

# ReactDOMClient

# ReactDOMServer

- 允许将组件渲染成静态标记，通常被使用在Node服务端上

```js
// ES modules
import ReactDOMServer from 'react-dom/server';
// CommonJS
var ReactDOMServer = require('react-dom/server');
```

- 可以被使用在服务端和浏览器环境
    - renderToString()
        - 在服务端生成HTML，在首次请求时标记加快页面加载速度进行SEO优化

```js
  ReactDOMServer.renderToString(element)
```

    - renderToStaticMarkup()
        - 与renderToString相似，但他不会在React内部创建额外DOM属性

- 只能在服务端使用（输出HTML完全等同于上面）
    - renderToNodeStream()
    - renderToStaticNodeStream()

# DOM 元素

- 所有DOM 特性和属性（包括事件处理）都是小驼峰命名

## 属性差异

- checked
    - `<input>` 组件的 type 类型为 `checkbox`、`radio`，组件支持 checked属性，`defaultChecked`是非受控组件的属性

- className
- danderouslySetInnerHTML
    - React为浏览器DOM提供 innerHTML的替换方案
    - 直接设置容易暴露用户

```js
function createMarkup() {
    return {__html: 'First &middot; Second'};
}

function MyComponent() {
    return <div dangerouslySetInnerHTML={createMarkup()}/>`
}
```

- htmlFor
- onChange
- selected
    - <option> 组件支持 selected 
- style
    - 多用于在渲染过程中添加动态计算的样式
    - React自动添加'px'    `<div style={{ height: 10 }}> </div>`

- suppressContentEditableWarning
    - React 服务端渲染会在服务端与客户端渲染不同的内容时发出警告
    - 为 true，则不会警告，它只对元素一级深度有效

- value
    - `<input>`、`<textarea>` 支持value，`defaultValue` 是非受控组件的属性

- All Supported HTML Attributes（自定义DOM属性：全为小写）

# 合成事件

# Test Utilities（React组件测试）

```js
import ReactTestUtils from 'react-dom/test-utils'; // ES6
var ReactTestUtils = require('react-dom/test-utils'); // ES5 使用 npm 的方式
```

- act()
- mockComponent()
- isElement()
- isElementOfType()

```js
isElementOfType(
    element,
    componentClass
)
```

- isDOMComponent()
- isCompositeComponent()
- isCompositeComponentWithType()
- findAllInRenderedTree()

```js
// 遍历所有在参数 tree 中的组件，记录所有 test(component) 为 true 的组件
findAllInRenderedTree(
    tree,
    test
)
```

- scryRenderedDOMComponentsWithClass()

```javascrip
// 查找渲染树中组件的所有 DOM 元素， css 与className 匹配的 DOM 组件
scryRenderedDOMComponentsWithClass(
  tree,
  className
)
```

- findRenderedDOMComponentWithClass() 与上一致，找到返回一个结果，相反抛异常
- scryRenderedDOMComponentsWithTag()
- findRenderedDOMComponentWithTag()
- scryRenderedComponentsWithType

```js
scryRenderedComponentsWithType(
    tree,
    componentClass
)
```

- findRenderedComponentWithType()

- renderIntoDocument()

```js
// 渲染 React 元素到 document 中的某个单独的 DOM 节点上。
renderIntoDocument(element)
// 相当于

const domContainer = document.createElement('div');
ReactDOM.render(element, domContainer);

// ⚠️：在引入 React 之前确保 window 存在，window.document 和 window.document.createElement 能在全局环境中获取到
```

- Simulate

```js
// 使用可选的 eventData 事件数据来模拟在 DOM 节点上触发事件。

Simulate.
{
    eventName
}
(
    element,
        [eventData]
)

// 点击元素

// <button ref={(node) => this.button = node}>...</button>
const node = this.button;
ReactTestUtils.Simulate.click(node);

```

# Test Renderer

- 提供React渲染器，用于将组件渲染成js对象，无需依赖DOM、原生移动环境
- 不依赖浏览器或jsdom情况下，返回某个时间点由 React DOM 或者 React Native 平台渲染出的视图结构（类似与 DOM 树）快照

```js
import TestRenderer from 'react-test-renderer'; // ES6
const TestRenderer = require('react-test-renderer'); // ES5 with npm

// example

import TestRenderer from 'react-test-renderer';

function Link(props) {
    return <a href={props.page}>{props.children}</a>;
}

const testRenderer = TestRenderer.create(
    <Link page="https://www.facebook.com/">Facebook</Link>
);

console.log(testRenderer.toJSON());
// { type: 'a',
//   props: { href: 'https://www.facebook.com/' },
//   children: [ 'Facebook' ] }
```

## TestRenderer

- TestRenderer.create()
    - 它并不使用真实的 DOM，但是它依然将组件树完整地渲染到内存，以便于你对它进行断言

```js
// 创建 TestRenderer 实例
TestRenderer.create(element, options);
```

- TestRenderer.act()

```js
TestRenderer.act(callback);
```

## TestRenderer instance

- testRenderer.toJSON()
    - 返回一个已渲染的的树对象
- testRenderer.toTree()
    - 内容比 toJSON() 提供的内容要更加详细，并且包含用户编写的组件
    - 除非在测试渲染器（test renderer）之上编写自己的断言库，否则可能不需要这个

- testRenderer.update()

```js
// 使用新的根元素重新渲染内存中的树
// 新的元素和之前的元素有相同的 type 和 key，该树将会被更新；否则，它将重挂载一个新树
testRenderer.update(element)
```

- testRenderer.unmount()
    - 卸载内存中的树，会触发相应的生命周期事件
- testRenderer.getInstance()
    - 返回与根元素相对应的实例。如果根元素是函数定义组件，该方法无效，因为函数定义组件没有实例
- testRenderer.root
    - 返回根元素“测试实例”对象，利用它查找其他更深层的“测试实例”

## TestInstance

- testInstance.find()

```js
// 找到一个 test(testInstance) 返回 true 的后代测试实例,如果多个测试实例匹配，将会报错
testInstance.find(test)
```

- testInstance.findByType()

```js
// 如果多个测试实例匹配，将会报错
testInstance.findByType(type)
```

- testInstance.findByProps()

```js
// 如果多个测试实例匹配，将会报错
testInstance.findByProps(props)
```

- testInstance.findAll()
- testInstance.findAllByType()
- testInstance.findAllByProps()
- testInstance.instance
    - 该测试实例相对应的组件实例
    - 只能用于类组件
- testInstance.type
- testInstance.props
- testInstance.parent
- testInstance.children

# js 环境要求

- React依赖集合类型`Map`和`Set`，兼容浏览器与设备，在应用库中包含全局的polyfill（core-js 或 babel-polyfill ）

```js
import 'core-js/es/map';
import 'core-js/es/set';

import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('root')
);
```

# React 术语词汇表

## 单页面应用

- 一个应用程序，可以加载单个HTML页面和程序所需所有的必要资源（js、css）
- 与页面的任何交互，都不需要加载资源，页面不会重新加载

## Compiler（编译器）

- 常见用例：接受ES6 语法，转换为旧版本浏览器能够执行的语法（Babel）

## Bundler（打包工具）

- 接收单独模块的js和css代码，进行组合（webpack、Browserify）

## Package 管理工具

- 管理项目依赖的工具（npm、yarn：它们都是使用了相同 npm package registry 的客户端）

## CDN

- 内容分发网络
