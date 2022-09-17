---
title: TypeScript

date: 2019-09-07

tag: TypeScript

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/ae582e17cef4bb1881c2c9983d5f93e116bd7094.jpg
---

# 基础

## 基本类型

```ts
const num: number = 1;
const str: string = 'str'
const bool: boolean = true

// `null` 和 `undefined` 是所有类型的子类型，不指定 `--strictNullChecks`标记 情况下，null和undefined可以赋值给number类型变量
const nulls: null = null
const undefine: undefined = undefined

const symbols: symbol = Symbol('symbal')

const any: any = 'any types'  // 不清楚类型的变量（如：用户输入、第三方库），不希望类型检查
let unusable: void = undefined; // void类型的变量只能赋予 `undefined` 和 `null` (函数没有返回值时可用)

```

## Never

- 表示永不存在值的类型（抛异常、根本不会有返回值的函数、箭头函数的返回值）
- 任何类型的子类型，可以赋值给任何类型。除never本身，其他类型不能赋值给never类型

## 数组

```ts
// 两种方式定义
const nums: number[] = [1, 2, 3, 4]

let array: Array<number> = [1, 2, 3, 4]

// 长度+ 填充
let arr: Array<number> = new Array<number>(26).fill(0);

const dates: Date[] = [new Date(), new Date()]
```

## 联合类型

- 取值可以为其中一种

```ts
function getString(something: string | number): string {
    // 不确定类型时，只能访问类型共有的属性或方法（.toString()）访问length会报错不是公共的
    return something.toString()
}
```

- 值的类型是 A | B，确定的是它包含了两者的共有成员

```ts
interface Bird {
    fly();

    layEggs();
}

interface Fish {
    swim();

    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

- 变量被赋值时，会根据类型推论的规则推断出一个类型

```ts
  let myFavNumber: string | number
myFavNumber = 'seven' // 推断为 string
console.log(myFavNumber.length) // 5
myFavNumber = 7 // 编译报错
```

- 类型守卫

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

if (isFish(pet)) {
    pet.swim();  // 到这里，这个变量的类型进一步缩小 从 a| b 变成a，pet的类型等于fish
} else {
    pet.fly();
}

```

- in 操作符：`n in x`,x 为 联合类型

```ts
type attr = 'name' | 'mobile'
type attr2 = 'email' | 'avatar'
type person = {
    [key in attr]: string
[key in attr2]
:
string
}

// type person = {
//     name: string;
//     mobile: string
// }

type Fish = {
    swim: string
}
let fish: Fish = {
    swim: "sss"
}
if ("swims" in fish) {
    console.log("yes")
} else {
    console.log("no")
}

```

- typeof 类型守卫

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

- 索引类型查询操作符（typeof）

```ts
interface Car {
    manufacturer: string;
    model: string;
    year: number;
}

let carProps: keyof Car; // ('manufacturer' | 'model' | 'year')
```

- 索引访问操作符（T[k]）

```ts
// o: T propertyName: K  意味着  o[propertyName]: T[K]
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
    return o[propertyName]; // o[propertyName] is of type T[K]
}

let name: string = getProperty(taxi, 'manufacturer');
let year: number = getProperty(taxi, 'year');

// error, 'unknown' is not in 'manufacturer' | 'model' | 'year'
let unknown = getProperty(taxi, 'unknown');

```

- 映射类型（从旧类型中国呢创建新类型的方式：readonly）
- 有条件类型

```ts
T
extends
U ? X : Y
// T 能赋值给U 类型为X 否则为Y
```

## 接口（interface）

```ts
interface Person {
    age?: number;
    name: string;
    readonly id: number;

    [propName: string]: any // 一个接口只能定义一个任意属性(number | string也可以)
    // 只要定义了 任意属性，确定属性 和 可选属性 的类型都必须是它的类型的子集
}

let tom: Person = {
    name: 'Tom',
    id: 1,
    gender: 'male', //
    height: 1, // 可以赋值多个任意属性
}
```

## 数组的类型（「类型 + 方括号）

```ts
let array: number[] = [1, 3, 5, 6, 7]
array.push('8');
//报错了啦！！！ 只接受 number 类型参数
```

- 数组泛型（Array<elemType>）

```ts
let fibonacci: Array<number> = [1, 1, 2, 3, 5];
```

- 接口表示数组

```ts
interface NumberArray {
    [index: number]: number
}

let fibonacci: NumberArray = [1, 1, 2, 3, 5]

```

- 类数组

```ts
interface IArguments {
    [index: number]: any;

    length: number
    callee: Function;
}

function sum() {
    let args: IArguments = arguments
}
```

## 函数的类型

```ts
// 函数声明
function sum(x: number, y: number): number {
    return x + y;
}

// 函数表达式
// => 表示函数定义（输入类型 => 输出类型）
let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
    return x + y;
};
```

- 接口定义函数

```ts
interface SearchFunc {
    (source: string, subString: string): boolean
}

let mySearch: SearchFunc = function (source: string, subString: string) {
    return source.search(subString) !== -1
}
```

- 函数参数
    - 可选参数必须在必需参数后面
    - 添加默认值的参数变为可选参数
    - 剩余参数（...rest）只能是最后一个参数

```ts
function buildName(firstName: string, secondName: string = 'cat', thirdName?: string, ...fourthArray: any[]) {
...
}
```

- 重载
    - 函数接收不同数量或类型，不同处理

```ts
function reverse(x: number | string): number | string | void {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```

## 类型断言

- 用来手动指定值的类型： 值 as 类型 或 <类型>值
- 将联合类型断言为其中一个类型
-

```ts
interface Cat {
    name: string

    run(): void
}

interface Fish {
    name: string;

    swim(): void
}

function isFish(animal: Cat | Fish) {
    if (typeof (animal as Fish).swim === 'function') {
        return true
    }
    return false
}
```

- 父类可以被断言为子类

```ts
  class ApiError extends Error {
    code: number = 0
}

class HttpError extends Error {
    statusCode: number = 0
}

function isApiError(error: Error) {
    // typeof (error as ApiError).code === 'number') 报错：父类 Error 中没有 code 属性
    if (error instanceof ApiError) { //判断error是否是它的实例
        return true
    }
    return false
}

interface ApiError extends Error {
    code: number
}

interface HttpError extends Error {
    statusCode: number
}

function isApiError(error: Error) {
    // instanceof这里不适用，接口是类型不是真正的值，在编译中被删除
    if (typeof (error as ApiError).code === 'number') {
        return true
    }
    return false
}
```

- 任何类型都可以断言为 any
    - 如果不是非常确定，就不要使用 as any
    - any 可以断言为任何类型

```ts
function getCacheData(key: string): any {
    return (window as any).cache[key];
}

interface Cat {
    name: string;

    run(): void;
}

// 类型断言 vs 类型声明核心区别：类型声明比类型断言更严格
const tom = getCacheData('tom') as Cat;
tom.run();
// 等价于 const tom: Cat = getCacheData('tom')

// 上面也可以用泛型：最优解决
function getCacheData<T>(key: string): T {
    return (window as any).cache[key];
}

interface Cat {
    name: string;

    run(): void;
}

const tom1 = getCacheData<Cat>('tom');
tom1.run();
```

- 要使得 A 能够被断言为 B，只需要 A 兼容 B 或 B 兼容 A 即可
- 双重断言

```ts
interface Cat {
    run(): void;
}

interface Fish {
    swim(): void;
}

function testCat(cat: Cat) {
    return (cat as any as Fish);
}

// ⚠️：除非不得已，否则别用
```

## 元组

- 已知元数量和类型（不必相同）的数组

```ts
let tom: [string, number];
tom[0] = 'Tom'

tom = ['Tom', 25]; //必须是所有的项
```

## 枚举（enum）

```ts
enum Color {Red, Green = 2, Blue}

let c: Color = Color.Green

let colorName: string = Color[2]; // 根据值找映射的key
console.log(colorName); // 2
```

- 常数枚举（const enum定义）
    - 与普通区别：它在编译阶段被删除，不能包含计算成员

```ts
const enum Directions {
    Up, Down, Left, Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
// 编译结果：var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
// 包含计算成员 会报错
const enum Color {Red, Green, Blue = "blue".length};

```

- 外部枚举（declare enum）常用于声明文件

```ts
declare const enum Directions {
    Up, Down, Left, Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```

## 类

### ES6中类的用法

```ts
// 使用 extends 实现继承
class Cat extends Animal {
    constructor(name) {
        super(name); // 调用父类的 constructor(name)
        console.log(this.name);
    }

    sayHi() {
        return 'Meow, ' + super.sayHi(); // 调用父类的 sayHi()
    }
}

let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```

### TypeScript 中类的用法

- public：默认所有的属性和方法都是public的
- private: 私有，不能在声明它的类外部访问
- protected: 区别于private 它可以在子类中访问
    - 构造函数修饰为 private 时，该类 不允许被继承或者实例化

```ts
class Animal {
    public name;

    private constructor(name) {
        this.name = name;
    }
}

class Cat extends Animal {
    constructor(name) {
        super(name);
    }
}

let a = new Animal('Jack'); // 报错
```

- 构造函数修饰为 protected时，该类 只允许被继承

```ts
class Animal {
    public name;

    protected constructor(name) {
        this.name = name;
    }
}

class Cat extends Animal {
    constructor(name) {
        super(name);
    }
}

let a = new Animal('Jack'); // 报错
```

- readonly 只能出现在 `属性声明`或`索引签名`或`构造函数`中
  -

```ts
class Animal {
    // public readonly name;
    public constructor(public readonly name) {
        // this.name = name;
    }
}
```

- 抽象类
    - abstract 定义抽象类和其中的抽象方法
    - 不允许被实例化（不能new）
    - 抽象方法必须被子类实现

```ts
abstract class Animal {
    public name;

    public constructor(name) {
        this.name = name
    }

    public abstract sayHi();
}

class Cat extends Animal {
    public sayHi() {
        console.log(`${this.name} 实现啦！！！！`)
    }
}

let cat = new Cat('Tom')
```

# 类与接口

- 类实现接口（防盗门 <--报警器--> 车）
    - implements 关键字

```ts
interface Alarm {
    alert(): void
}

class Door {

}

class SecurityDoor extends Door implements Alarm {
    alert() {
        console.log('SecurityDoor alert')
    }
}

class Car implements Alarm {
    alert() {
        console.log('Car alert')
    }
}
```

- 接口继承接口

```ts
interface Alarm {
    alert(): void;
}

interface LightableAlarm extends Alarm {
    lightOn(): void;

    lightOff(): void;
}
```

- 接口继承类
    - `接口继承类` 和 `接口继承接口` 没本质区别
    -
        -

```ts
// 创建Point的类，也创建了 Point 的类型（实例的类型）
class Point {
    x: number;
    y: number;

    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}

interface PointInstanceType {  // 声明Point 类创建的类型 Point 不包含构造函数、静态属性｜方法
    x: number;
    y: number;
}

// 等价于 interface Point3d extends PointInstanceType
interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};

// 可以理解为：定义了一个接口 Point3d 继承了 另一个接口 PointInstanceType
```

# 泛型

- 定义函数、接口、类时，预先不指定具体类型，在使用时指定类型

```ts
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```

- 多个类型参数

```ts
// 用来交换输入的元组
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}

swap([7, 'seven']); // ['seven', 7]
```

- 泛型约束
    - 使用 extends 约束泛型T必须符合接口

```ts
interface Lengthwise {
    length: number;
}

// 使用 extends 约束
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

loggingIdentity(7); // 报错 没有length
```

- 多个类型参数约束

```ts
// T 继承 U，
function copyFields<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = (<T>source)[id];  // <T>source  source断言为T类型
    }
    return target;
}

let x = {a: 1, b: 2, c: 3, d: 4};

copyFields(x, {b: 10, d: 20});
```

- 泛型类

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
    return x + y;
};
```


