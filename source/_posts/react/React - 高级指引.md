---
title: React - 高级指引

date: 2019-05-02

tag: React

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/4.jpeg
---

# 代码分割

## 打包

- 大多数 React 应用都会使用 Webpack、Rollup、Browserify 这类构建工具打包文件
- 打包是一个将文件引入并合并到一个单独文件的过程，最终形成一个 “bundle”

## 代码分割

- 打包器 [Rollup](https://rollupjs.org/guide/en/#code-splitting)
    - [Webpack](https://webpack.docschina.org/guides/code-splitting/)
    - Browserify
    - [factor-bundle](https://github.com/browserify/factor-bundle) 能够创建多个包并在运行时动态加载
- 引入代码分割的最佳方式是动态 import()

```js
// Webpack 解析该语法时，会自动进行代码分割
import("./math").then(math => {
    console.log(math.add(16, 26));
});
```

- 使用 Babel 时，确保 Babel 能够解析动态 import 语法而不是将其转换。对于这一要求需要 babel-plugin-syntax-dynamic-import 插件

## React.lazy

- React.lazy 函数能像渲染常规组件一样处理动态引入（的组件）
- 想要在使用服务端渲染的应用中使用，推荐 [Loadable Components](https://github.com/gregberge/loadable-components) 这个库，它有一个很棒的服务端渲染打包指南

```js
import React, {Suspense} from 'react';

// React.lazy 接受一个函数(需要动态调用 import()).它必须返回一个 Promise，该 Promise 需要 resolve 一个 defalut export 的 React 组件
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

// fallback 属性接受任何在组件加载过程中展示的 React 元素
// Suspense 组件中渲染 lazy 组件，可以使用在等待加载 lazy 组件时做优雅降级（loading 等操作）
// Suspense 组件可以置于懒加载组件之上的任何位置，它可以包裹多个懒加载组件

function MyComponent() {
    return (
        <div>
            <Suspense fallback={<div>Loading...</div>}>
                <section>
                    <OtherComponent/>
                    <AnotherComponent/>
                </section>
            </Suspense>
        </div>
    );
}
```

## 基于路由的代码分割

```js
// 使用 React.lazy 和 React Router 这类的第三方库，来配置基于路由的代码分割
import React, {Suspense, lazy} from 'react';
import {BrowserRouter as Router, Route, Switch} from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
    <Router>
        <Suspense fallback={<div>Loading...</div>}>
            <Switch>
                <Route exact path="/" component={Home}/>
                <Route path="/about" component={About}/>
            </Switch>
        </Suspense>
    </Router>
);
```

# Context

## 何时用 Context

- Context 提供一种在组件之间共享此类值的方式，不必显式地通过组件树逐层传递 props
- 应用场景：很多不同层级的组件需要访问同样的数据，`谨慎使用，因为这会使得组件的复用性变差`，包括管理当前的 locale、theme、或者一些缓存数据
- 如果只是想避免层层传递一些属性，组件组合（component composition）有时是比 context 更好的解决方案

```js
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');

class App extends React.Component {
    render() {
        // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
        // 无论多深，任何组件都能读取这个值。
        // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
        return (
            <ThemeContext.Provider value="dark">
                <Toolbar/>
            </ThemeContext.Provider>
        );
    }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
    return (
        <div>
            <ThemedButton/>
        </div>
    );
}

class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context。
    // React 会往上找到最近的 theme Provider，然后使用它的值。
    // 在这个例子中，当前的 theme 值为 “dark”。
    static contextType = ThemeContext;

    render() {
        return <Button theme={this.context}/>;
    }
}
```

## API

- React.createContext

```js
// React 渲染订阅Context 对象的组件，组件会从组件树中离自身最近匹配的 Provider 中读取到当前的 context 值
// 只有组件所处的树中没有匹配到 Provider ，其 defaultValue 参数才会生效
// 将 undefined 传递给 Provider 的 value 时，消费组件的 defaultValue 不会生效
const MyContext = React.createContext(defaultValue);
```

- Context.Provider
    - 每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化
    - Provider 接收一个 value 属性，传递给消费组件
    - 一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据
    - value 变化时，它内部所有消费组件都会重新渲染
    - Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，当 consumer 组件在其祖先组件退出更新的情况下也能更新

```js
<MyContext.Provider value={/* 某个值 */}>
```

- Class.contextType
    - 此属性可以让你使用 this.context 来获取最近 Context 上的值

```js
// 可以在任何生命周期中访问到它，包括 render 函数中
class MyClass extends React.Component {
    componentDidMount() {
        let value = this.context;
        /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
    }

    componentDidUpdate() {
        let value = this.context;
        /* ... */
    }

    componentWillUnmount() {
        let value = this.context;
        /* ... */
    }

    render() {
        let value = this.context;
        /* 基于 MyContext 组件的值进行渲染 */
    }
}

MyClass.contextType = MyContext;

```

- Context.Consumer

```js
// 传递给函数的 value 值等同于往上组件树离这个 context 最近的 Provider 提供的 value 值,没有Provider 就是createContext的defaultValue
<MyContext.Consumer>
    {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>

```

- Context.displayName
    - context 对象接受 displayName 的 property，字符串类型。React DevTools 使用该字符串来确定 context 要显示的内容
  ```js
    const MyContext = React.createContext(/* some value */);
    MyContext.displayName = 'MyDisplayName';

    <MyContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
    <MyContext.Consumer> // "MyDisplayName.Consumer" 在 DevTools 中
  ```
- 动态 Context
    - 在嵌套组件中更新 Context(通过 context 传递一个函数，使得消费组件更新 context)
  ```js
  // 确保传递给 createContext 的默认值数据结构是调用组件能匹配的
  export const ThemeContext = React.createContext({
    theme: themes.dark,
    toggleTheme: () => {},
  });

  // 调用方
  function ThemeTogglerButton() {
  // Theme Toggler 按钮不仅仅只获取 theme 值，
  // 它也从 context 中获取到一个 toggleTheme 函数
    return (
      <ThemeContext.Consumer>
        {({theme, toggleTheme}) => (
          <button
            onClick={toggleTheme}
            style={{backgroundColor: theme.background}}>
            Toggle Theme。。。
          </button>
        )}
      </ThemeContext.Consumer>
    );
  }

  ```
- 注意
    - `context` 根据引用标识决定何时渲染（本质上是 value 属性值的浅比较），陷阱：provider 父组件进行重渲染时，可能会在消费组件中触发意外的渲染
  ```js
  // 每一次 Provider 重渲染时，value 属性总是被赋值为新的对象,会重新渲染下面所有的消费组件
  // 改进：value 状态提升到父节点的 state 
  class App extends React.Component {
    render() {
      return (
        <MyContext.Provider value={{something: 'something'}}> // value={this.state.value}
          <Toolbar />
        </MyContext.Provider>
      );
    }
  }
  ```
- 错误边界
    - 为了解决：部分UI的js错误导致整个应用崩溃
    - 它是React 组件，可以捕获并打印发生在其子组件树任何位置的js错误，会渲染出备用 UI，而不是渲染那些崩溃了的子组件树
    - 在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误
    - 无法捕获的错误
        - 事件处理
        - 异步代码（setTimeout 或 requestAnimationFrame 回调函数）
        - 服务端渲染
        - 它自身抛出来的错误（并非它的子组件）
    - class 组件定义了 static getDerivedStateFromError() 或 componentDidCatch() 任意一个（或两个）时，它就变成一个错误边界
  ```js
  class ErrorBoundary extends React.Component {
    constructor(props) {
      super(props);
      this.state = { hasError: false };
    }

    static getDerivedStateFromError(error) {
      // 更新 state 使下一次渲染能够显示降级后的 UI
      return { hasError: true };
    }

    componentDidCatch(error, errorInfo) {
      // 你同样可以将错误日志上报给服务器
      logErrorToMyService(error, errorInfo);
    }

    render() {
      if (this.state.hasError) {
        // 你可以自定义降级后的 UI 并渲染
        return <h1>Something went wrong.</h1>;
      }

      return this.props.children; 
    }
  }
  ```
    - 工作方式类似于js的 catch {}，不同于错误边界只针对 React 组件
    - 只有 class 组件才可以成为错误边界组件。大多数情况下, 只需要声明一次错误边界组件, 可以在整个应用中使用它
    - 可以包装在最顶层的路由组件展示一个 “Something went wrong” 的错误信息，就像服务端框架处理崩溃一样

## 未捕获错误（Uncaught Errors）的新行为

- 任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载

## 组件栈追踪

- 渲染期间发生的所有错误打印到控制台,仅用于开发环境，`生产环境必须将其禁用`
- 组件名称在栈追踪中的显示依赖于 Function.name 属性
- 如要支持 未提供该功能的旧版浏览器和设备（例如 IE 11），在打包（bundled）应用程序中包含一个 Function.name 的 polyfill（function.name-polyfill）或在所有组件上显式设置
  displayName 属性

## 关于 try/catch

- 仅用于命令式代码
- 错误边界无法捕获事件处理器内部的错误 使用try/catch
- 错误边界保留了 React 的声明性质，例：即使错误发生在 componentDidUpdate 方法中，由某一个深层组件树的 setState 引起，仍然能冒泡到最近的错误边界

# Refs

- Refs 可以`访问DOM节点`、`React元素`
- 使用场景(勿过度使用)
    - 处理焦点，文本选择或媒体播放
    - 触发强制动画
    - 集成第三方DOM库

- 创建 Refs

```js
// React.create()创建，通过 ref 属性附加到元素上
class MyComponent extends React.Component {
    constructor(props) {
        super(props)
        this.myRef = React.createRef()
    }

    render() {
        return <div ref={this.myRef}/>
    }
}
```

- 访问 Refs，节点的类型影响ref的取值
- ref属性指向一个 DOM 元素或 class组件
- 不能在函数组件上使用ref属性，因为它没有实例

```js
function MyFunctionComponent() {
    return <input/>;
}

class Parent extends React.Component {
    constructor(props) {
        super(props);
        this.textInput = React.createRef();
    }

    render() {
        // This will *not* work!
        return (
            <MyFunctionComponent ref={this.textInput}/>
        );
    }
}

// 组件中想使用ref,用下面forwardRef或转换为class组件
```

- 在父组件中引用子节点的DOM节点，使用ref转发

## 转发 refs 到 DOM 组件

- Ref 转发是一个可选特性，组件可以像暴露自己的ref一样暴露子组件的ref

```js
const FancyButton = React.forwardRef((props, ref) => (
    <button ref={ref} className="FancyButton">
        {props.children}
    </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

## 在高阶组件（HOC）中转发 refs

- ref 不是 属性,如果对 HOC 添加 ref，ref 会引用最外层的容器组件，而不是被包裹的组件

```js
function logProps(Component) {
    class LogProps extends React.Component {
        componentDidUpdate(prevProps) {
            console.log('old props:', prevProps);
            console.log('new props:', this.props);
        }

        render() {
            const {forwardedRef, ...rest} = this.props;

            // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
            return <Component ref={forwardedRef} {...rest} />;
        }
    }

    // 注意 React.forwardRef 回调的第二个参数 “ref”。
    // 将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”
    // 然后它就可以被挂载到被 LogProps 包裹的子组件上。
    return React.forwardRef((props, ref) => {
        return <LogProps {...props} forwardedRef={ref}/>;
    });
}
```

## 回调 Refs

- 内联函数定义的 ref 回调,更新执行两次，第一次传参为null,第二次才是DOM元素
    - 因为每次渲染会创建新的函数实例，定义为class 绑定函数的方式可避免

```js
function CustomTextInput(props) {
    return (
        <>
            <input ref={props.inputRef}/>
        </>
    );
}

class Parent extends React.Component {
    render() {
        return (
            // 传递一个函数，参数为DOM元素/组件实例
            <CustomTextInput
                inputRef={el => this.inputElement = el}
            />
        );
    }
}
```

# Fragments

- 一个组件返回多个元素,Fragments 将子列表分组,不用向 DOM 添加额外节点

```js
class Columns extends React.Component {
    render() {
        return (
            <dl>
                {props.items.map(item => (
                    // 没有`key`，React 会发出一个关键警告
                    <React.Fragment key={item.id}>
                        <dt>{item.term}</dt>
                        <dd>{item.description}</dd>
                    </React.Fragment>
                ))}
            </dl>

            // 没有map包含 可以短语法 <></> 不支持 key 或属性
        );
    }
}
```

# 高阶组件

- 基于 React 的组合特性而形成的设计模式
- 高阶组件是参数为组件，返回值为新组件的函数（组件转化为另一个组件）,纯函数
- HOC 不会修改传入的组件，也不会使用继承来复制其行为

## 不要改变原始组件，使用组合

```js
function logProps(InputComponent) {
    InputComponent.prototype.componentDidUpdate = function (prevProps) {
        console.log('Current props: ', this.props);
        console.log('Previous props: ', prevProps);
    };
    // 返回原始的 input 组件，暗示它已经被修改。
    return InputComponent;
}

// 每次调用 logProps 时，增强组件都会有 log 输出。
const EnhancedComponent = logProps(InputComponent);

// 修改后

function logProps(WrappedComponent) {
    return class extends React.Component {
        componentDidUpdate(prevProps) {
            console.log('Current props: ', this.props);
            console.log('Previous props: ', prevProps);
        }

        render() {
            // 将 input 组件包装在容器中，而不对其进行修改。Good!
            return <WrappedComponent {...this.props} />;
        }
    }
}

```

## 将不相关的 props 传递给被包裹的组件

- (HOC 应该透传与自身无关的 props)[https://react.docschina.org/docs/higher-order-components.html]

## 最大化可组合性

- 最常见的HOC

```js
// React Redux 的 `connect` 函数
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

```js
// connect 是一个函数，它的返回值为另外一个函数。
const enhance = connect(commentListSelector, commentListActions);
// 返回值为 HOC，它会返回已经连接 Redux store 的组件
const ConnectedComment = enhance(CommentList);

```

## 不要在 render 方法中使用 HOC

```js
render()
{
    // 每次调用 render 函数都会创建一个新的 EnhancedComponent
    // EnhancedComponent1 !== EnhancedComponent2
    const EnhancedComponent = enhance(MyComponent);
    // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！ 
    // 重新挂载组件会导致该组件及其所有子组件的状态丢失
    return <EnhancedComponent/>;
}
```

## 务必复制静态方法

- 新组件没有原始组件的任何静态方法

```js
// 在返回之前把这些方法拷贝到容器组件上
function enhance(WrappedComponent) {
    class Enhance extends React.Component {/*...*/
    }

    // 必须准确知道应该拷贝哪些方法 :(
    Enhance.staticMethod = WrappedComponent.staticMethod;
    return Enhance;
}

// hoist-non-react-statics自动拷贝所有非 React 静态方法
import hoistNonReactStatic from 'hoist-non-react-statics';

function enhance(WrappedComponent) {
    class Enhance extends React.Component {/*...*/
    }

    hoistNonReactStatic(Enhance, WrappedComponent);
    return Enhance;
}
```

# 与第三方库协同

- React 不会理会 React 自身之外的 DOM 操作
- 它根据内部虚拟 DOM 来决定是否需要更新，如果同一个 DOM 节点被另一个库操作了，React 会觉得困惑而且没有办法恢复
- 避免冲突的最简单方式就是防止 React 组件更新（渲染无需更新的 React 元素，比如一个空的 <div />）

# 性能优化

## 使用生产版本

## 虚拟化长列表

- 应用渲染长列表（上百甚至上千的数据）使用“虚拟滚动”
- 热门的虚拟滚动库：[react-window](https://react-window.vercel.app/#/examples/list/fixed-size)
  和 [react-virtualized](https://bvaughn.github.io/react-virtualized/)

## 避免调停

- 组件的`props`或`state`变更，React会将最新返回的元素与之前渲染的元素对比，决定是否有必要更新真实的 DOM。如不相同，React 会更新该 DOM
- React 只更新改变了的 DOM 节点，重新渲染仍然花费一些时间，如果很慢，可用`shouldComponentUpdate`提速,该方法会在重新渲染前被触发

```js
// 默认实现总是返回 true
// 知道在什么情况下组件不需要更新，可以在 shouldComponentUpdate 中返回 false 跳过整个渲染过程
shouldComponentUpdate(nextProps, nextState)
{
    return true;
}
```

- 大部分情况可以继承 `React.PureComponent` 以代替手写 shouldComponentUpdate()

## shouldComponentUpdate 的作用

```js
class CounterButton extends React.Component {
...

    // 当 props.color 或者 state.count 的值改变才需要更新
    shouldComponentUpdate(nextProps, nextState) {
        if (this.props.color !== nextProps.color) {
            return true;
        }
        if (this.state.count !== nextState.count) {
            return true;
        }
        return false;
    }

    render() {
    ...
    }
}
```

```js
// 这个代替上面，仅做对象的浅比较，无法检查深层差别
class CounterButton extends React.PureComponent {
...

    render() {
    ...
    }
}

// 仅是简单对比
// 问题：state中 words: ['marklar'] 变成 ['marklar','marklar'] 比较结果相同 并不会更新
```

- 避免上面问题：改变指针

```js
handleClick()
{
    this.setState(state => ({
        words: [...state.words, 'marklar'],
    }));
}
;
```

```js
function updateColorMap(colormap) {
    colormap.right = 'blue';
}

// 改为
function updateColorMap(colormap) {
    return Object.assign({}, colormap, {right: 'blue'});
}
```

# Portals

- 将子节点渲染到父组件以外的DOM节点

```js
ReactDOM.createPortal(child, container)
// child 任何可渲染的react子元素
// container DOM元素
```

```js
render()
{
    // React 挂载了一个新的 div，并且把子元素渲染其中
    return (
        <div>
            {this.props.children}
        </div>
    );
}
// 
render()
{
    // React 并没有创建一个新的 div。它只是把子元素渲染到 `domNode` 中。
    // `domNode` 是一个可以在任何位置的有效 DOM 节点。
    return ReactDOM.createPortal(
        this.props.children,
        domNode
    );
}
```

- 应用：父组件有 `overflow: hidden`或 `z-index` 需要在视觉上跳出容器
- portal 存在于 React 树， 且与 DOM 树 中的位置无关
- 从 portal 内部触发的事件会一直冒泡至包含 React 树的祖先，即便这些元素并不是 DOM 树 中的祖先

# Profiler

- 可以添加在 React 树中的任何地方测量树中这部分渲染所带来的开销
- 两个prop： id(组件)、组件更新被调用的回调函数

```js
render(
    <App>
        <Profiler id="Navigation" onRender={callback}>
            <Navigation {...props} />
        </Profiler>
        <Profiler id="Main" onRender={callback}>
            <Main {...props} />
        </Profiler>
    </App>
);

```

## 回调参数

```js
function onRenderCallback(
    id, // 监听的组件
    phase, // "mount" | "update" 来判断造成组件重渲染的原因（第一次、props、state、hooks引起）
    actualDuration, // 本次更新在渲染Profiler和子代上花费的时间，理想情况：子代因特定的prop改变而重渲染，值在第一次之后下降
    baseDuration, // 
    startTime, // 本次更新中 React 开始渲染的时间戳。
    commitTime, // 
    interactions // 本次更新交互的集合
) {
    // 合计或记录渲染时间。。。
}
```

## 没有使用ES6

- class 关键字定义组件，还可以使用 `create-react-class` 模块

```js
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
    render: function () {
        return <h1>Hello, {this.props.name}</h1>;
    }
});
```

- 函数组件和class组件，都有defaultProps属性

```js
class Greeting extends React.Component {
    // ...
}

Greeting.defaultProps = {
    name: 'Mary'
};

// 使用模块创建
var Greeting = createReactClass({
    getDefaultProps: function () {
        return {
            name: 'Mary'
        };
    },
    // ...
});
```

- 初始化 state

```js
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {count: props.initialCount};
    }

    // ...
}

// getInitialState方法
var Counter = createReactClass({
    getInitialState: function () {
        return {count: this.props.initialCount};
    },
    // ...
});
```

- createReactClass()创建组件，方法自动绑定到实例不需要在constructor中.bind(this)

## Mixins

- es6本身不包含任何 `mixin`  ，class组件不支持
- mixins（js对象：通过它封装通用的函数） 调用在组件之前
-

# 协调

## Diffing 算法

- `对比不同类型的元素`：根结点为不同类型元素时，React会拆卸原有的树建起新的树。根节点以下的组件也会被卸载，它们的状态会被销毁
- `对比同一类型的元素`：会保留DOM节点，仅对比更新有改变的属性
- `对比同类型的组件元素`：组件实例不变，更新组件实例的props 跟 最新的元素保持一致
- `对子节点进行递归`：递归DOM节点的子元素时，React会同时遍历两个子元素列表，产生差异时，生成一个mutation

```js
// 匹配完前两个最后插入第三个
<ul>
    <li>first</li>
    <li>second</li>
</ul>

<ul>
    <li>first</li>
    <li>second</li>
    <li>third</li>
</ul>
// 插入头部会很影响性能，开销变大
```

- `keys`：为了解决以上消耗性能，React使用key匹配原有树上的子元素与最新树上的子元素
    - 避免使用index，有顺序修改，diff变慢

# Render Props

- 用于告知组件需要渲染什么内容的函数prop

```js
// 追逐鼠标
... //Cat
const mouse = this.props.mouse;
return (
    <img src="/cat.jpg" style={{position: 'absolute', left: mouse.x, top: mouse.y}}/>
);
//  封装Mouse记录鼠标移动,
<Mouse render={mouse => (
    <Cat mouse={mouse}/>
)}/>
```

```js
// 如果你出于某种原因真的想要 HOC，那么你可以轻松实现
// 使用具有 render prop 的普通组件创建一个！
function withMouse(Component) {
    return class extends React.Component {
        render() {
            return (
                <Mouse render={mouse => (
                    <Component {...this.props} mouse={mouse}/>
                )}/>
            );
        }
    }
}
```

```js
// 不必使用render也可以
<Mouse children={mouse => (
    <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
)}/>
// 直接放到元素内部也可以
<Mouse>
    {mouse => (
        <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
    )}
</Mouse>
```

- `Render Props`与`React.PureComponent` 避免一起使用，每一个 render 对于render prop总会生成新值，浅比较props的时候总会是false

```js
class MouseTracker extends React.Component {
    // 定义为实例方法，`this.renderTheCat`
    // 在渲染中使用它时，它指的是相同的函数
    renderTheCat(mouse) {
        return <Cat mouse={mouse}/>;
    }

    render() {
        return (
            <div>
                <Mouse render={this.renderTheCat}/>
            </div>
        );
    }
}
```

# 静态类型检查

- 运行前识别类型问题，使用 `Flow` 或 `TypeScript` 来代替 `PropTypes`

## Flow

- 通过类型注释的特殊语法 扩展js，浏览器解析不了这种语法
- 编译后的代码去除 Flow 语法

## TypeScript

- 在`tsconfig.json`中定义配置项
- .tsx 包含 JSX 代码的 ts文件

## 类型定义

- 判断一个库是否包含类型，`index.d.ts` 或者 package.json 文件的 `typings` 或 `types` 属性中指定类型文件
- `DefinitelyTyped` 为没有声明文件的js库提供类型定义

```js
// yarn
yarn
add--
dev
@types/
react
```

- 局部声明：使用的包里没有声明文件，在 DefinitelyTyped 上也没有，创建本地定义文件，根目录declarations.d.ts 文件

```js
declare
module
'querystring'
{
    export function stringify(val: object): string

    export function parse(val: string): object
}
```

# 严格模式

- `StrictMode` 显示应用程序中潜在问题的工具，不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告

```js
<React.StrictMode>
    <div>
        <ComponentOne/>
        <ComponentTwo/>
    </div>
</React.StrictMode>
```

## 识别不安全的生命周期

## 使用过时字符串 ref API 的警告

- createRef方式会警告，回调 ref 依旧适用

## 使用废弃的 findDOMNode 方法的警告

- 只读一次的 API：调用只会返回第一次查询的结果。子组件渲染不同的节点，无法跟踪更改
- findDOMNode使父组件需要单独渲染子组件，产生重构；
- 仅在组件返回单个且不可变的 DOM节点才有效
- ref传递、转发

## 检测意外的副作用

- React 工作的两个阶段
    - 渲染：确定进行的更改（新旧DOM树对比）
    - 渲染阶段的声明周期可能会被多次调用
        - constructor
        - componentWillMount (or UNSAFE_componentWillMount)
        - componentWillReceiveProps (or UNSAFE_componentWillReceiveProps)
        - componentWillUpdate (or UNSAFE_componentWillUpdate)
        - getDerivedStateFromProps
        - shouldComponentUpdate
        - render
        - setState 更新函数（第一个参数）
    - 提交：React 应用变化时（React DOM插入、更新、删除节点）调用生命周期方法

## 检测过时的 context API

# 使用 PropTypes 进行类型检查

- 限制单个元素

```js
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
    render() {
        // 这必须只有一个元素，否则控制台会打印警告。
        const children = this.props.children;
        return (
            <div>
                {children}
            </div>
        );
    }
}

MyComponent.propTypes = {
    children: PropTypes.element.isRequired
};
```

- 默认 Prop 值
    - 类型检查适用 defaultProps，发生在它赋值后

```js
class Greeting extends React.Component {
    static defaultProps = {
        name: 'stranger'
    }

    render() {
        return (
            <div>Hello, {this.props.name}</div>
        )
    }
}

// 或者
Greeting.defaultProps = {
    name: 'Stranger'
};
```

# 非受控组件

- 受控组件：表单数据React组件管理
- 非受控组件：表单数据由DOM节点处理（ref操作）
- 默认值

```js
 <input
    defaultValue="Bob"
    type="text"
    ref={this.input}/>
```

- `<input type="file"/>` 始终是一个非受控组件

# Web Components

- 为可复用组件提供了强大的封装，而 React 提供了声明式的解决方案，使 DOM 与数据保持同步

```js
class XSearch extends HTMLElement {
    connectedCallback() {
        const mountPoint = document.createElement('span');
        this.attachShadow({mode: 'open'}).appendChild(mountPoint);

        const name = this.getAttribute('name');
        const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
        ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
    }
}

// 注册自定义元素
customElements.define('x-search', XSearch);
```
