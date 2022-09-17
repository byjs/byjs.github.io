---
title: JavaScript - 继承

date: 2019-06-05

tag: JavaScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/f02edc75f539318a6b642f6cb91f33f49c0074ed.jpg
---

# 继承

## 原型链

- 让一个引用类型继承另一个引用类型的属性和方法
- 由 `Prototype` 不断把实例和原型对象联系起来的结构

- 过程
    - 原型对象通过 `constructor` 属性指向构造函数
    - 实例通过 `Prototype` 属性指向原型对象
- 缺陷
    - 在赋值时，赋给变量的是它在内存中的地址
    - 实例没办法向构造函数传参

## 继承方式

### 借用构造函数继承

- 子类型的构造函数中借用父类型的构造函数（为了解决引用类型）

```js

function A() {
    this.name = "a"
    this.color = ['red', 'green']
}

function B() {
    // "借用"就在这里体现
    A.call(this)
}

var b1 = new B()
var b2 = new B()
console.log(b1.name, b2.name) // a a
console.log(b1.color, b2.color) // ['red','green'] ['red','green']
// 改变b1
b1.name = 'b'
b1.color.push('black')
console.log(b1.name, b2.name) // b a


```

### 组合继承

- 属性独立，方法复用 最常用！
- 原型链负责原型对象上的方法，call借用构造函数负责让子类型拥有各自的属性
- 缺点：子类型会调用两次父类型的构造函数

```js
function A(name) {
    this.name = name
    this.color = ['red', 'green']
}

A.prototype.sayA = function () {
    console.log('form A')
}

function B(name, age) {
    A.call(this, name)
    this.age = age
}

B.prototype = new A()
B.prototype.sayB = function () {
    console.log('form B')
}
var b1 = new B('hong', 12)
var b2 = new B('lv', 13)
console.log(b1, b2)
```

### 原型式继承

- Object.create()，对对象的浅复制，父子对象的指针指向同一个引用类型

```js
var A = {
    name: 'A',
    color: ['red', 'green']
}
var B = Object.create(A)
B.name = 'B'
B.color.push('black')

var C = Object.create(A)
C.name = "C"
B.color.push('blue')
console.log(A.name, B.name, C.name) // A B C
console.log(A.color) // ['red', 'green', 'black', 'blue']

```

### 寄生式继承(工厂模式)

```js
function createA(name) {
    var obj = Object(name)
    obj.say0 = function () {
        console.log('from 0')
    }
    return obj
}

var A = {
    name: 'A',
    color: ['red', 'green', 'blue']
}
// 实现继承
var B = createA(A)
console.log(B) // {name: 'A', color: Array(3), say0: ƒ}
```

### 寄生组合式继承

- 换一种方式实现 `B.prototype = new A()`
- 避免两次调用父类的构造函数

```js
// 默认A是父类型，B是子类型
function inheritPrototype(B, A) {
    // 之前 B.prototype = new A() 也是为了得到 A的prototype 的复制品 实现原型链
    // 复制一个A的原型对象，把这个原型对象替换掉B的原型对象
    var pro = Object(A.prototype)
    pro.constructor = B
    B.prototype = pro
    // 此时B的原型对象上不会有A的属性，因为它不是A的实例了
}
```

### ES6 `extends`

- 属性放在构造函数里，方法绑定在原型对象上



