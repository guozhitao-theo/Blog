---
title: webpack
date: 2021-03-31 07:52:03
tags: webpack
keywords: webpack|loader|plugins
categories: 笔记
cover: true
---

[toc]

# webpack

[项目地址](https://gitee.com/guozhitao/webpack-study)

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
  
  ### webpack 打包图片
  
  * 在js中创建图片来引入
  * 在css中引入 background（“url”）
  * \<img src="url" / >
  * file-loader 默认在内部生成一些文件在build目录下，把生成的文件的名字（路径）返回
  * html-withimg-loader 解析html编译img（这个loader 会与file-loader 、url-loader冲突(生成的是defalut对象) 需要在file-loader 设置 options->esModule: false）
  * url-loader 可以做限制，当图片小于多少k的时候 用base64转化 否则用file-loader 产生真实的文件
  
  ### 打包分类
  
   filename 加个上级目录 
  
  cdn 加个publicPath 