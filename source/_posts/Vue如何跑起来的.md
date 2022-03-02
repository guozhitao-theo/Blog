---
title: Vue如何跑起来的
date: 2022-03-02 22:29:31
tags: javascript基础
keywords: Vue3
categories: 笔记
---
# Vue如何跑起来的

## 入口文件说明

在vue3的入口文件`main.js`中 能看到与一下代码相似的代码

```javascript
// 引入 createApp 
import { createApp } from 'vue'
// 引用Element3 第三方库
import Element3 from 'element3'
// 引用Element3 第三方库样式
import 'element3/lib/theme-chalk/index.css'
// 引用Vue 根组件
import App from './App.vue'
// 引入 路由
import router from './router/index'
// 引入 vuex
import store from './store/index'

createApp(App).use(store).use(router).use(Element3).mount('#app')

```

## 简单的实现createApp

​		返推倒回去，知道` createApp() `传入一个`component` 返回的一个对象，对象中包含`use`，`mount`等方法。

​		vue的项目都是从一个根组件搭起来的，入口文件先将vue的根组件挂载上，然后再实际的节点中挂载到对应的节点上。

### vue 的执行流程

1. 入口文件`createApp` 调用，创建渲染器（`runtime-dom`）
2. 创建渲染器的时候将实际的客户端对节点的操作传入由`vue`渲染成为`html`节点.
    * 这一点vue能够实现跨端原因。
3. 渲染器要做两件事：
    1. `render` 渲染器完成渲染动作（创建虚拟节点，patch, 节点刷新）
    2. 将vue上的方法和配置挂载到对应的实例上（use, component, provide, directive, mount: mount的时候调用`render` 完成渲染）

```javascript
   // 返回一个包含createApp 的对象
   function ensureRenderer () {
       return (
           // createRenderer(nodeOps) 的返回值是一个创建vue实例的方法
           render || (render = createRenderer(nodeOps))
       )
   }
   
   // 创建app的方法
   export const createApp = (...args) => {
       return ensureRenderer().createApp(...args)
   }
   
   // 创建渲染器方法
   function createRenderer (options) {
       // 组件渲染动作 创建虚拟节点，patch算法, 节点刷新
       function render () {
   
       }
       return {
           // 创建vue组件api，并在其中调用渲染
           createApp: createAppAPI(render)
       }
   }
   
   
   
   // 创建vue组件api
   let uid = 0 // 全局唯一标识 每次创建一个实例 加一
   /**
    * @description 创建vue组件 内部api的方法
    * @returns 创建Vue App的方法
    */
   export function createAppAPI(render) {
       return function createApp(rootComponent) {
           // 上下文对象 -- 随便写几个意思一下
           const context = {
               provides: {},
               components: {},
               directives: {},
               plugins: new Set(), // 插件去重其他可被覆盖
               mixins: []
           }
   
           // 实例上的数据和方法
           const app = {
               _uid: uid++, // 每个vue组件的唯一标识
               _context: context, // 组件的上下文
               mount (rootContainer) {
                   // 节点挂在
                   /**
                    * 1. 创建虚拟节点
                    * 2. 给虚拟节点挂载context
                    * 3. 节点渲染
                    */
                   // 调用 render函数
                   const vnode = createVNode(rootComponent)
                   vnode.appContext = context
                   render(vnode, rootContainer)
                   return app
               },
               // 插件注册
               use(plugins, options) {
                   /**
                    * 1. 判断插件是否已经存在
                    * 2. 不存在则添加插件
                    * 3. 调用插件的install 方法 完成注册
                    */
                   if(context.plugins.has(plugins)) {
                       context.plugins.add(plugins)
                       plugins.install(app, ...options)
                   }
                   return app
               },
               // 简单的数据存储
               provide (key, val) {
                   // 将数据存储到 context 中
                   context.provides[key] = val
                   return app
               },
               // 组件注册
               component(name, component) {
                   // 注册的组件存储在context 上
                   context.components[name] = component
                   return app
               },
               // ... 其他的类似
               
           }
           return app
   
       }
   }
   
   // 创建虚拟节点
   function createVNode() {
     let vnode = null
     return vnode
   }
```



### render 渲染

`render(vnode, container)` 接受两个参数，`vnode` 是新的虚拟节点， `container`是需要更新的组件的“上下文”

vue重点就是虚拟节点渲染的时候中间的patch 环节。

```javascript
// 虚拟节点渲染函数
function render(vnode, container) {
    // 获取旧的节点
    const prevVNode = container._vnode
		// 如果新的节点是null
    if (vnode == null) {
      // 如果上一个节点存在的话
      if (preVNode) {
        unmount(preVNode) // 传递vnode是null，直接全部卸载
      }
    } else {
      // 如果新的节点存在 调用patch
      patch(container._vnode || null, vnode, container)
    }
    container._vnode = vnode // 缓存vnode，作为下次render的prev
  }


// 虚拟Dom 核心 咱先只考虑 入口文件
function patch (oldNode, newNode, container) {
  // 新老虚拟Dom对比 处理
  // 暂时不管其他情况 先处理组件类型
  processComponent(oldNode, newNode, container)
}

//处理组件
function processComponent(oldNode, newNode, container) {
  // 如果不存在旧的虚拟节点则直接挂载新的节点
  if (!oldNode) {
    // 初始化 component
    mountComponent(newNode, container)
  } else {
    // 组件更新
  }
}

//挂载组件
function mountComponent(vnode, container) {
  // 创建组件实例，其实就是个对象，包含组件的各种属性
  const instance = vnode.component = {
    vnode,
    type:vnode.type,
    props:vnode.props,
    setupState:{}, //响应式状态
    slots:{},
    ctx:{},
    emit:()=>{}
  }
  // 启动setup函数中的各种响应式数据 并且将模板渲染成html --- 至此 页面渲染完成
  setupComponent(instance)
  // 组件中setup中的 effect 执行之后就触发响应式数据更新
  setupRenderEffect(instance, container)
}
```
至此 页面渲染完成。
