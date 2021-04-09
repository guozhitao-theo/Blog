---
title: webpack第二期
date: 2021-04-09 08:40:34
tags: webpack
keywords: webpack|loader|plugins
categories: 笔记
cover: true
---

[toc]
# [webpack-dev-2](https://gitee.com/guozhitao/webpack-dev-2)

## 配置多页面打包

* 配置多入口

  ```javascript
  entry: {
      home: './src/index.js',
      other: './src/other.js'
    }
  ```

* 配置多出口

  ```javascript
  output: {
      // [name] home, other
      filename: '[name].js',
      path: path.resolve(__dirname, 'dist')
    }
  ```

* 配置多模板

  ```javascript
  new HtmlWebpackPlugin({
      template: './index.html',
      filename: 'home.html',
      chunks: ['home']
  }),
      new HtmlWebpackPlugin({
      template: './index.html',
      filename: 'other.html',
      chunks: ['other', 'home']
  })
  ```

## 配置source-map

可能在解析js 的过程中会把高级语法解析成低级语法（@babel/core @babel/preset-env babel-loader ）

[webpack: devtool](https://webpack.js.org/configuration/devtool/)

* devtool: 'source-map' 源码映射 会单独生成一个sourcemap文件，出错了会标识当前报错的列和行， 大 和 全
* devtool: 'eval-source-map' 不会产生单独的文件 但是可以显示行和列
* devtool: 'cheap-module-source-map'不会产生列 但是是一个单独的映射文件
* devtool: 'eval-cheap-module-source-map'不会产生文件 集成在打包后的文件中，也不会产生列

## watch 的用法

watch: true

watchOptions: {} // 监控的选项

```javascript
watch: true,
watchOptions: { // 监控的选项
   poll: 1000, // 每秒问我1000次
   aggregateTimeout: 500, // 防抖 我一直输入代码 500ms内我输入的只打包一次
   ignored: '/node_module'
}
```

## webpack 小插件应用

1. cleanWebpackPlugin
2. copyWebpackPlugin
3. bannerPlugin (内置)  （总是会生成一个新的文件，未找到解决方案）

## webpack跨域问题

1. 服务端接口

```javascript
devServer: {
    proxy: { // 重写的方式 把请求代理到服务器上
      "/api/**": {
        target: "http://localhost:3000/",
        pathRewrite: { "^/api": "" }
      }
    }
  }
```

2. 前端mock 数据

   ```javascript
   devServer: {
       // app 为express实例
       before (app) {
         app.get('/user', (req, res) => {
           res.json({
             name: 'gzt-before'
           })
         })
       }
   }
   ```

3. 有服务端 不用代理来处理 能不能在服务端启动webpack 端口用服务端端口
   * 引入webpack
   * 安装中间件 `webpack-dev-middleware`
   * 将`webpack.config.js`通过webpack编译之后传入中间件
   * 服务端使用中间件



## resolve 属性的配置

```javascript
resolve: { // 解析 第三方 common
    modules: [path.resolve('node_modules')], // 缩小模块查找范围
    extensions: ['.js', '.css', '.json'], // 先找js 没有 就找css 再找json
    mainFields: ['style', 'main'], // 配置入口(package.json)
    // mainFiles: [], // 入口文件的名字 默认index
    alias: { // 别名 vue.runtime
      // 'bootstrap': 'bootstrap/dist/css/bootstrap.css'
    }

  }
```

## 定义环境变量

webpack.DefinePlugin 插件 定义变量

```javascript
new webpack.DefinePlugin({
    DEV: JSON.stringify('production'),
    FLAG: 'true',
    EXPRESSION: '1 + 1'
})
```

## 区分不同的环境

1. 创建三个配置文件

   `webpack.base.js`: 公共的配置

   `webpack.dev.js`： 开发环境使用到的配置

   `webpack.prod.js`： 生产环境使用到的配置

2. 安装插件 [webpack-merge](https://www.npmjs.com/package/webpack-merge)
3. 通过 `npm run xxx -- --config webpack.xx.js` 来指定配置文件