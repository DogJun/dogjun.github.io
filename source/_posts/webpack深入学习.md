---
title: webpack深入学习
date: 2018-04-01 23:24:49
tags: webpack
categories: 前端
---
webpack4刚发布的那天就开始关注了，自己也是顺利把公司的项目从webpack3升级到webpack4，对一些知识做下归纳总结吧...
<!--more-->
------
# webpack what
可以看做一个模块化打包机，分析项目结构，处理模块化依赖，转换成为浏览器可运行的代码
- 代码转换: TypeScript 编译成 JavaScript、SCSS,LESS 编译成 CSS.
- 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片。
- 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。
- 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。
- 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。

# webpack的执行过程
webpack启动之后会从entry里配置的module开始递归解析entry依赖的所有module。每找到一个module，就会根据配置的loader去找出相应的转换规则，对module进行转换后，再解析出当前module依赖的module。这些模块会以entry为单位进行分组，一个entry和其所有依赖的module被分到一个组也就是一个chunk。最后webpack会把所有的chunk转换成文件输出，在整个流程中webpack会在恰当的时机执行plugin里定义的逻辑

# loader
loader是一个模块导出函数，在正则匹配成功的时候调用，webpack把文件数组传入进来，在this上下文可以访问loader API
- this.context: 当前处理文件的所在目录，假如当前 Loader 处理的文件是 /src/main.js，则 this.context 就等于 /src。
this.resource: 当前处理文件的完整请求路径，包括 querystring，例如 /src/main.js?name=1。
- this.resourcePath: 当前处理文件的路径，例如 /src/main.js。
- this.resourceQuery: 当前处理文件的 querystring。
- this.target: 等于 Webpack 配置中的 Target
- this.loadModule: 但 Loader 在处理一个文件时，如果依赖其它文件的处理结果才能得出当前文件的结果时， 就可以通过 - - -  this.loadModule(request: string, callback: function(err, source, sourceMap, module)) 去获得 request 对应文件的处理结果。
- this.resolve: 像 require 语句一样获得指定文件的完整路径，使用方法为 resolve(context: string, request: string, callback: function(err, result: string))。
- this.addDependency: 给当前处理文件添加其依赖的文件，以便再其依赖的文件发生变化时，会重新调用 Loader 处理该文件。使用方法为 addDependency(file: string)。
- this.addContextDependency: 和 addDependency 类似，但 addContextDependency 是把整个目录加入到当前正在处理文件的依赖中。使用方法为 addContextDependency(directory: string)。
- this.clearDependencies: 清除当前正在处理文件的所有依赖，使用方法为 clearDependencies()。
- this.emitFile: 输出一个文件，使用方法为 emitFile(name: string, content: Buffer|string, sourceMap: {...})。
- this.async: 返回一个回调函数，用于异步执行。

# plugin
webpack整个构建流程有许多钩子，开发者可以在指定的阶段加入自己的行为到webpack构建流程中。插件由以下构成:
- 一个 JavaScript 命名函数。
- 在插件函数的 prototype 上定义一个 apply 方法。
- 指定一个绑定到 webpack 自身的事件钩子。
- 处理 webpack 内部实例的特定数据。
- 功能完成后调用 webpack 提供的回调。
整个webpack流程由compiler和compilation构成,compiler只会创建一次，compilation如果开起了watch文件变化，那么会多次生成compilation. 那么这2个类下面生成了钩子.
写了一个小插件，将htmlwebapckplugin生成的css和js插入到模板指定的位置

```js
const pluginName = 'htmlAfterWebpackPlugin';
const assetsHelp = (data)=>{
    let css=[],js= []; 
    const dir ={
        js:item=>`<script src="${item}"></script>`,
        css:item=>`<link rel="stylesheet" href="${item}">`
    };
    for(let jsitem of data.js){
        js.push(dir.js(jsitem))
    }
    for(let cssitem of data.css){
        css.push(dir.css(cssitem))
    }
    return{
        css,
        js
    }
}
class htmlAfterWebpackPlugin {
    apply(compiler) {
        //html-webpack-plugin-before-html-processing
        compiler.hooks.compilation.tap(pluginName, compilation => {
            compilation.hooks.htmlWebpackPluginBeforeHtmlProcessing.tap(pluginName,htmlPluginData=>{
                let _html = htmlPluginData.html;
                const result = assetsHelp(htmlPluginData.assets);
                _html = _html.replace("<!--injectcss-->",result.css);
                _html = _html.replace("<!--injectjs-->",result.js);
                // console.log('得到的值',_html);
                htmlPluginData.html = _html;
            })
        });
    }
}
module.exports = htmlAfterWebpackPlugin;
```

# scope hoisting
如果项目是用ES2015的模块语法，并且webpack3+，那么建议启用这一插件，把所有的模块放到一个函数里，减少了函数声明，文件体积变小，函数作用域变少。
```js
new webpack.optimize.ModuleConcatenationPlugin()
```

# 提取第三方库
方便长期缓存第三方的库，新建一个入口，把第三方库作为一个chunk，生成vendor.js
```js
module.exports = {
    entry: {
        main: './src/index.js',
        vendor: ['react', 'react-dom'],
    }
}
```

# happypack
webpack是基于nodejs的，是单线程的，happypack可以开启多个子线程并发执行，子线程处理完后把结果交给主进程
```js
const HappyPack = require('happypack');
module.exports = {
	entry: './src/index.js',
	output: {
		path: path.join(__dirname, './dist'),
		filename: 'main.js',
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                exclude: /node_modules/,
                use: 'happypack/loader?id=babel',
            },
        ]
    },
    plugins: [
        new HappyPack({
            id: 'babel',
            threads: 4,
            loaders: ['babel-loader']
        }),
    ]
}
```
# webpack4 带来的新特性
## 环境支持
不再支持node4，node6，使用v8 5.0版本，支持93%的ES6语法，因为webpack4使用了很多JS新的语法，他们在新版本v8里进行了优化
## 0配置
webpack不再硬性要求webpack.config.js作为打包的入口配置文件，默认入口'./src',默认出口'./dist'
## Mode
webpack需要配置mode属性，可选development或者production
```json
"script": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production"
}
```
development模式：
- 浏览器调试工具
- 注释、开发阶段的详细错误日志和提示
- 快速和优化的增量构建机制  

production模式：
- 开启所有的优化代码
- 更小的bundle大小
- 去除掉只在开发阶段运行的代码
- Scope hoisting和Tree-shaking

## 插件和优化
webpack4删除了CommonsChunkPlugin插件，它使用内置API optimization.splitChunks 和 optimization.runtimeChunk，这意味着webpack会默认为你生成共享的代码块。其它插件变化如下:

- NoEmitOnErrorsPlugin 废弃，使用optimization.noEmitOnErrors替代，在生产环境中默认开启该插件。
- ModuleConcatenationPlugin 废弃，使用optimization.concatenateModules替代，在生产环境默认开启该插件。
- NamedModulesPlugin 废弃，使用optimization.namedModules替代，在生产环境默认开启。
uglifyjs-webpack-plugin升级到了v1.0版本, 默认开启缓存和并行功能。

## 开箱即用WebAssembly
对这一块不了解,后续补上

