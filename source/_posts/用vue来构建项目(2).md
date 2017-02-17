---
title: 用vue来构建项目2
date: 2016-9-14
category: web前端
tags: vue
---

## vue入口

首先介绍一个webpack 插件： html-webpack-plugin
在插件中可以进行一系列的配置，支持如下的配置信息

> * title: 用来生成页面的 title 元素
> * filename: 输出的 HTML 文件名，默认是 index.html, 也可以直接配置带有子目录。
> * template: 模板文件路径，支持加载器，比如 html!./index.html
> * inject: true | 'head' | 'body' | false  ,注入所有的资源到特定的 template 或者 templateContent 中，如果设置为 true 或者 body，所有的 javascript 资源将被放置到 body 元素的底部，'head' 将放置到 head 元素中。
> * favicon: 添加特定的 favicon 路径到输出的 HTML 文件中。
> * minify: {} | false , 传递 html-minifier 选项给 minify 输出
> * hash: true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS 文件，对于解除 cache 很有用。
> * cache: true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
> * showErrors: true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
> * chunks: 允许只添加某些块 (比如，仅仅 unit test 块)
> * chunksSortMode: 允许控制块在添加到页面之前的排序方式，支持的值：'none' | 'default' | {function}-default:'auto'
> * excludeChunks: 允许跳过某些块，(比如，跳过单元测试的块) 
下面的示例演示了如何使用这些配置。

下面就是编写自定义模板
```javascript
plugins: [
  new HtmlWebpackPlugin({
    title: 'Custom template',
    template: 'my-index.html', // 加载一个自定义的html 
    inject: 'body' // 插入所以的脚步到body中
  })
]
```

在这个project中就是用了这个插件，将所有的脚步加载到页面中的。

## vue-router

用 Vue.js + vue-router 创建单页应用，是非常简单的。使用 Vue.js 时，我们就已经把组件组合成一个应用了，当你要把 vue-router 加进来，只需要配置组件和路由映射，然后告诉 vue-router 在哪里渲染它们。

```javascript
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

下面做一个简单的demo，包含了vue-router的学习内容。