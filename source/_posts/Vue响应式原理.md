---
title: Vue响应式原理
date: 2022-03-02 22:28:35
tags:
---
# VUE 响应式原理

##  核心：proxy、effact

### proxy

```javascript
// proxy 浅层代理
let handler = {
  get (target, property, receiver) {
    let res = Reflect.get(target, property, receiver)
    console.log(`proxy -- target: ${JSON.stringify(target)} property: ${property}  get: ${JSON.stringify(res)}`)
    return res
  },
  set (target, property, value, receiver) {
    let res = Reflect.set(target, property, value, receiver)
    console.log(`proxy target: ${JSON.stringify(target)} value: ${value} property: ${property} -- set: ${JSON.stringify(res)}`)
    return res
  }
}

let obj = {
  a: 1,
  b: {
    c: 2
  }
}
const objProxy = new Proxy(obj,handler)

const arr = [1]
const arrProxy = new Proxy(arr, handler)
```

```javascript
// 使用递归深层代理vue
function isObject(val){
  return typeof val === "object" && val !== null
}
function myProxy (target) {
  console.log(target)
  let handler = {
    get (target, property, receiver) {
      let res = Reflect.get(target, property, receiver)
      console.log(`proxy -- target: ${JSON.stringify(target)} property: ${property}  get: ${JSON.stringify(res)}`)
      // 判断获取的值是否是对象，如果是对象也要让它的get 和 set 也被拦截
      if(isObject(res)) {
        return myProxy(res)
      }
      return res
    },
    set (target, property, value, receiver) {
      let res = Reflect.set(target, property, value, receiver)
      console.log(`proxy target: ${JSON.stringify(target)} value: ${value} property: ${property} -- set: ${JSON.stringify(res)}`)
      return res
    }
  }
  return new Proxy(target,handler)
}
```

### effect（观察者）

作用：

1. 生成依赖的标志-通过全局变量和回调函数的方式获取需要保存的依赖
    * 将回调函数保存到全局变量中，当触发响应式数据的get 方法的时候和 将该回调保存到 **依赖集合**
    * 使用回调函数的方式目的时候方便，响应式数据更改触发set方法时，将依赖集合中的 方法重新触发一遍 使数据更新。

```javascript
function effect(fn, options = {}) {
    // effect 通过队列管理
    const effectFn = () => {
        try {
            activeEffect = effectFn
            // fn 执行的时候，内部读取响应式数据的时候，就能在get配置里读取到activeEffect
            return fn()
        } finally {
            activeEffect = null
        }
    }
    // 立即调用更新（实际会有调度控制）
    effectFn()
}
```



使用一个单元测试示例说明：

```javascript
// 生成proxy代理过的数据-响应式数据
function reactive() {}
// 副作用函数，触发更新
function effect() {} 
// 测试
describe('测试响应式', () => {
    test('reactive基本使用', () => {
        const ret = reactive({ num: 0 })
        let val
        effect(() => {
            val = ret.num
        })
        expect(val).toBe(0)
        ret.num++
        expect(val).toBe(1)
        ret.num = 10
        expect(val).toBe(10)
    })
})
```

以上代码执行步骤

	1. 创建对象的响应式数据`ret`。
 	2. 调用effect 函数传入回调函数，此时内部使用了`ret` 会触发`reactive` 函数中 `get` 收集依赖。
 	3. 收集的依赖 类似`{dep: () => {val = ret.num}}`（简单举个例子，实际并非这样）。
 	4. 当执行`ret.num++` 时候 触发 `set`方法， `ret.num =ret.num + 1 ` 触发依赖集合中的方法。
 	5. 此时 `dep` 方法被调用, `val = ret.num` 又执行一次， `val`的值就变成了 + 1 的值。
 	6. `ret.num = 10` 同理

##  依赖收集 - 怎么存依赖、怎么收集

### 怎么存依赖

1. 全局保存一个`weakMap`。
2. 将响应式数据对象作为 `weakMap`的键，一个`Map` 作为值。
3. 在`Map` 中，将响应式数据对象的 属性（property） 作为键， `Set` 作为值。
4. 当某一个响应式数据被使用的时候，将`effect` 中的回调函数通过全局变量获取到之后 保存到 `Set` 之中。

### 怎么收集依赖

在proxy 中的get方法被触发的时候调用`track` 方法。

```javascript
let activeEffect = null
const targetMap = new WeakMap()

// 依赖收集
function track(target, key) {
    console.log(`触发 track -> target: ${target} key: ${key}`)
    // 基于target找到对应的dep
    let depsMap = targetMap.get(target)
    if(!depsMap) {
        // 如果是第一次，需要初始化依赖项
        targetMap.set(target, (depsMap = new Map()))
    }
    // 找到当前响应式对象key所有的依赖集合（Set）
    let deps = depsMap.get(key)
    if(!deps) {
        // 若不存在则初始化
        deps = new Set()
    }
    // 全局变量-用于保存依赖
    if(!deps.has(activeEffect) && activeEffect) {
        // 防止重复注册- 如果activeEffect 存在并且 依赖集合中不存在则注册依赖
        deps.add(activeEffect)
    }
    // 依赖集合更新
    depsMap.set(key, deps)
}
```



## 依赖触发更新

1. 响应式数据对象`set`方法被调用的时候，触发依赖更新-（将依赖集合中的方法全执行一遍）。

```javascript
function trigger (target, key) {
  // 从全局变量中找到依赖
  let depsMap = targetMap.get(target)
  if(!depsMap) {
    return
  }
  // 找到对应 属性的 依赖Set
  let deps = depsMap.get(key)
  if(!deps) {
    return
  }
  // 遍历Set
  deps.forEach(effectFn => {
    // 调用effect 方法更新数据
  	effectFn()
  });
}
```




## 知识点

1.[ES6](https://es6.ruanyifeng.com/#README)

