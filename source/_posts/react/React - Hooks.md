---
title: React - Hooks

date: 2019-05-03

tag: React

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/2.jpeg
---

# Hook

- 一些可以在函数组件中使用React state和生命周期等特性的函数
- 不能在 class 组件中使用

## State Hook

- useState
    - 类似class组件中的 this.setState，但它不会合并 新旧state
    - 在函数调用时保存变量。state中的变量会被React保留
    - 唯一的参数就是初始state
    - 返回值：当前state和更新state的函数

```js
// 引入Hook
import React, {useState} from 'react';

function Example() {
    // 声明
    const [count, setCount] = useState(0);
    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Click me
            </button>
        </div>
    );
}
```

```js
// 函数式 setState 结合展开运算符 来合并更新对象
setState(prevState => {
    // 也可以使用 Object.assign
    return {...prevState, ...updatedValues};
});
// useReducer 是另一种方案，它更适合用于管理包含多个子值的 state 对象
```

- 方括号用途

```js
 const [fruit, setFruit] = useState('banana');
// 等价于
var fruitStateVariable = useState('banana'); // 返回一个有两个元素的数组
var fruit = fruitStateVariable[0]; // 数组里的第一个值
var setFruit = fruitStateVariable[1]; // 数组里的第二个值
```

## Effect Hook

- React组件中副作用： 是否需要清除

- 无需清除的effect
    - 在更新DOM之后运行额外代码（网络请求、手动改变DOM、记录日志）
    - 在class组件中：副作用操作放到 `componentDidMount` 和 `componentDidUpdate` 函数中
    - ⚠️ 与以上俩API不同，useEffect

- useEffect
    - 类似：class组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 合成
    - 通过它可以告诉React组件需要在渲染后执行的操作
    - 每次渲染都会执行，在执行下一个 effect 之前，上一个effect就已经被清除

- 需要清除的effect（订阅外部数据源）
    - 返回函数的清除机制。添加移除放一起
    - React在组件卸载时执行清除操作

```js
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // 销毁时执行
    return () => {
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
});
```

- 执行时机
    - 会在浏览器会之后延迟执行，但会保证在新的渲染前执行
- 条件执行
    - 执行只运行一次的 effect（仅在组件挂载和卸载时执行），第二个参数为[]，代表effect不依赖 props或state中的任何值

```js
useEffect(() => {
    const subscription = props.source.subscribe();
    return () => {
        subscription.unsubscribe();
    }
}, [props.source]);
```

## 自定义Hook

- 组件重用状态逻辑解决方案：`高阶组件`、 `render props`、`自定义Hook`
- Hook是一种复用状态逻辑的方式，它不复用 state 本身
- Hook的每次调用都有一个完全独立的state

## 提取自定义Hook

- 名称以 "use" 开头，函数内部可以调用其他Hook
- 多个组件使用相同的hook不会共享state

```js
function useFriendStatus(friendID) {
    const [isOnline, setIsOnline] = useState(null);
    // ...
    return isOnline;
}
```

## 其他Hook

- useContext：不使用组件嵌套就可以订阅 React的Context
    - 调用了useContext的组件总会在 context 值变化时重新渲染
        - 只是能够`读取 context 的值` 和`订阅 context的变化`，仍需要在上层组件中使用 `<MyContext.Provider>` 为下层组件提供 context

```js
// 参数必须是 context 对象本身
const value = useContext(MyContext);

// 相当于class组件中的 static contextType = MyContext 或 <MyContext.Consumer>

```

```js
const themes = {
    light: {
        foreground: "#000000",
        background: "#eeeeee"
    },
    dark: {
        foreground: "#ffffff",
        background: "#222222"
    }
};

const ThemeContext = React.createContext(themes.light);

function App() {
    return (
        <ThemeContext.Provider value={themes.dark}>
            <Toolbar/>
        </ThemeContext.Provider>
    );
}

function Toolbar(props) {
    return (
        <div>
            <ThemedButton/>
        </div>
    );
}

function ThemedButton() {
    const theme = useContext(ThemeContext);
    return (
        <button style={{background: theme.background, color: theme.foreground}}>
            I am styled by theme context!
        </button>
    );
}
```

- useReducer：管理组件本地的复杂state
    - useState 的替代方案
    - 场景：state逻辑较复杂并且包含多个子值或 下个state 依赖于之前的state
    - 给触发深更新的组件做性能优化：可以向子组件传递 dispatch 而不是回调函数
    - 返回值与当前 state相同，React 将跳过子组件的渲染和副作用的执行

```js
function Todos() {
    const [state, dispatch] = useReducer(reducer, initialArg, init);

// ...

```

- useCallback

```js
const memoizedCallback = useCallback(() => {
    doSomething(a, b);
}, [a, b]);

// useCallback(fn, deps) 相当于 useMemo(() => fn, deps)
```

- useMemo
    - 传入的函数会在渲染期间执行，不要在此函数内执行与渲染无关的操作
    - 如果没有依赖项数组，useMemo 在每次渲染时都会计算新值

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- useRef
    - 返回一个可变的 ref对象，它在组件的整个生命周期内保持不变
    - useRef() 和自建一个 {current: ...} 对象的唯一区别是，useRef 会在每次渲染时返回同一个 ref 对象
    - 在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref 来实现

```js

const refContainer = useRef(initialValue);
```

- useImperativeHandle
    - 在使用 ref 时自定义暴露给父组件的实例值。应当与 forwardRef一起使用

```js
useImperativeHandle(ref, createHandle, [deps])
```

```js
function FancyInput(props, ref) {
    const inputRef = useRef();
    useImperativeHandle(ref, () => ({
        focus: () => {
            inputRef.current.focus();
        }
    }));
    return <input ref={inputRef} ...
    />;
}

FancyInput = forwardRef(FancyInput);

// 渲染 <FancyInput ref={inputRef} /> 的父组件可以调用 inputRef.current.focus()
```

- useLayoutEffect
    - 会在所有的 DOM 变更之后同步调用 effect
    - useLayoutEffect 与 componentDidMount、componentDidUpdate 的调用阶段是一样的
    - 使用服务端渲染，useLayoutEffect 和 useEffect 都无法在 js 代码加载完之前执行
    - 服务端渲染中排除依赖布局 effect 的组件，可通过` showChild && <Child /> `进行条件渲染,使用 useEffect(() => { setShowChild(true); }, [])
      延迟展示组件来避免UI错乱

- useDebugValue
    - 用于在 React 开发者工具中显示自定义 hook 的标签
    - 第二个可选参数为格式化函数，只有在Hook被检查时才会被调用

```js
function useFriendStatus(friendID) {
    const [isOnline, setIsOnline] = useState(null);
    // ...
    // 在开发者工具中的这个 Hook 旁边显示标签
    // e.g. "FriendStatus: Online"
    useDebugValue(isOnline ? 'Online' : 'Offline');
    return isOnline;
}
```

```js
useDebugValue(date, date => date.toDateString());
```

# Hook 规则

## 只在最顶层使用Hook

- 只能在 `函数最外层` 调用，不要在循环、条件判断、子函数中调用
- 只能在`React函数组件``自定义Hook`中调用

- ESLint插件： eslint-plugin-react-hooks

```js
// ESLint 配置
{
    "plugins"
:
    [
        // ...
        "react-hooks"
    ],
        "rules"
:
    {
        // ...
        "react-hooks/rules-of-hooks"
    :
        "error", // 检查 Hook 的规则
            "react-hooks/exhaustive-deps"
    :
        "warn" // 检查 effect 的依赖
    }
}
```
