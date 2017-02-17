---
title: 用vue来构建项目
date: 2016-9-14
category: web前端
tags: vue
---

## vue-cli
当我们开始一个新项目时，我们都会先编写一些预处理文件，和构建项目目录。而vue-cli就是为了做这方面工作的，它是由官方命令行工具，生成一套提前定义好的构建文件，和相应的文件。它带来现代化的前端开发流程。只需几分钟即可创建并启动一个带热重载、保存时静态检查以及可用于生产环境的构建配置的项目。

> $ npm install -g vue-cli
> $ vue init webpack my-project
> $ cd my-project
> $ npm install
> $ npm run dev

可以打开 http://localhost:8080 看看,就可以看到默认的页面了。

## 项目结构
![项目结构图](http://olgrokwaw.bkt.clouddn.com/sd.png)

1.build目录，主要利用webpack与node插件启动一些相关服务的js文件
2.config目录主要是针对开发环境，生产环境，测试环境的配置信息
3.src是我们自己开发时的源码目录
4.static是一些第三方库的包用到的静态资源目录

## 源码解读

#### 1. build/dev-server.js

npm run dev 其实是通过node执行这个文件，启动一个express服务器。这个服务器加载了几个中间件，如：http-proxy-middleware(代理转发中间件), webpack-dev-middleware(webpack开发中间件),webpack-hot-middleware(webpack热替换中间件)。

下面是一些原文加上注释
```javascript
// 引入检查版本js模块  
require('./check-versions')()  
// 引入配置文件信息模块  
var config = require('../config')  
// 判断开发环境  
if (!process.env.NODE_ENV) process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)  
// 引入nodejs的path模块，进行一些路径的操作，详情可以查看node官方的api  
var path = require('path')  
// 引入nodejs的一个框架express,可以帮助我们快速搭建一个node服务 github https://github.com/expressjs/express  
var express = require('express')  
// 引入node为webpack提供的一个模块服务 github https://github.com/webpack/webpack  
var webpack = require('webpack')  
// 可以指定打开指定的url使用指定的浏览器或者应用，详情可以去看一下github https://github.com/sindresorhus/opn  
var opn = require('opn')  
// 一个可以设置帮助我们进行服务器转发代理的中间件 https://github.com/chimurai/http-proxy-middleware  
var proxyMiddleware = require('http-proxy-middleware')  
// 根据当前启动环境选择加载相应的配置文件，webpack.prod.conf与webpack.dev.conf文件的说明后面也有  
var webpackConfig = process.env.NODE_ENV === 'testing'  
  ? require('./webpack.prod.conf')  
  : require('./webpack.dev.conf')  
  
// 端口号的设置  
var port = process.env.PORT || config.dev.port  
// 获取需要代理的服务api  
// https://github.com/chimurai/http-proxy-middleware  
var proxyTable = config.dev.proxyTable  
// 启动一个express服务  
var app = express()  
// 加载webpack配置  
var compiler = webpack(webpackConfig)  
// webpack的开发中间件，专门为webpack提供的一个简单的中间件，可以让文件都加载内存中，不去读写硬盘，并且当文件被改动的时候，不会刷新页面就会部署成功  
var devMiddleware = require('webpack-dev-middleware')(compiler, {  
  publicPath: webpackConfig.output.publicPath,  
  quiet: true  
})  
// 一个为webpack提供的热部署中间件。https://github.com/glenjamin/webpack-hot-middleware  
var hotMiddleware = require('webpack-hot-middleware')(compiler, {  
  log: () => {}  
})  
// 当html被改变的时候，让html被强制部署，使用这个中间件html-webpack-plugin，https://github.com/ampedandwired/html-webpack-plugin  
compiler.plugin('compilation', function (compilation) {  
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {  
    hotMiddleware.publish({ action: 'reload' })  
    cb()  
  })  
})  
  
// 遍历代理的配置信息,并且使用中间件加载进去  
Object.keys(proxyTable).forEach(function (context) {  
  var options = proxyTable[context]  
  if (typeof options === 'string') {  
    options = { target: options }  
  }  
  app.use(proxyMiddleware(context, options))  
})  
  
// 当访问找不到的页面的时候，该中间件指定了一个默认的页面返回https://github.com/bripkens/connect-history-api-fallback  
app.use(require('connect-history-api-fallback')())  
  
// 使用中间件  
app.use(devMiddleware)  
  
// 热部署  
app.use(hotMiddleware)  
  
// 根据配置信息拼接一个目录路径，然后将该路径下的文件交给express的静态目录管理  
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)  
app.use(staticPath, express.static('./static'))  
  
var uri = 'http://localhost:' + port  
  
devMiddleware.waitUntilValid(function () {  
  console.log('> Listening at ' + uri + '\n')  
})  
// 导出的对象  
module.exports = app.listen(port, function (err) {  
  if (err) {  
    console.log(err)  
    return  
  }  
  
  // when env is testing, don't need open it  
  if (process.env.NODE_ENV !== 'testing') {  
    opn(uri)  
  }  
})  
```
#### 2. build/dev-client.js

这个文件主要是注入到浏览器，监听webpack-hot-middleware传过来的事件，如reload action。用于代码热更新等。

#### 3. build/build.js

npm run build，主要是执行这个文件，主要的动作是下面

* copy static目录到dist目录
* 根据webpack.prod.conf.js配置文件，对源码进行编译，输出到dist目录

#### 4. build/utils.js

主要是生成webpack css-loader, sass-loader(与styles相关的Loader)等配置。

## webpack的文件

#### 1. webpack.base.conf.js

```javascript
var path = require('path')  
var config = require('../config')  
// 工具类，下面会用到  
var utils = require('./utils')  
// 工程目录，就是当前目录build的上一层目录  
var projectRoot = path.resolve(__dirname, '../')  
  
var env = process.env.NODE_ENV  
// 是否在开发环境中使用cssSourceMap,默认是false,该配置信息在config目录下的index.js中可以查看  
var cssSourceMapDev = (env === 'development' && config.dev.cssSourceMap)  
var cssSourceMapProd = (env === 'production' && config.build.productionSourceMap)  
var useCssSourceMap = cssSourceMapDev || cssSourceMapProd  
// 导出的对象，就是webpack的配置项，详情可以参考的webpack的配置说明，这里会将出现的都一一说明一下  
module.exports = {  
  // 指明入口函数  
  entry: {  
    app: './src/main.js'  
  },  
  // 输出配置项  
  output: {  
    // 路径，从config/index读取的，值为：工程目录下的dist目录，需要的自定义的也可以去修改  
    path: config.build.assetsRoot,  
    // 发布路径，这里是的值为/，正式生产环境可能是服务器上的一个路径,也可以自定义  
    publicPath: process.env.NODE_ENV === 'production' ? config.build.assetsPublicPath : config.dev.assetsPublicPath,  
    filename: '[name].js'  
  },  
  // 配置模块如何被解析，就是import或者require的一些配置  
  resolve: {  
    // 当使用require或者import的时候，自动补全下面的扩展名文件的扩展名，也就是说引入的时候不需要使用扩展名  
    extensions: ['', '.js', '.vue', '.json'],  
    // 当我们require的东西找不到的时候，可以去node_modules里面去找，  
    fallback: [path.join(__dirname, '../node_modules')],  
    // 别名,在我们require的时候，可以使用这些别名，来缩短我们需要的路径的长度  
    alias: {  
      'vue$': 'vue/dist/vue.common.js',  
      'src': path.resolve(__dirname, '../src'),  
      'assets': path.resolve(__dirname, '../src/assets'),  
      'components': path.resolve(__dirname, '../src/components')  
    }  
  },  
  // 同上  
  resolveLoader: {  
    fallback: [path.join(__dirname, '../node_modules')]  
  },  
  // 对相应文件的编译使用什么工具的配置  
  module: {  
    // loader之前的配置，会对.vue,.js的文件用eslint进行编译,include是包含的文件，exclude是排除的文件，可以使用的正则  
    preLoaders: [  
      {  
        test: /\.vue$/,  
        loader: 'eslint',  
        include: [  
          path.join(projectRoot, 'src')  
        ],  
        exclude: /node_modules/  
      },  
      {  
        test: /\.js$/,  
        loader: 'eslint',  
        include: [  
          path.join(projectRoot, 'src')  
        ],  
        exclude: /node_modules/  
      }  
    ],  
    // 这里也是相应的配置，test就是匹配文件，loader是加载器，  
    // query比较特殊，当大小超过10kb的时候，会单独生成一个文件，文件名的生成规则是utils提供的方法，当小于10kb的时候，就会生成一个base64串放入js文件中  
    loaders: [  
      {  
        test: /\.vue$/,  
        loader: 'vue'  
      },  
      {  
        test: /\.js$/,  
        loader: 'babel',  
        include: [  
          path.join(projectRoot, 'src')  
        ],  
        exclude: /node_modules/  
      },  
      {  
        test: /\.json$/,  
        loader: 'json'  
      },  
      {  
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,  
        loader: 'url',  
        query: {  
          limit: 10000,  
          name: utils.assetsPath('img/[name].[hash:7].[ext]')  
        }  
      },  
      {  
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,  
        loader: 'url',  
        query: {  
          limit: 10000,  
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')  
        }  
      }  
    ]  
  },  
  // eslint的配置  
  eslint: {  
    formatter: require('eslint-friendly-formatter')  
  },  
  // vue-loader的配置  
  vue: {  
    loaders: utils.cssLoaders({ sourceMap: useCssSourceMap }),  
    postcss: [  
      require('autoprefixer')({  
        browsers: ['last 2 versions']  
      })  
    ]  
  }  
}  
```


#### 2. webpack.dev.conf.js

```javascript
var config = require('../config')  
var webpack = require('webpack')  
// https://github.com/survivejs/webpack-merge 提供一个合并生成新对象函数  
var merge = require('webpack-merge')  
var utils = require('./utils')  
var baseWebpackConfig = require('./webpack.base.conf')  
var HtmlWebpackPlugin = require('html-webpack-plugin')  
var FriendlyErrors = require('friendly-errors-webpack-plugin')  
  
// 在浏览器不刷新的情况下，也可以看到改变的效果，如果刷新失败了，他就会自动刷新页面  
Object.keys(baseWebpackConfig.entry).forEach(function (name) {  
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])  
})  
  
module.exports = merge(baseWebpackConfig, {  
  module: {  
    // 后面会有对utils的解释,这里是对单独的css文件，用相应的css加载器来解析  
    loaders: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })  
  },  
  // 在开发模式下，可以在webpack下面找到js文件，在f12的时候，  
  devtool: '#eval-source-map',  
  // 将webpack的插件放入  
  plugins: [  
    // 通过插件修改定义的变量  
    new webpack.DefinePlugin({  
      'process.env': config.dev.env  
    }),  
    // webpack优化的这个一个模块，https://github.com/glenjamin/webpack-hot-middleware#installation--usage  
    new webpack.optimize.OccurrenceOrderPlugin(),  
    // 热加载  
    new webpack.HotModuleReplacementPlugin(),  
    // 当编译出现错误的时候，会跳过这部分代码  
    new webpack.NoErrorsPlugin(),  
    // filename生成的文件名，template是模版用的文件名,https://github.com/ampedandwired/html-webpack-plugin  
    new HtmlWebpackPlugin({  
      filename: 'index.html',  
      template: 'index.html',  
      // 让打包生成的html文件中css和js就默认添加到html里面，css就添加到head里面，js就添加到body里面  
      inject: true  
    }),  
    new FriendlyErrors()  
  ]  
})  

```
