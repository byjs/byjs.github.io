---
title: React - 核心概念

date: 2019-05-01

tag: React

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/3.jpeg
---

# hello world

```js
ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('root')
);
```

# JSX (语法糖)

```js
const element = <h1>Hello, world!</h1>;
```

- JSX 仅仅只是 React.createElement(component, props, ...children) 函数的语法糖
- jsx包含 `视图与事件`
- 可嵌入表达式
- 可包含多个子元素
- 防止注入攻击：渲染前，输入内容会被转义(字符串)

```js
const title = response.potentiallyMaliciousInput;
// 直接使用是安全的：
const element = <h1>{title}</h1>;
```

- `Babel` 把JSX转义成 `React.createElement()` 函数调用

```js
const element = (
    <h1 className="greeting">
        Hello, world!
    </h1>
);
// 不使用JSX 等价
const element = React.createElement(
    'h1',
    {className: 'greeting'},
    'Hello, world!'
);
```

```js
// 预先检查，实际内部（简化过的结构）
const element = {
    type: 'h1',
    props: {
        className: 'greeting',
        children: 'Hello, world!'
    }
};
```

# 深入 JSX

## 指定 React 元素类型

- React 必须在作用域内, React 库也必须包含在 JSX 代码作用域内

```js
// 如果不使用 JavaScript 打包工具而是直接通过 <script> 标签加载 React，必须将 React 挂载到全局变量中
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
    // return React.createElement(CustomButton, {color: 'red'}, null);
    return <CustomButton color="red"/>;
}
```

- 在 JSX 类型中使用 `点语法`

```js
import React from 'react';

const MyComponents = {
    DatePicker: function DatePicker(props) {
        return <div>Imagine a {props.color} datepicker here.</div>;
    }
}

function BlueDatePicker() {
    return <MyComponents.DatePicker color="blue"/>;
}
```

- 自定义组件必须以大写字母开头
- 在运行时选择类型，不能将通用表达式作为 React 元素类型

```js
const components = {
    photo: PhotoStory,
    video: VideoStory
};

function Story(props) {
    // JSX 类型可以是大写字母开头的变量。
    const SpecificStory = components[props.storyType];
    return <SpecificStory story={props.story}/>;
}
```

## JSX 中的 Props

- js表达式

```js
<MyComponent foo={1 + 2 + 3 + 4}/>
```

- 字符串字面量

```js
<MyComponent message="hello world"/>
// 等价
<MyComponent message={'hello world'}/>
```

- Props 默认值为 True

```js
<MyTextBox autocomplete/>
// 等价
<MyTextBox autocomplete={true}/>
```

- 属性展开传递

```js
function App2() {
    const props = {firstName: 'Ben', lastName: 'Hector'};
    return <Greeting {...props} />;
}
```

## JSX 中的子元素

- JSX 表达式作为特定属性 `props.children` 传递给外层组件
- 允许由多个 JSX 元素组成

```js
// 字符串字面量 与 JSX 子元素 可以一起使用
// JSX 类似 HTML，以下是合法的 JSX 并且也是合法的 HTML
<div>
    Here is a list:
    <ul>
        <li>Item 1</li>
        <li>Item 2</li>
    </ul>
</div>
```

```js
// React 组件也能返回存储在数组中的一组元素
render()
{
    // 不需要额外元素包裹列表
    return [
        // 不要忘记设置 key
        <li key="A">First item</li>,
        <li key="B">Second item</li>,
        <li key="C">Third item</li>,
    ];
}
```

- js表达式作为子元素

```js
function Hello(props) {
    return <div>Hello {props.addressee}!</div>;
}
```

- 函数作为子元素

```js
// 调用子元素回调 numTimes 次，来重复生成组件
function Repeat(props) {
    let items = [];
    for (let i = 0; i < props.numTimes; i++) {
        items.push(props.children(i));
    }
    return <div>{items}</div>;
}

function ListOfTenThings() {
    return (
        <Repeat numTimes={10}>
            {(index) => <div key={index}>This is item {index} in the list</div>}
        </Repeat>
    );
}
```

- 布尔类型、Null、Undefined 不会渲染

```js
// 渲染结果相同
<div/>

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

- 依据特定条件渲染 React 元素

```js
<div>
    {showHeader && <Header/>}
    <Content/>
</div>
```

- ⚠️ 有一些 false 值，如数字 0，仍会被 React 渲染

```js
// {props.messages.length}会被渲染
<div>
    {0 < props.messages.length &&
        <MessageList messages={props.messages}/>
    }
</div>
```

# 元素渲染

- React元素是`开销极小`的`普通对象`
- React Dom负责 `更新Dom` 与 React元素保持一致

## 元素渲染为DOM

- React构建的应用只有单一的根DOM节点，已有应用可以包含多个独立根DOM节点

## 更新已渲染的元素

- React元素是`不可变对象`，一旦创建无法修改它的子元素或属性
- 一个元素代表了特定时刻的UI

## React 只更新它需要更新的部分

- React DOM 会将元素和它的子元素与它们之前的状态进行比较，只会进行`必要的更新DOM`

# 组件 & Props

```js
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}

class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}
```

## 渲染组件

- React元素可以是 `DOM标签` 或 `自定义组件`
- 自定义组件：JSX接收的属性转化为单个对象传递 称为 `props对象`

## 组合组件

- 每个新的React应用的顶层都是App组件 不包含集成情况

## 提取组件

- 嵌套复杂难维护应提取组件，命名从组件自身角度命名，不依赖上下文调用

## Props 的只读性

- 所有 React 组件都必须像`纯函数`一样，它们的 props 不能被更改

```js
// 不会更改自己的入参
function sum(a, b) {
    return a + b;
}
```

# State & 生命周期

- State 与 props 类似，state 是`私有的`,完全受控于当前组件

## 函数组件转化为class组件

```js
class Clock extends React.Component {
    render() {
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}

// 每次组件更新 render调用，只要在相同的DOM节点中渲染，就只有一个组件的class实例被创建使用
```

## class组件添加局部的state

```js
class Clock extends React.Component {
    constructor(props) {
        super(props);
        // Class 组件应该始终使用 props 参数调用父类的构造函数
        this.state = {date: new Date()};
    }

    render() {
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}

ReactDOM.render(
    <Clock/>,
    document.getElementById('root')
)
```

## 生命周期方法添加到 Class 中

```js
// 1. Clock 被传给 ReactDOM.render()，react调用组件的构造函数
// 2. React调用组件的render
// 3. Clock插入DOM，调用componentDidMount(),组件向浏览器请求设置定时器调用tick方法
// 4. 浏览器每秒调用一次tick方法，Clock组件通过setState()更新UI state改变重新调用render()渲染输出 更新DOM
// 5. Clock 组件从 DOM 中被移除，React就会调用 componentWillUnmount() 计时器销毁
class Clock extends React.Component {
    constructor(props) {
        super(props);
        this.state = {date: new Date()};
    }

    componentDidMount() {
        this.timerID = setInterval(
            () => this.tick(),
            1000
        );
    }

    componentWillUnmount() {
        clearInterval(this.timerID);
    }

    tick() {
        this.setState({
            date: new Date()
        });
    }

    render() {
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}

ReactDOM.render(
    <Clock/>,
    document.getElementById('root')
);

```

## State 的更新可能是异步的

- 不要直接修改 State

```js
// 可换 =>
this.setState(function (state, props) {
    return {
        counter: state.counter + props.increment
    }

})
```

## State 的更新会被合并（浅合并）

- 调用setState()时，React 会把新对象合并到当前的 state
- state 是局部的或是封装的， 除拥有或设置它的组件，其它组件无法访问
- React 可能把多个 setState() 合并成一个调用

```js
  constructor(props)
{
    super(props);
    this.state = {
        posts: [],
        comments: []
    };
}
// 可单独更新
this.setState({
    posts: response.posts
});
// 完整保留 this.state.posts 完全替换 this.state.comments
this.setState({comments})
```

## 数据是向下流动的

- 任何的 state 总是属于特定的组件，从 state 派生的任何数据或 UI 只能影响树中`低于`它们的组件

```js
// 自定义组件传递参数
<FormattedDate date={this.state.date}/>

// 组件本身无法知道它是来自 Clock 的 state，还是 Clock 的 props
function FormattedDate(props) {
    return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

# 事件处理

- React事件命名 小驼峰
- 使用JSX语法 传入 `事件处理函数` 不是字符串

```js
// 传统HTML
<button onclick="activateLasers()">
    Activate Lasers
</button>

// React
<button onClick={activateLasers}>
    Activate Lasers
</button>
```

- 不能通过返回 false 阻止默认行为, 必须使用 `preventDefault`

```js
// 
<a href="#" onclick="console.log('The link was clicked.'); return false">
    Click me
</a>


function ActionLink() {
    function handleClick(e) {
        // e是合成事件，不必考虑浏览器兼容
        e.preventDefault();
        console.log('The link was clicked.');
    }

    return (
        <a href="#" onClick={handleClick}>
            Click me
        </a>
    );
}
```

- 一般不需要使用 addEventListener 监听DOM元素，在元素初始渲染时添加监听器
- class 的方法默认不会绑定 `this`, 所以调用函数时 this 为 undefined

```js
class Toggle extends React.Component {
    constructor(props) {
        super(props);
        // 为了在回调中使用 `this`，这个绑定是必不可少的 
        this.handleClick = this.handleClick.bind(this);
    }
}
```

- 除bind绑定this外，还可以
    - 如果使用实验性的[public class fields](https://babeljs.io/docs/en/babel-plugin-proposal-class-properties) 语法，可以使用 class
      fields 绑定回调函数，Create React App 默认启用此语法

  ```js
  class LoggingButton extends React.Component {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    // 注意: 这是 *实验性* 语法。
    handleClick = () => {
      console.log('this is:', this);
    }
    render() {
      return (
        <button onClick={this.handleClick}>
          Click me
        </button>
      );
    }
  }
  ```
    - ⚠️ 回调中使用箭头函数：每次渲染组件都会创建不同的回调函数，如果作为props传给子组件，组件会重复渲染，所以在构造器中绑定
  ```js
    handleClick() {
      console.log('this is:', this);
    }
    <button onClick={() => this.handleClick()}>
      Click me
    </button>
  ```
  ## 向事件处理程序传递参数

  ```js
  // 箭头函数：事件对象必须显式传递
  <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>

  // bind绑定：事件对象、更多参数会隐式传递
  <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
  ```

## 条件渲染

- JSX中嵌入任意表达式：if 语句、与运算符 &&、三目运算
- 隐藏组件 render 直接返回 null（不会影响组件的生命周期）

# 列表 & Key

- key 为确定标识,帮助 React 识别元素的改变
- 没有确定的id, 万不得已可以使用index：列表顺序变化，性能变差，还可能引起组件状态问题
- 数组元素使用的 key 在兄弟节点之间必须唯一

```js
function Blog(props) {
    const sidebar = (
        <ul>
            {props.posts.map((post) =>
                // id唯一
                <li key={post.id}>
                    {post.title}
                </li>
            )}
        </ul>
    );
    // 一个数组渲染两个不同节点，可以使用相同的key
    const content = props.posts.map((post) =>
        <div key={post.id}>
            <h3>{post.title}</h3>
        </div>
    );
...
}
```

# 表单

## 受控组件

- HTML：表单元素自己维护 state，根据输入更新。 React：可变状态保存在组件的 state 属性中 只能通过 `setState()` 更新
- 受控组件：渲染表单的 React 组件控制输入过程中表单发生的操作（输入的值始终由 React 的 state 驱动）

```js

class NameForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {value: ''};

        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }

    handleChange(event) {
        this.setState({value: event.target.value});
    }

    handleSubmit(event) {
        alert('提交的名字: ' + this.state.value);
        event.preventDefault();
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                    名字:
                    <input type="text" value={this.state.value} onChange={this.handleChange}/>
                </label>
                <input type="submit" value="提交"/>
            </form>
        );
    }
}

/*
  显示的值为 this.state.value，这使得 React 的 state 为唯一数据源
  由于 handlechange 在每次按键都会执行并更新 React 的 state，因此显示的值会随着用户输入而更新
  <input type="text"/>、textarea、select类似input 都接受value，使用它来实现受控组件
  <input type="file"> 非受控组件
*/
```

- 处理多个input输入，添加name属性，处理函数根据 event.target.name 的值选择要执行的操作
- 在受控组件上指定 value 的 prop 会阻止用户更改输入

# 状态提升

- 任何可变数据应当只有一个相对应的`唯一数据源`
- 多个组件共用state，提升至它们最近共同父组件中，依靠自上而下的数据流，而不是在组件之间同步 state

```js
// 例：两个输入框同时改变值 
// 作出改变：父组件控制state，自组件改变通知父组件修改state

class Calculator extends React.Component {
    constructor(props) {
        super(props);
        this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
        this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
        this.state = {temperature: '', scale: 'c'};
    }

    handleCelsiusChange(temperature) {
        this.setState({scale: 'c', temperature});
    }

    handleFahrenheitChange(temperature) {
        this.setState({scale: 'f', temperature});
    }

    render() {
        const scale = this.state.scale;
        const temperature = this.state.temperature;
        const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
        const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

        return (
            <div>
                <TemperatureInput
                    scale="c"
                    temperature={celsius}
                    onTemperatureChange={this.handleCelsiusChange}/>
                <TemperatureInput
                    scale="f"
                    temperature={fahrenheit}
                    onTemperatureChange={this.handleFahrenheitChange}/>
                <BoilingVerdict
                    celsius={parseFloat(celsius)}/>
            </div>
        );
    }
}
```

# 组件 vs 继承

- 包含关系：JSX 嵌套，子组件为任意组件
- 组件可以接受任意 props，包括基本数据类型、React 元素、函数

```js
// 将任何东西作为 props 进行传递

function SplitPane(props) {
    return (
        <div className="SplitPane">
            {props.children}
        </div>
    );
}

function App() {
    return (
        <SplitPane
            children={
                <Contacts/>
            }
        />
    );
}

```

- 继承
- 并没有发现需要使用继承来构建组件层次的情况
- 如果想要在组件间复用非 UI 的功能，可以将其提取为一个单独的 JavaScript 模块，如函数、对象或者类。组件可以直接引入（import）而无需通过 extend 继承它

# React 哲学

- 根据UI划分为组件层级
- 创建React静态版本
- 确定应用所需的 state 的最小集合
- 确定 state 放置的位置
    - 找到根据这个 state 渲染的所有组件
    - 找到他们的共同所有者（common owner）组件（在组件层级上高于所有需要该 state 的组件）
    - 该共同所有者组件或者比它层级更高的组件应该拥有该 state
    - 如果你找不到一个合适的位置来存放该 state，就可以直接创建一个新的组件来存放该 state，并将这一新组件置于高于共同所有者组件层级的位置
- 添加反向数据流
    - 父组件传入
