---
title: vue-cli webpack 配置解析
date: 2017-08-24 20:56:29
tags: [vue-cli,webpack]
categories: 前端
---
最近对vue官方脚手架vue－cli中的webpack配置做了一番研究，良心至极，省去了繁琐的配置。最近脑容量真的是不够用啊，还是记录下以便日后查阅吧，本文基于vue-cli版本为2.8.1。
<!--more-->
------

## dev-server.js

```javascript
// 检查 Node 和 npm 版本
require('./check-versions')()

// 获取 config/index.js 的默认配置
var config = require('../config')

// 如果 Node 的环境无法判断当前是 dev/product 环境
// 使用 config.dev.env.NODE_ENV 作为当前的环境
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}

// 一个可以强制打开浏览器并跳转到指定 url 的插件
var opn = require('opn')
// 使用 NodeJS 自带的文件路径工具
var path = require('path')
// 使用 express
var express = require('express')
// 使用 webpack
var webpack = require('webpack')
// 使用 proxyTable
var proxyMiddleware = require('http-proxy-middleware')
// 使用 dev 环境的 webpack 配置
var webpackConfig = require('./webpack.dev.conf')

// default port where dev server listens for incoming traffic
// 如果没有指定运行端口，使用 config.dev.port 作为运行端口
var port = process.env.PORT || config.dev.port
// automatically open browser, if not set will be false
// 自动打开浏览器
var autoOpenBrowser = !!config.dev.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
// 使用 config.dev.proxyTable 的配置作为 proxyTable 的代理配置
var proxyTable = config.dev.proxyTable

// 使用 express 启动一个服务
var app = express()
// 启动 webpack 进行编译
var compiler = webpack(webpackConfig)

// 启动 webpack-dev-midddleware,将编译后的文件暂存到内存中
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

// 启动 webpack-hot-middleware，也就是我们常说的 Hot-reload
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {},
  heartbeat: 2000
})
// force page reload when html-webpack-plugin template changes
// 当 html-webpack-plugin 模板更新的时候强制刷新页面
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

// proxy api requests
// 将 proxyTable 中的请求配置挂载到启动的 express 服务上
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
// 使用 connect-history-api-fallback 配置资源，如果不匹配就可以重定向到指定地址
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 将暂存到内存中的 webpack 编译后的文件挂载到 express 服务上
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
// 将 Hot-reload 挂载到 express 服务上并且输出相关的状态、错误
app.use(hotMiddleware)

// serve pure static assets
// 拼接 static 文件夹得静态资源路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
// 为静态资源提供相应服务
app.use(staticPath, express.static('./static'))

// 让我们这个 express 服务监听 port 的请求，并且将此服务作为 dev-server.js 的接口暴露
var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})

console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  // when env is testing, don't need open it
  // 如果不是测试环境，自动打开浏览器并跳转到我们的开发地址
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
  _resolve()
})

var server = app.listen(port)

module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```
## webpack.dev.conf.js

```javascript
// 使用了一些小工具
var utils = require('./utils')
// 使用 webpack
var webpack = require('webpack')
// 使用了config/index.js
var config = require('../config')
// 使用 webpack 配置合并插件
var merge = require('webpack-merge')
// 加载 webpack.base.conf
var baseWebpackConfig = require('./webpack.base.conf')
// 使用 html-webpack-plugin 插件，这个插件可以帮我们自动生成 html 并且注入到 .html 文件中
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 使用 friendly-errors-webpack-plugin
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
// 将 hot-reload 相对路径添加到 webpack.base.conf 对应的 entry 前
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

// 将 webpack.dev.conf.js 的配置和 webpack.base.conf.js 的配置合并
module.exports = merge(baseWebpackConfig, {
  module: {
    // 使用 styleLoaders
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
  // 使用 #cheap-module-eval-source-map 模式作为开发工具
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    // definePlugin 接收字符串插入到代码中，需要的话可以写上 JS 的字符串
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    // HotModule 插件在页面进行变更的时候只会重回对应的页面模块，不会重绘整个 html 文件
    new webpack.HotModuleReplacementPlugin(),
    // 使用了 NoEmitOnErrorsPlugin 后页面的报错不会阻塞，但是会在编译结束后报错
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    // 将 index.html 作为入口,注入 html 代码后生成 index.html 文件
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```

## webpack.base.conf.js

```javascript
// 使用 NodeJS 自带的文件路径插件
var path = require('path')
// 引入一些小工具
var utils = require('./utils')
// 引入 config/index.js
var config = require('../config')
// 引入 vue-loader.ocnf.js
var vueLoaderConfig = require('./vue-loader.conf')

// 拼接我们的工作区路径为一个绝对路径
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  // 编译文件入口
  entry: {
    app: './src/main.js'
  },
  output: {
    // 编译输出的静态文件根路径
    path: config.build.assetsRoot,
    // 编译输出文件名
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    // 自动补全的扩展名
    extensions: ['.js', '.vue', '.json'],
    alias: {
      // 默认路径代理
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  },
  module: {
    // 需要处理的文件以及使用的 loader
    rules: [
      {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```

## config/index.js

```javascript
// see http://vuejs-templates.github.io/webpack for documentation.
var path = require('path')

module.exports = {
  // production 环境
  build: {
    // 使用 config/prod.dev.js 中定义的编译环境
    env: require('./prod.env'),
    // 编译输入的 index.html 文件
    index: path.resolve(__dirname, '../dist/index.html'),
    // 编译输出的静态资源路径
    assetsRoot: path.resolve(__dirname, '../dist'),
    // 编译输出的二级目录
    assetsSubDirectory: 'static',
    // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
    assetsPublicPath: '/',
    // 是否开启 cssSourceMap
    productionSourceMap: true,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    // 是否开启 gzip
    productionGzip: false,
    // 需要使用 gzip 压缩的文件扩展名
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  // dev 环境
  dev: {
    // 使用 config/dev.env.js 中定义的编译环境
    env: require('./dev.env'),
    // 运行测试页面的端口
    port: 8080,
    // 自动打开浏览器
    autoOpenBrowser: true,
    // 编译输出的二级目录
    assetsSubDirectory: 'static',
    // 编译发布的根目录，可配置为资源服务器域名或 CDN 域名
    assetsPublicPath: '/',
    // 需要 proxyTable 代理的接口（可跨域）
    proxyTable: {},
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    // 是否开启 cssSourceMap(因为一些 bug 此选项默认关闭，详情可参考 https://github.com/webpack/css-loader#sourcemaps)
    cssSourceMap: false
  }
}
```

## build.js

```javascript
// 检查 Node 和 npm 版本
require('./check-versions')()

process.env.NODE_ENV = 'production'

// 一个很好看的 loading 插件
var ora = require('ora')
// node环境下 rm -rf 的命令库
var rm = require('rimraf')
// 使用 NodeJS 自带的文件路径工具
var path = require('path')
// 终端显示带颜色的文字
var chalk = require('chalk')
// 加载 webpack
var webpack = require('webpack')
// 加载 config/index.js
var config = require('../config')
// 加载 webpack.prod.conf.js
var webpackConfig = require('./webpack.prod.conf')

var spinner = ora('building for production...')
spinner.start()

rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
    // 开始 webpack 的编译
  webpack(webpackConfig, function (err, stats) {
    // 编译成功的回调
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```

## webpack.prod.conf.js

```javascript
// 使用 NodeJS 自带的文件路径工具
var path = require('path')
// 使用了一些小工具
var utils = require('./utils')
// 加载 webpack
var webpack = require('webpack')
// 加载 config/index.js
var config = require('../config')
// 加载 webpack 配置合并工具
var merge = require('webpack-merge')
// 加载 webpack.base.conf.js
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
// 一个可以插入 html 并且创建新的 .html 文件的插件
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 一个 webpack 扩展，可以提取一些代码并且将它们和文件分离开
// 如果我们想将 webpack 打包成一个文件 css js 分离开，那我们需要这个插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

var env = config.build.env

// webpack.base.conf.js
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    // 使用的 loader
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  // 是否使用 #source-map 开发工具
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    // 编译输出目录
    path: config.build.assetsRoot,
    // 编译输出文件名
    filename: utils.assetsPath('js/[name].[chunkhash].js'), // 我们可以在 hash 后加 :6 决定使用几位 hash 值
    // 没有指定输出名的文件输出的文件名
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    // definePlugin 接收字符串插入到代码当中, 所以你需要的话可以写上 JS 的字符串
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // 压缩 js (同样可以压缩 css)
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
    // 将 CSS 分离出来
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    // 构建要输出的 index.html 文件，HTMLWebpackPlugin 可以生成一个 html 并且在其中插入你构建生成的资源
    new HtmlWebpackPlugin({
      // 生成的 html 文件名
      filename: config.build.index,
      // 使用的模板
      template: 'index.html',
      // 是否注入 html （有多重注入方式，可以选择注入的位置）
      inject: true,
      // 压缩的方式
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    // split vendor js into its own file
    // CommonsChunkPlugin用于生成在入口点之间共享的公共模块（比如jquery，vue）的块并将它们分成独立的包。而为什么要new两次这个插件，这是一个很经典的bug的解决方案，在webpack的一个issues有过深入的讨论webpack/webpack#1315 .----为了将项目中的第三方依赖代码抽离出来，官方文档上推荐使用这个插件，当我们在项目里实际使用之后，发现一旦更改了 app.js 内的代码，vendor.js 的 hash 也会改变，那么下次上线时，用户仍然需要重新下载 vendor.js 与 app.js
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
// 开启 gzip 的情况下使用下方的配置
if (config.build.productionGzip) {
  // 加载 compression-webpack-plugin 插件
  var CompressionWebpackPlugin = require('compression-webpack-plugin')
  // 向webpackconfig.plugins中加入下方的插件
  webpackConfig.plugins.push(
    // 使用 compression-webpack-plugin 插件进行压缩
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig
```
