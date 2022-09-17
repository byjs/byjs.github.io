---
title: webpack - 指南
date: 2019-09-20
tag: Webpack
cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/6cab717967c13e87efcce0d7e97e2ee005863ae8.jpg
---

# 模块热替换

## 启用 HMR

- 更新 `webpack-dev-server`配置，使用webpack内置的HMR插件

```js
 module.exports = {
    devServer: {
        contentBase: './dist',
        hot: true  // +
    },
    plugins: [
        new CleanWebpackPlugin(['dist']),
        new HtmlWebpackPlugin({
            title: 'Hot Module Replacement'
        }),
        new webpack.NamedModulesPlugin(),  // +
        new webpack.HotModuleReplacementPlugin() // +
    ]
}

// NamedModulesPlugin查看要修补的依赖
```

## 通过 Node.js API

- 当使用 webpack dev server 和 Node.js API 时，不要将dev server选项放在webpack配置对象中，而是在创建选项时作为第二个参数传递

```js
const webpackDevServer = require('webpack-dev-server');
const webpack = require('webpack');

const config = require('./webpack.config.js');
const options = {
    contentBase: './dist',
    hot: true,
    host: 'localhost'
};

webpackDevServer.addDevServerEntrypoints(config, options);
const compiler = webpack(config);
const server = new webpackDevServer(compiler, options);

server.listen(5000, 'localhost', () => {
    // ...
});
```

# tree shaking

- 用于描述移除 js 上下文中的未引用代码，它依赖es5中的静态结构特性（如：import、export）
- 通过 `package.json` 的 `sideEffects`属性作为标记，表明项目中哪些文件时纯的es5模块，可以安全删除文件中未使用的部分

```js
{
    "name"
:
    "your-project",
        "sideEffects"
:
    false
}
```

- 压缩输出

```js
mode: "production"
// 使用 --optimize-minimize
```

# 生产环境构建

- 避免在生产环境中使用 `inline-***` 和 `eval-***` 它们可以增加包大小影响性能

## 指定环境

- 当使用 `process.env.NODE_ENV === 'production'` 针对具体用户环境删除或添加重要代码，可以使用`webpck` 内置 的`DefinePlugin` 为所有的依赖定义这个变量

```js
new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('production')
})
```

- 添加此插件后，任何位于`/src`的本地代码都可以关联到 `process.env.NODE_ENV`环境变量

# 代码分离

## 防止重复

- `CommonsChunkPlugin` 可以将公共的依赖模块提取到已有的入口chunk或者提取到一个新生成的chunk中

```js
new webpack.optimize.CommonsChunkPlugin({
    name: 'common' // 指定公共 bundle 的名称。
})
```

- 代码分离的插件和loaders
    - `ExtractTextPlugin`：用于将css从主应用程序中分离
    - `bundle-loader`：用于分离代码和延迟加载生成的bundle
    - `promise-loader`：类似`bundle-loader`

# 动态导入

- `import()` 会返回一个`promise` 可以和`async函数` 一起用，需要使用像Babel这样的预处理器

```js
async function getComponent() {
    const _ = await import(/* webpackChunkName: "lodash" */ 'lodash')
}
```

- `require.ensure`

# 缓存

- 通过使用 `output.filename`进行文件名替换，【hash】替换用于在文件名中包含一个构建相关的hash,更好的方式是用[chunkhash]

```js
module.exports = {
    output: {
        filename: '[name].[chunkhash].js',
        path: path.resolve(__dirname, 'dist')
    }
}
```

## 提取模版

- `CommonChunkPlugin`能在每次修改后的构建结果中，将webpack的样板和manifest 提取出来

```js
 new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest'
})
```

## 模块标识符

- 对于依赖第三方构建保持后缀不变

```js
plugins:[
    new webpack.HashedModuleIdsPlugin(),
]
```

# shimming

## 全局变量

```js
plugins: [
    new webpack.ProvidePlugin({
        // _: 'lodash'    // 在页面中不用导入 直接使用 _.join
        join: ['lodash', 'join'] // join暴露全局 直接join 将 lodash 没用到的部分去除
    })
]
```

## 细粒度 shimming

```js
// 一般模块依赖 `this`指向`window`在CommonJS中可以这样
module: {
    rules: [
        {
            test: require.resolve('index.js'),
            use: 'imports-loader?this=>window'
        }
    ]
}
,
```

## 全局 export

```js
// globals.js 
var file = 'blah.txt';
var helpers = {
    test: function () {
        console.log('test something');
    },
    parse: function () {
        console.log('parse something');
    }
}

module:{
    rules:[{
        test: require.resolve('globals.js'),
        use: 'exports-loader?file,parse=helpers.parse'
    }]
}

// 使用：import { file, parse } from './globals.js'
```

## 加载 polyfills