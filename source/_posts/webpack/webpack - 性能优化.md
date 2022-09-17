---
title: webpack - 性能优化
date: 2019-10-03
tag: Webpack
cover: https://byblog.oss-cn-hangzhou.aliyuncs.com/e0afbb89286c6e5fe44fec1db6d6a9d5e80e8bd0.jpg
---

# 优化

## 使用 include 或者 exclude 配置，避免重复打包

- 引入的一些插件、类库都是被打包过了的，用babel编译的时候，需要配置一下，给已经编译过的语法剔除掉，这样就能减少打包时间
    - 比如在项目中引入jq插件，取用jq时，npm搜索会先从当前目录的node_modules中找，找不到，就往上一级目录node_modules，一直往上找到磁盘。如果都没有，才报错
    - 找到 node_modules 包之后找到 package.json 文件，在文件中找到 入口文件去加载，加载后发现已经是编译过的文件了

```js
{
    test: /\.js$/;
    include: path.resolve(__dirname, '../src'); // 指定编译文件夹
    exclude: /node_modules/;  // 排除指定文件夹
    use: [{loader: 'babel-loader'}]
}
```

## 利用缓存减少打包时间

```js
{
    test: /\.js$/;
    use: [{
        loader: 'babel-loader?chaheDirectory' // 文件没被改动使用缓存
    }]
}
```

- 一些公用不改动的库可以使用，还可 `cache-loader` 在开销较大的loader前使用即可

```js
{
    test: /\.js$/;
    use:[
        'cache-loader',
        'babel-loader'
    ];
    include:path.resolve('src')
}
```

## 合理使用plugin，减少打包时间和体积

- 尽量选择官方推荐的plugin
- 使用插件避免引用无用模块代码

    - 比如：使用moment库时， 打包默认会把整个库引入进来，这样导致库非常大，可以使用 `igonrePlugin` 忽略插件的某个无用文件夹，这样就可以减少打包体积
    - 比如：使用压缩css的OptimizeCSSAssetsPlugin时 只需在生产环境下对代码做压缩，在开发环境下就不需要这个插件，缩短压缩时间

## 合理配置relosve，防止减慢打包时间

- 引入es6模块时，不用写文件后缀也能引用进来，这就是 webpack 的 relosve 配置参数

```js
resolve:{
    extensions:['js', 'jsx']
}
```

- 比如jpg、css、png等，由于不写后缀，在查找的时候，会给`extensions`数组中所有元素遍历完找不到才会报错，增加了查找时间，所以要合理利用

## 启用多线程打包

- 使用happy、推荐使用 `thread-loader`、`ParallelUglifyPlugin`
- 项目较大推荐使用，项目较小打包开启多线程会有额外的性能开销，反而将时间拖慢了

### happypack

- 本身就是对node开启一个多进程打包

```js
const HappyPack = require('happypack')
// 在loader中使用
{
    test: /\.js$/;
    use: ['happypack/loader?id=babel'];
    include: srcPath
}
// 在plugin中使用 HappyPack
new HappyPack({
    id: 'babel', // 用唯一标识符id 表示 当前的 HappyPack 用来处理一类特定的文件
    loaders: ['babel-loader?cacheDirectory'] // 处理 .js文件 开启缓存
})
```

### thread-loader

- `thread-loader` 放置在其他loader之前，之后的loader就会在一个单独的worker池中运行，并且还支持定义配置，方便性能优化

```js
{
    test:/\.js$/;
    exclude:/node_modules/;
    // 创建一个js worker 池
    use:[
        'thread-loader', // 直接在loader之前使用
        'babel-loader'
    ]
}
,
//  自定义配置行
use[
    {
        loader: "thread-loader",
        // loaders with equal options will share worker pools
        // 设置同样option的loaders会共享
        options: {
            // worker的数量，默认是cpu核心数
            workers: 2,
            // 一个worker并行的job数量，默认为20
            workerParallelJobs: 50,
            // 添加额外的node js 参数
            workerNodeArgs: ['--max-old-space-size=1024'],
            // 允许重新生成一个dead work pool
            // 这个过程会降低整体编译速度
            // 开发环境应该设置为false
            poolRespawn: false,
            //空闲多少秒后，干掉work 进程
            // 默认是500ms
            // 当处于监听模式下，可以设置为无限大，让worker一直存在
            poolTimeout: 2000,
            // pool 分配给workder的job数量
            // 默认是200
            // 设置的越低效率会更低，但是job分布会更均匀
            poolParallelJobs: 50,
        }
    }
'babel-loader'
]

```

### ParallelUglifyPlugin

- 压缩JS 需要先将代码解析成AST语法树，根据复杂规则分析和处理AST，最后将AST还原成JS，这个过程涉及大量计算，因此比较耗时

```js
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin')
plugin:[
    new ParallelUglifyPlugin({
        // c传递给 UglifyJS的参数     uglifyJS压缩
        uglifyJS: {
            output: {
                beautify: false, // 最紧凑的输出
                comments: false  // 删除所有注释
            },
            compress: {
                drop_console: true, // 删除所有的 console语句，可以兼容ie
                collapse_vars: true, // 内嵌定义了但是只用到一次的变量
                reduce_vars: true // 提取出现多次但是没有定义成变量去引用的静态值
            }
        }
    })
]
```

## 开发中使用热更新替换自动跟新

- 项目体积过大，每次改完代码都要刷新页面

```js
const webpack = require('webpack')
plugin:[
    new webpack.HotModuleReplacementPlugin() // 使用webpack提供的热更新方式
];
devServer:{
    hot:true
}
```

## 使用 DllPlugin 插件，提高打包时间

- 使用比较稳定的库时，比如：react、vue、jq 几个月都不会更新，就不需要重复打包，只需要打包一次，下次打包引用即可

- 编写dll的配置文件

```js
const path = require('path')
const DllPlugin = require('webpack/lib/DllPlugin')
const {srcPath, distPath} = require('./paths')

module.exports = {
    mode: 'development',
    entry: {
        react: ['react', 'react-dom'] // 以 react 为例
    },
    output: {
        filename: '[name].dll.js', // 输出的动态链接库的文件名称，[name]代表当前动态链接库的名称：上面entry 中 react
        path: distPath, // 输出文件放到dist目录下，
        library: '_dll_[name]' // 存放动态链接库的全局变量名称，如对应react就是 _dll_react   加_dll_ 为了防止全局变量冲突
    },
    plugins: [
        new DllPlugin({
            name: '_dll_[name]', // 动态链接库的全局变量名称，需要和 output.library中保持一致
            // 字段的值就是输出的 manifest.json 中name字段的值 如：react.manifest.json中就有'name':'_dll_react'
            path: patj.join(distPath, '[name].manifest.json') // 描述动态链接库的manifest.json文件输出时的文件名称
        })
    ]
}
```

- 在webpack中配置映射关系，防止打包时再次引用npm包

```js
const DllReferencePlugin = require('webpack/lib/DllReferencePlugin')
plugins:[
    // 告诉 Webpack 使用了哪些动态链接库
    new DllReferencePlugin({
        manifest: require(paht.join(dispath, 'react.manifest.json'))
    })
]
```

- html中引用打包的公用模块，因为在配置 `DllReferencePlugin`
  时，底层其实时在实行的时候在window中寻找这个包，所以必须引入进来打包后的这个文件，相应的这个公用模块也会在window下挂一个全局变量，引入方式可以使用`AddAssetHtmlWebpackPlugin`插件引入，也可以手动引入

## 打包时合理使用hash，如果没改动的文件 命中缓存

```js
output:{
    filename:'bundle.[contentHash:8].js'; // 打包代码加 hash 后缀
    path:distPath
}
```

## 提取公用代码，代码分割

- 提取多个模块的公共代码，只需要打包一次，实现更小的代码体积

```js
optimization:{
    // 分割代码块
    splitChunks: {
        chunks: "async";//默认作用于异步chunk，值为all/initial/async/function(chunk),值为function时第一个参数为遍历所有入口chunk时的chunk模块，chunk._modules为chunk所有依赖的模块，通过chunk的名字和所有依赖模块的resource可以自由配置,会抽取所有满足条件chunk的公有模块，以及模块的所有依赖模块，包括css
        minSize: 30000; //表示在压缩前的最小模块大小,默认值是30kb
        minChunks: 1;  // 表示被引用次数，默认为1；
        maxAsyncRequests: 5;  //所有异步请求不得超过5个
        maxInitialRequests: 3;  //初始话并行请求不得超过3个
        automaticNameDelimiter:'~';//名称分隔符，默认是~
        name: true;  //打包后的名称，默认是chunk的名字通过分隔符（默认是～）分隔
        cacheGroups: { //设置缓存组用来抽取满足不同规则的chunk,下面以生成common为例
            common: {
                name: 'common' //抽取的chunk的名字
                chunks(chunk)
                {
                    //同外层的参数配置，覆盖外层的chunks，以chunk为维度进行抽取
                }
                test(module, chunks)
                {
                    //可以为字符串，正则表达式，函数，以module为维度进行抽取，只要是满足条件的module都会被抽取到该common的chunk中，为函数时第一个参数是遍历到的每一个模块，第二个参数是每一个引用到该模块的chunks数组。自己尝试过程中发现不能提取出css，待进一步验证。
                }
                priority: 10;  //优先级，一个chunk很可能满足多个缓存组，会被抽取到优先级高的缓存组中
                minChunks: 2;  //最少被几个chunk引用
                reuseExistingChunk: true;//  如果该chunk中引用了已经被抽取的chunk，直接引用该chunk，不会重复打包代码
                enforce: true  // 如果cacheGroup中没有设置minSize，则据此判断是否使用上层的minSize，true：则使用0，false：使用上层minSize
            }
        }
    }
}
```

## 使用 tree-shaking 去除无用代码减少代码体积

```js
// a.js
export const a = 'a';
export const b = 'b';

// b.js
import * as c from "./a";

export {c};

// index.js
import * as module from "./b";

console.log(module.c.a);

// 变量b不会被打包
mode:'prodution'
```

## `TerserPlugin.terserMinify` 替换 `esbuild`方式

- 默认使用的是 `TerserPlugin.terserMinify`

```js
`npm install esbuild - D`

new TerserPlugin({
    minify: TerserPlugin.esbuildMinify
}),
```































