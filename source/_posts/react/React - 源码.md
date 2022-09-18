---
title: React -  源码
date: 2022-08-13
tag: React
categories:
- React
cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/3.jpeg
---

# react 源码架构

- 源码目录结构

    - fixtures：为代码贡献者提供的测试React
    - packages：主要部分，包含Scheduler（调度器：排序优先级，优先级高的任务先进性处理）,reconciler（协调器：找出哪些节点发生了变化，打上不同的tag）等
    - scripts：react构建相关


- packages主要包含模块
    - react：核心Api（React.createElement、React.Component）
    - render相关的：react-art（canvas、svg的渲染）、react-dom（浏览器环境）、react-native-renderer（原生相关）、react-noop-renderer（调试或fiber用）
    - 试验型的包：react-server（ssr相关）、react-fetch（请求相关）、react-interactions（事件相关）、react-reconciler（构建节点）
    - shared：公共方法和变量
    - 辅助包：react-is（判断类型）、react-client（流相关）、react-fetch（数据请求）、react-refresh（热加载）、scheduler（调度器）、react-reconciler（构建节点）

# jsx & 核心Api

- virtual Dom

- 用js对象表示dom信息和结构，用类似react-dom等模块与真实dom同步，这个过程为协调（reconciler），这种方式可以声明式的渲染相应的ui状态，在react中以fiber树的形式存放组件树的相关信息，在更新时可以增量渲染相关dom，所以fiber也是virtual Dom实现的一部分

- 大量的dom操作慢，很小的更新都可能引起页面的重新排列，js对象优于内存中，处理更快，可以通过diff算法比较新老virtual Dom的差异，批量、异步、最小化的执行dom的变更，提高性能

- 可以跨平台，jsx-->ReactElement对象-->真实节点

- jsx文件要声明 `import React from 'react'` （react17 之后不用导入）是因为
- jsx是`ClassComponent的render函数`或者`FunctionComponent`的返回值，表示组件内容，经过babel编译后，最后被编译成React.createElement

- `React.createElement`的源码做了几件事
    - 处理config,剩余的属性将添加到props
    - 处理children
    - 处理defaultProps
    - 调用ReactElement返回jsx对象（virtual-dom）

```js
// ReactElement.js
export function createElement(type, config, children) {
    let propName;
    const props = {}
    let key = null;
    let ref = null;
    let self = null;
    let source = null;

    if (config != null) {
        //处理config，把除了保留属性外的其他config赋值给props
        //...
    }

    const childrenLength = arguments.length - 2;
    //把children处理后赋值给props.children
    //...

    //处理defaultProps
    //...

    return ReactElement(
        type,
        key,
        ref,
        self,
        source,
        ReactCurrentOwner.current,
        props,
    );
}

const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    $$typeof: REACT_ELEMENT_TYPE, // 表示是ReactElement类型
    type: type, // class或function
    key: key, // key
    ref: ref, // ref属性
    props: props, // props
    _owner: owner,
  };

  return element;
};
```

- 组件的类型：$$typeof，源码中有检查是否是合法Element的函数
  - `ClassComponent` type是class本身，`FunctionCoponent`type是这个function，创建的组件首字母必须大写，否则被当成普通节点，type是字符串
```js
// ReactElement.js
export function isValidElement(object){
    return (
        typeof object === 'object' && object !== null && object.$$typeof === REACT_ELEMENT_TYPE
    )
}

// const REACT_ELEMENT_TYPE = Symbol.for('react.element');
```
- component

```js
// ReactBaseClasses.js
function Component(props,context,updater){
    this.props = props; // props属性
    this.context = context; // 当前context
    this.refs = emptyObjext; // ref挂载的对象
    this.updater = updater || ReactNoopUpdateQueue; // 更新的对象
}
Component.prototype.isReactComponent = {}; // 表示是classComponent

// 和Component类似，进行原型继承，赋值isPureReactComponent
function PureComponent(props,context,updater){
    // ...同Component
}

const pureComponentPrototype  = (PureComponent.prototype = new ComponentDummy())
pureComponentPrototype.constructor = PureComponent;
Object.assign(pureComponentPrototype,Component.prototype)
pureComponentPrototype.isPureReactComponent = true

```

# 模式入口函数

- react有3中进入主体函数的入口
  - legacy：`ReactDOM.render(<APP />,rootNode)`
  - blocking(实验)：`ReactDOM.createBlockingRoot(rootNode).render(<APP />)`
  - concurrent：`ReactDOM.createRoot(rootNode).render(<APP />)`

- 不同
  - legacy常用，构建dom的过程是同步的，在render的`reconciler`中，如果diff过程很耗时，导致js一直阻塞高优先级别的任务(点击事件)，页面会卡顿无法响应
  - react未来模式：`concurrent Mode`，用时间片调度实现了异步可中断任务，根据设备性能不同，时间片长度也不一样，在每个时间片中，任务到了过期时间，主动让出线程给高优先级的任务（scheduler&lane模型）
  
- 主要执行流程
  - `createRootImpl`创建`fiberRootNode、rootFiber`
  - `updateContainer`创建Update对象，保存在`updateQueue环状链表`
  - `scheduleUpdateOnFiber`在Fiber上调度update,`ensureRootIsScheduled`调度根节点
  - `performSyncWorkOnRoot | performConcurrentWorkOnRoot(render阶段)`
  - commitRoot(commit阶段)

## legacy

- render -> legacyRenderSubtreeIntoContainer -> createRootImpl -> createFiberRoot -> createHostRootFiber -> legacyRenderSubtreeIntoContainer(调用updateContainer) —>  -> scheduleUpdateOnFiber(调用performSYncWorkOnRoot)进入render 和commit阶段
  
```js
function legacyRenderSubtreeIntoContainer(parentComponent, children, container, forceHydrate, callback){
    var root = container._reactRootContainer;
    var fiberRoot;
    
    if(!root){
        // mount
        root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate); // 创建root节点
        fiberRoot = root._internalRoot;
        
        if(typeof callback === 'function'){
            var originalCallback = callback;
            
            callback = function(){
                var instance = getPublicRootInstance(fiberRoot);
                originalCallback.call(instance);
            }
        }
        unbatchedUpdates(function (){
            updateContainer(children, fiberRoot, parentComponent, callback); // 创建update入口
        })
    }else{
        // update
      
      fiberRoot = root._internalRoot;
      if(typeof callback === 'function'){
          var _originalCallback = callback;
          
          callback = function(){
              var instance = getPublicRootInstance(fiberRoot);
            _originalCallback.call(instance)
          }
      }
      updateContainer(children,fiberRoot,parentComponent,callback)
    }
}
```

```js

function createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks){
    var root = new FiberRootNode(containerInfo, tag, hydrate); // 创建fiberRootNode
    const uninitializedFiber = createHostRootFiber(tag); // 创建rootFiber
  // rootFiber和fiberRootNode连接
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  // 创建updateQueue
  initialzeUpdateQueue(uninitializedFiber)
  return root
}

function initializeUpdateQueue<state>(fiber:Fiber):void{
    const queue: UpdateQueue<state> ={
        baseState: fiber.memoizedState, // 初试state, 之后基于它根据Update计算新的state
        firstBaseUpdate: null, //Update链表头
        lastBaseUpdate: null, //Update链表尾
      // 新产生的update以单向环状链表保存在shared.padding上，计算state时剪开这个环状链表 lastBaseUpdate后
        shared:{
            padding:null
        },
      effects: null
    };
    fiber.updateQueue = queue
}
```

```js
function updateContainer(element, container,parentComponent, callback){
  var lane = requestUpdateLane(current$1) //获取当前可用lane
  var update = createUpdate(eventTime, lane); // 创建update
  
  update.payload = {
      element: element // jsx
  }
  enqueueUpdate(current$1, update); 
  scheduleUpdateOnFiber(current$1,lane,eventTime) // 调度update
}

function scheduleUpdateOnFiber(fiber, lane,eventTime){
    if(lane === SyncLane){ // 同步lane 对应legacy模式
        
        // ...
      performSyncWorkOnRoot(root) // render阶段的起点
    }else{ // concurrent模式
        ensureRootIsScheduled(root, eventTime) // 确保root被调度
    }
}
```

## concurrent

```js
function ensureRootIsScheduled(root,currentTime){
    var nextLanes = getNextLanes(root, root === workInProgressRoot?workInProgressRootRenderLanes : NoLanes) // 计算nextLanes
  // 将lane的优先级转换成schduler的优先级
  var schedulerPriorityLevel = lanePriorityToSchedulerPriority(newCallbackPriority)
  // 以schedulerPriorityLevel的优先级执行performConcurrentWorkOnRoot 也就是concurrent模式的起点
  newCallbackNode = scheduleCallback)(schedulerPriorityLevel,performConcurrentWorkOnRoot.bind(null,root))
  
}
```

- 不同点
  - 传参不同：createRootImpl第二个参数，一个是LegacyRoot 一个是ConcurrentRoot
  - requestUpdateLane 获取的lane的优先级不同
  - 函数scheduleUpdateOnFiber中根据不同优先级进入不同分支：legacy模式进入performSyncWorkOnRoot, concurrent模式会异步调度performConcurrentWorkOnRoot

# Fiber

- react15在render阶段的reconcile是不可打断的，进行大量节点的reconcile可能卡顿，因为js是单线程，浏览器所有时间都交给js执行。所以react16之后有了scheduler进行时间片的调度，给每个工作单元一定的时间，如果在这个时间内没执行完，也要交出执行权给浏览器进行绘制和重排。所以异步可中断的更新需要在内存中来保存工作的信息，这个数据结构就是Fiber

- Fiber做的事情
  - `工作单元 任务拆解`：保存原生节点｜组件节点对应信息(包括优先级)，这些节点通过指针形成Fiber树
  - `增量渲染`：jsx对象和current Fiber对比，生成最小的差异补丁，应用到真实节点
  - `根据优先级暂停、继续、排列优先级`：保存节点优先级，通过不同节点优先级对比，达到任务暂停、继续、排列优先级等，也为上层实现批量更新、Suspense提供了基础
  - `保存状态`：因为Fiber能保存状态和更新信息，所以就能实现函数组建的状态更新，就是hooks

- fiberRoot：整个应用的根节点，只有一个
- rootFiber：ReactDOM.render｜ReactDOM.unstable_createRoot创建的应用节点，可以存在多个

```js
// ReactFiber.old.js
function FiberNode(
    tag: WorkTag,
    peddingProps: mixed,
    key: null | string,
    mode: TypeOfMode
){
  // 作为静态数据结构，保存节点信息
  this.tag = tag; // 对应组件类型
  this.key = key; // key属性
  this.elementType = null; // 元素类型
  this.type = null; // func｜class
  this.stateNode = null; // 真实dom节点 
  
  // 作为fiber树架构，连接成fiber树
  this.return = null; // 指向副节点
  this.child = null; // 指向child
  this.sibling = null; // 指向兄弟节点
  this.index = 0
  
  this.ref = null
  // 用作为工作单元 计算state
  this.pendProps = peddingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;
  
  this.mode = mode;
  
  // effect相关
  this.effectTag = NoEffect;
  this.nextEffect = null;
  this.firstEffect = null;
  this.lastEffect = null;
  
  // 优先级相关属性
  this.lanes = NoLanes;
  this.childLanes = NoLanes;
  
  // current 和 workInProgress的指针
  this.alternate = null;
}
```

## 双缓存

- Fiber双缓存：经过reconcile(diff)形成了新的`workInProgress Fiber`，将`workInProgress Fiber`切换成`current Fiber`应用到真实dom中
- 好处：内存中形成视图的描述，最后应用到dom，减少对dom的操作

- 真实dom对应在内存中的Fiber节点会形成Fiber树，在react中叫`current Fiber`，就是当前dom树对应的Fiber树，正在构建Fiber树叫 `workInProgress Fiber`，两棵树的节点通过 `alternate` 相连

- 构建 `workInProgress Fiber` 发生在 `createWorkInProgress` 中，它能创建｜复用Fiber
  - mount：创建fiberRoot和rootFiber -> 根据jsx创建`workInProgress Fiber` -> `workInProgress Fiber`切换成`current Fiber`
  - update：根据current Fiber创建 workInProgress Fiber树 -> `workInProgress Fiber`切换成`current Fiber`
 

# render

- render阶段主要：构建Fiber树和生成effectList
- 






























