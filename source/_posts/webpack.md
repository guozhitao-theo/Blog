---
title: webpack
date: 2021-03-31 07:52:03
tags:
---

[toc]

# webpack

## webpack 基础配置

### webpack 安装

- 安装本地webpack
- webpack webpack-cli

### webpack 可以进行0配置

- 打包工具 -> 输出后的结果(js模块)
- npx webpack -> 找 node_modules下webpack.cmd

- 打包（支持js的模块化）

### 手动配置webpack

- 默认配置文件的名字`webpack.config.js`

- npx webpack --config xxxx.js 手动指定配置文件

- package.json 配置脚本

  ```json
  "scripts": {
      "build": "webpack --config webpack.config.js"
    }
  ```

  #### html 插件

  - 安装静态服务 `yarn add webpack-dev-server -D` 把文件生成到内存中，不会生成实际文件（webpack4 以及以上使用 webpack serve）
  - 配置html-webdpack-plugin
  
  ### 样式处理
  
  * css-loader 解析@import 语法
  
  * style-loader 他是把css 插入到head的标签中
  
  * mini-css-extract-plugin 抽离css样式
  
  * postcss-loader autoprefixer 添加样式前缀
  
    * 注意： 
  
      创建postcss.config.js文件的时候，需要在autoprefixer插件中使用 overrideBrowsweslist属性表示支持的版本。否则不生效
  
  * optimize-css-assets-webpack-plugin 压缩css (webpack5 需要设置minimize属性为true)
  
    * 使用之后必须使用 uglifyjs-webpack-plugin 才能压缩js （mode为production 才会压缩哦）
  
  ### 转换高级js语法
  
  * bael-loader 转化 @babel/core 核心模块 @babel/preset-env 将高级语法转成低级语法
  
  ### 全局变量引入问题
  
   * expose-loader 暴露全局的loader, 内联的loader。示例：`import $ from "expose-loader?exposes=$,jQuery!jquery"; // 把jquery暴露成$和jQuery`
   * 或者配到配置文件里面
     	* pre 前面执行的loader
     	* normal 普通的loader
     	* 内联loader
     	* 后置 postloader
  	* webpack.ProvidePlugin({$:"jquery"}) 在每个模块中都注入\$符号
  
  * 模板文件直接引入cdn
  * externals 不需要打包