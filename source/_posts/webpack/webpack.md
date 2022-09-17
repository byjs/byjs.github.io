---
title: webpack
date: 2019-09-18
tag: Webpack

cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/9a879ad324876c5212340ecd23346bb021065dfc.jpg
---

# webpack

静态模块打包器。处理应用程序时，它会递归地构建一个依赖关系图，包含应用程序需要的每个模块，将所有这些模块打包成一个或多个 bundle

## 入口（entry）

- webpack 应该使用哪个模块作为构建内部依赖图的开始
- webpack 配置的 `entey` 属性 指定入口起点。默认为 `./src`

### 单个入口

```js
const config = {
    entry: './path/to/my/entry/file.js'  //  简写
    // entry: {
    //   main: './path/to/my/entry/file.js'
    // }
};

module.exports = config;

```

### 对象

```js
// 用法：entry: {[entryChunkName: string]: string|Array<string>}
// 分离应用程序（app）和第三方库（vendor）
const config = {
    entry: {
        app: './src/app.js',
        vendors: './src/vendors.js'
    }
};
```

### 多页面应用程序

```js
const config = {
    entry: {
        pageOne: './src/pageOne/index.js',
        pageTwo: './src/pageTwo/index.js',
        pageThree: './src/pageThree/index.js'
    }
};
// webpack 需要3个独立分离的依赖图
```

## 出口（output）

- output 告诉webpack 在那里输出它创建的包，以及如何命名这些文件，默认为 `./dist`

- `filename`：输出文件名  `path`：输出目录的绝对路径

```js
// webpack.config.js
const path = require('path'); // Node.js核心模块，用于操作文件路径

module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
    }
};
```

### 多个入口起点

```js
{
    entry: {
        app: './src/app.js',
            search
    :
        './src/search.js'
    }
,
    output: {
        filename: '[name].js',
            path
    :
        __dirname + '/dist'
    }
}

// 写入到硬盘：./dist/app.js, ./dist/search.js

```

### 高级进阶

```js
output: {
    path: "/home/proj/cdn/assets/[hash]",
        publicPath
:
    "http://cdn.example.com/assets/[hash]/"
}
// 编译时不知道 publicPath，可以先忽略它，并且在入口起点设置 __webpack_public_path__
__webpack_public_path__ = myRuntimePublicPath

// 剩余的应用程序入口
```

## loader

loader可以帮助webpack 处理 非js文件（**webpack自身只理解js**），将所有类型的文件转换为 webpack 能够处理的 **有效模块**。能够导入（import）任何类型的模块（.css文件）

- 配置中定义loader，定义在 `module.rules`中
    - `test` 属性：标识应该被对应的 loader 进行转换的某个或某些文件
    - `use` 属性：使用哪个loader 进行转换

```js
const path = require('path');

const config = {
    output: {
        filename: 'my-first-webpack.bundle.js'
    },
    module: {
        rules: [
            {test: /\.txt$/, use: 'raw-loader'}
        ]
    }
};

module.exports = config;
```  

### 使用 loader

- 配置（推荐）：`webpack.config.js` 中指定
- 内联：每个 `import` 语句中指定

```js
// ! 分割loader
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

- CLI：在shell 命令中指定

```js
webpack--
module - bind
jade - loader--
module - bind
'css=style-loader!css-loader'
```

### 特性

- 支持链式传递（第一个loader返回值给下一个，最后一个返回webpack）
- 可以同步，也可以异步
- 运行在`node.js`中，并能执行任何可能的操作
- 接收查询参数
- 能使用`options`对象配置
- 除了使用 `package.json`常见的`main`属性，还可以将普通的npm 模块导出为loader：定义一个 `loader` 字段
- loader 能产生额外的任意文件
- 自定义loader（导出为函数）命名：xxx-loader

## 模式

```js
module.exports = {
    mode: 'production' // development / production
};
// 或在CLI参数重传递
`webpack --mode=production`
```

- mode: development

```js
// 会将 process.env.NODE_ENV 的值设为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin
module.exports = {
    mode: 'development'
    // plugins: [
    //     new webpack.NamedModulesPlugin(),
    //     new webpack.DefinePlugin({"process.env.NODE_ENV": JSON.stringify("development")}),
    // ]
}
```

- mode: production

```js
// webpack.production.config.js
module.exports = {
    mode: 'production',
    // plugins: [
    //     new UglifyJsPlugin(/* ... */),
    //     new webpack.DefinePlugin({"process.env.NODE_ENV": JSON.stringify("production")}),
    //     new webpack.optimize.ModuleConcatenationPlugin(),
    //     new webpack.NoEmitOnErrorsPlugin()
    // ]
}
```

## 插件（plugins）

- 执行范围更广的任务：打包优化和压缩，重新定义环境中的变量

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

const config = {
    module: {
        rules: [
            {test: /\.txt$/, use: 'raw-loader'}
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({template: './src/index.html'})
    ]
};

module.exports = config;
```

- 插件是一个具有`apply`属性的js对象，`apply` 属性会被 `webpack` 调用

# 模块（modules）

- `webpack`模块能以各种方式表达它们的依赖关系
    - `es5 import` 语句
    - `CommonJS require` 语句
    - `AMD define、require` 语句
    - `css/sass/less`文件的 `@import` 语句
    - 样式 `url(...)` 或HTML文件 `<img src=...>`中的图片链接

### 支持的模块类型

- 包括：CoffeeScript、TypeScript、ESNext(Babel)、Sass、Less、Stylus

# 模块解析

# 依赖图

- `webpack` 接收非代码资源（图像或web字体）作为依赖（依赖关系：一个文件依赖于另一个文件）提供给应用
- `webpack` 从命令行或配置文件定义的入口起点开始，它会递归构建一个 依赖图（包含应用所需每个模块，将所有模块打包，通常只有一个）

# manifest 和 runtime

- 管理所有模块的交互
- 编译器开始执行、解析和映射应用程序时，所有模块详细数据会保留在 `Manifest` ，完成打包并发送到浏览器时，会在运行时通过`Manifest`解析和记载模块
  `import`或`require`都会转为`__webpack_require__`指向模块标识符，通过使用`manifest`中的数据，`runtime`能够查询模块标识符，检索到对应的模块

# 构建目标

```js
var path = require('path');
var serverConfig = {
    target: 'node',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'lib.node.js'
    }
    //…
};

var clientConfig = {
    target: 'web', // <=== 默认是 'web'，可省略
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'lib.js'
    }
    //…
};

module.exports = [serverConfig, clientConfig];
```

# 模块热替换

- 会在应用运行过程中替换、添加或删除模块，而无需加载整个页面，通过几个方式来加速开发
    - 保留重新加载页面的应用程序状态
    - 只更新变更内容
    - 调整样式更加快速

- 在程序中置换模块
    - 应用程序代码 要求 模块热替换 runtime 检查更新
    - runtime（异步）下载更新，通知应用程序
    - 应用程序代码 要求 HMR runtime 应用跟新

# webpack 构建流程

- 初始化：启动构建，读取、合并并配置参数（webpack.config.js 中的各个配置项拷贝到 options 对象中），加载Plugin
- 编译：从Entry发出，调用Loader翻译模块，再找出模块的依赖模块，递归进行编译处理
- 输出：编译后的模块组成 `chunk`，再把`chunk` 转换成文件 加入到输出列表

- 核心：插件化设计：在不同的阶段执行相应的一些插件执行某些功能
- webpack 的生命周期hook，实际就是一个个插件的集合，某个阶段需要挂载某些插件

# Loader 和 plugin 的 区别

- loader（转换器）用于转化文件，也可以压缩，打包，语言翻译，将A文件进行编译输出B文件，操作文件
- plugin（扩展器）除了打包优化和压缩之外，在webpack运行的声明周期中会广播出许多事件，plugin会监听这些事件，在合适的时机通过webpack提供的API改变输出结果

# 常用loader

- file-loader：把文件输出到一个文件夹中，在代码中通过相对URL 去引用输出的文件
- image-loader：加载并压缩图片
- babel-loader：ES6转换为ES5
- css-loader：记载css，支持模块化、压缩、文件导入等
- source-map-loader：加载额外的Source Map文件 方便调试

# 常用plugin

- html-webpack-plugin：可以根据模板自动生成html代码，并自动引用css和js文件
- terser-webpack-plugin：压缩ES6（webpack 4）
- mini-css-extract-plugin：分离样式文件
- webpack内置CommonsChunkPlugin：提高打包效率，将第三方库和业务代码分开打包
- clean-webpack-plugin：目录清理

































