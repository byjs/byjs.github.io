---
title: Vue - 核心重点

date: 2017-07-18

tag: Vue

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/44db917e1cf70c5dff9e4a34d2ca6e96a7cb0ca8.jpg
---

# Vue

## 计算属性

- 相比method有缓存
- setter

```js
set(newValue)
{
    var names = newValue.split(' ')
    this.firstName = names[0]
    this.lastName = names[names.length - 1]
}
// 现在再运行 vm.fullName = 'John Doe' 时，setter 会被调用
```

## watch

```js
watch: {
    a(val, oldVal)
    {
        console.log('new: %s, old: %s', val, oldVal)
    }

    // 方法名
    b: 'someMethod'
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c:{
        handler(val, oldVal)
        { /* ... */
        }
        deep: true
    }
    // 该回调将会在侦听开始之后被立即调用
    d: {
        handler: 'someMethod'
        immediate:true
    }
    e: [
        'handle1',
        function handle2(val, oldVal) { /* ... */
        },
        {
            handler: function handle3(val, oldVal) { /* ... */
            }
            /* ... */
        }
    ]
// watch vm.e.f's value: {g: 5}
//     e.f: function (val, oldVal) { /* ... */
//     }
}

```

## 列表渲染

```js
// v-for 也可以接受整数。在这种情况下，它会把模板重复对应次数。
<span v-for="n in 10">{{n}} </span>
```

## 数组更新检测

```js
// Vue对被侦听的数组方法进行了包装，让push() pop()能响应变化
arr[1] = 123;
arr.length = 2;
// 这种方式无法响应

// 解决方法：
Vue.set(vm.items, indexOfItem, newValue);
items.splice(newLength)
```

## 事件处理

内联语句

```js
// 访问原始的 DOM 事件。可以用特殊变量 $event 把它传入方法
<button v:on-click="warn('hello', $event)"></button>
```

## 修饰符

### stop

```vue
<a v-on:click.stop="doThis"></a>
```

### prevent

```vue
// 提交事件不再重载页面
<form v-on:submit.prevent="onSubmit"></form>

// 只有修饰符
<form v-on:submit.prevent></form>
```

### capture

```vue
// 捕获，即元素自身触发的事件先在此处理，然后才交由内部元素进行处理
<div v-on:click.capture="doThis">...</div>
```

### self

只当在 event.target 是当前元素自身时触发处理函数

### 修饰符的顺序：相应的代码会以同样的顺序产生。

因此，用 `@click.prevent.self` 会阻止所有的点击， // 而 `@click.self.prevent` 只会阻止对元素自身的点击。

- once: 只触发一次 passive

按键修饰符

- .tab
- .delete
- .esc
- .space
- .up
- .down
- .left
- .right

## 系统修饰键

```vue
<input @keyup.alt.67="clear">
// ctrl alt shift meta (Mac对应command键)
<div @click.ctrl="doSomething">Do something</div>
```

## .exact

```vue
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

- `.lazy`

```vue
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg">
```

- `.number` 自动将用户的输入值转为数值类型
- `.trim` 自动过滤用户输入的首尾空白字符

## 动态组件

```vue
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component :is="currentTabComponent"></component>

<!--  使用keep-alive 来缓存组件状态 -->
<keep-alive>
<component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

## 异步组件

```js
new Vue({
    components: {
        'my-component': () => import('./my-async-component')
    }
})
```

## 避免HTML限制标签内需要特定标签

<table>
    <tr is="blog-post-row"></tr>
</table>

```js

<script type="text/x-template"></script>
```

## 基础组件自动全局注册

```js
const requireComponent = require.context(
    './components', true, /[\w-]+\.vue$/
)

export default {
    install() {
        requireComponent.keys().forEach((fileName) => {
            const componentConfig = requireComponent(
                fileName).default;
            Vue.component(componentConfig.name, componentConfig);
        })
    }
}
```

## Prop

传入一个对象的所有属性

<blog-post v-bind="post"></blog-post>

<blog-post  :id="post.id"  :title="post.title"></blog-post>

## 自定义检查函数

```js
validator: function (value) { // 这个值必须匹配下列字符串中的一个 return ['success', 'warning', 'danger'].indexOf(value) !== -1 }

    type可以是自定义的构造函数

    function Person(firstName, lastName) {
        this.firstName = firstName
        this.lastName = lastName
    } // 你可以使用：props: { author: Person }    
}
// 验证 author prop 的值是否是通过 new Person 创建的。
// 组件上写的attr最后都会添加到组件的根元素上

Vue.component('my-component', {inheritAttrs: false,})
// 但不会影响style和class的绑定
```

## 自定义事件

将原生事件绑定到组件 (.native)

```vue
// 因为在组件上@绑定事件，语义是@某自定义事件，命名空间都是自定义事件， // 不与原生事件产生混合，所以要在组件的根元素上绑定浏览器点击事件，就必须加.native

// 如果一个组件里没$emit('click')事件，调用这个组件时
<Menu @click="handler()"></Menu> 是无效的，因为组件内没触发名为click的自定义事件。

<base-input @focus.native="onFocus"></base-input>
```

### .sync

```vue

<text-document :title.sync="doc.title"></text-document>

// 等价于
<text-document
    :title="doc.title"
    @update:title="doc.title = $event"
></text-document>
```

### slot

```vue

<template v-slot:[dynamicSlotName]>

  <current-user v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </current-user>
```

## 缩写

```vue

<template v-slot:[dynamicSlotName]></template>

<current-user v-slot:default="slotProps">
{{ slotProps.user.firstName }}
</current-user>

<current-user #default="{ user }">
{{ user.firstName }}
</current-user>
```

## 依赖注入

```js
provide()
{
    return {
        getMap: this.getMap
    }
}

inject: ['getMap']
```

## 程序化的事件侦听器

```js
var picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
});

this.$once('hook:beforeDestroy', function () {
    picker.destroy();
})
```

## 循环引用

### 递归组件

组件内自己调用自己 组件是可以在它们自己的模板中调用自身的。不过它们只能通过 name 选项来做这件事

```js
// 会自动把第一个参数作为组件的name Vue.component('unique-name-of-my-component', {   
// ... })
```

### 组件之间的循环引用

互相引用无法使用import导入

```js
// 但可以
components: {
    TreeFolderContents:   () => import('./tree-folder-contents.vue')
}
```

Vue.component 全局注册组件的时候，这个悖论会被自动解开

## 模板定义的替代品

### 内联模板(inline-template)

```vue

<menu-bar inline-template>
这里是你定义的模板，会用这个模板替代组件原先的模板
</menu-bar>
```

### 关联模板(X-Template)

```js
// 仅仅只是把模板定义在别的地方，是不好的用法。
<script type="text/x-template" id="hello-world-template">
    <p>Hello hello hello</p>
</script>

<script>
    Vue.component('hello-world', {
    template: '#hello-world-template'
})
</script>
```

## 控制更新

### 强制更新

```vue
// 迫使 Vue 实例重新渲染。注意它仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。 vm.$forceUpdate()

// 使模板只更新一次
<p v-once>
{{ text }}
</p>
```