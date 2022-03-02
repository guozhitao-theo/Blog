---
title: 'ref,computed'
date: 2022-03-02 22:27:44
tags: javascript基础
keywords: Vue3
categories: 笔记
---
## ref 函数

原理：`ref`使用的时候都是使用`.value` 。 实际上在`reactive` 的基础上`ref` 比较简单。ref是对value 属性的 拦截实现的，如果是引用类型则使用`reactive`。

### jest

```javascript
import {ref} from '../ref'
import {effect} from '../effect'

describe('ref测试响应式', () => {
  test('ref 基本类型', () => {
    // 创建响应式数据
    let value = ref(0)
    let val
    effect(() => {
      val = value
    })
    expect(val.value).toBe(0)
    val.value++
    expect(val.value).toBe(1)
    val.value = 10
    expect(val.value).toBe(10)
  })
  test('ref 引用类型', () => {
    // 创建引用响应式数据
    let value = ref({a:0})
    let val,val1
    effect(() => {
      val = value.value
    })
    effect(() => {
      val1 = value.value.a
    })
    expect(val.a).toBe(0)
    val.a++
    expect(val.a).toBe(1)
    val.a = 10
    expect(val.a).toBe(10)
    expect(val1).toBe(10)
  })
  test('ref 引用类型 Ref实例', () => {
    let value1 = ref({a:0})
    let value = ref(value1)
    let val,val1
    effect(() => {
      val = value.value
    })
    effect(() => {
      val1 = value.value.a
    })
    expect(val.a).toBe(0)
    val.a++
    expect(val.a).toBe(1)
    val.a = 10
    expect(val.a).toBe(10)
    expect(val1).toBe(10)
  })
})

```

### ref 实现

```javascript
import { isObject } from '../utils'
import { reactive } from './reactive'
import { track, trigger } from './effect'

export function ref (value) {
  // 不用拦截所有的对象中的属性，只需要拦截value
  // 判断这个值是否是RefImpl实例
  if(value && value.__ref) {
    return value
  }

  class RefImpl {
    constructor (val) {
      this.__ref = true
      // 判断value是引用类型则返回响应式数据
      this._val = convert(val)
    }
    // 拦截value 属性
    get value () {
      // 依赖收集
      track(this, 'value')
      return this._val
    }

    // 拦截set属性
    set value (val) {
      // 判断当前的值是否有更改
      if(this._val !== val) {
        // 更改之后要将当前的_val 改成响应式
        this._val = convert(val)
        // 触发依赖
        trigger(this, 'value')
      }
      return this._val
    }
  }
  // 返回实例
  return new RefImpl(value)
}

// 引用对象返回 reactive处理之后的响应式数据
function convert (val) {
  return isObject(val) ? reactive(val) : val
}
// 判断是否是引用对象
function isObject(val){
  return typeof val === "object" && val !== null
}


```

## computed

### 原理

​		当我们使用`computed` 的时候 实际上会返回一个响应式的值。而`computed` 的值要变成响应式要求函数，或者对象的get和set 中的代码，至少有一个响应式数据，因为这样才能触发`effect()` 函数调用来产生副作用。所以`computed`的返回值也应该是`effect` 函数产生的副作用生成的值。

​		想一想 `effect` 函数在什么时候调用的呢？在响应式数据触发`get`的时候, 主动的去触发执行 `effect` 的函数参数 从而实现数据更新。所以在实现 `computed` 的时候至少有两个点要实现：

> 1. `computed` 的返回值是响应式的，也就是说`computed` 的返回值使用的时候是要通过 `get` 函数 触发执行 `effect` 实现响应式。
> 2. `computed` 函数参数中的响应式数据必须要被`effect`函数包裹起来。这样才能触发数据更改。

​		倒着推回去看，`computed` 函数的参数是一个`function` 或者 一个 包含`get` 、`set` 方法的对象。这一点非常巧妙的就实现了前面两点。

* 如果数`function` 刚好作为 `getter` 将计算结果作为`computed`的返回值， 并根据上面的第2点，将包含响应式的数据放在`effect` 函数中。

  如此之后有一个问题：之前的`effect`函数都是立即执行的，这样我们怎么去拿到计算结果呢？并作为`computed` 的返回值呢。

  为了解决这个问题，将`effect` 的调用时机进行可调控的，让它在`computed` 的 `get` 函数触发的时候调用就可以了。

* 如果作为对象传入的话，也一样，只是多了一个`set` 函数。调用`set` 的时候`computed` 需要将所收集的依赖进行更新(就是 调用一次`effect`方法)才能使数据响应变化。

### jest 模拟调用

```javascript
// computed 单元测试
import { ref } from '../ref'
import {reactive} from '../reactive'
import {computed} from '../computed'

describe('computed测试', () => {
  it('computed 基本使用', () => {
    // 创建响应式对象
    const ret = reactive({count: 1})
    const num = ref(2)
		// 调用computed 传入一个 function
    let sum = computed(() => num.value + ret.count)
    // 判断返回值是否符合预期
    expect(sum.value).toBe(3)
    ret.count ++
    expect(sum.value).toBe(4)
    num.value = 10
    expect(sum.value).toBe(12)
    sum.value = 10
  })
  it('computed 对象基本使用', () => {
    const ret = reactive({count: 1})
    const num = ref(2)
    const author = ref('小明')
    const title = ref('三国')
    // 调用computed 传入一个 包含get 属性和 set 属性的对象 
    let info = computed({
      get () {
        return author.value + '-' + title.value
      },
      set (val) {
        [author.value, title.value] = val.split('-')
      }
    })
    // 判断返回值是否符合预期
    expect(info.value).toBe('小明-三国')
    author.value = '小李'
    title.value = '绿楼'
    expect(info.value).toBe('小李-绿楼')
    info.value = '小红-东游'
    expect(author.value).toBe('小红')
    expect(title.value).toBe('东游')
  })

})

```

### computed实现

```javascript
import { effect, track } from './effect'

export function computed (getterOrOptions) {
  // getterOrOptions 可以是函数，也可以是一个对象，支持get 和set
  // 当值是函数的时候不允许直接对其结果进行更改；当getterOrOptions是对象的时候通过get 和 set 进行设置
  let getter,setter
  if(typeof getterOrOptions === 'function') {
    getter = getterOrOptions
    setter = () => {
      console.warn('计算属性不允许直接更改')
    }
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  return new ComputedRefImpl(getter, setter)

}
// 拦截计算属性的 value
class ComputedRefImpl {
  constructor (getter, setter) {
    // 实例化的时候直接将getter 使用effect 包住收集依赖，当里面的值变化的时候就直接触发
    this.effect = effect(getter, {lazy: true}) // effect调用的时候不调用回调函数，而是将它存起来，当触发get 的时候再调用
    // 保存setter
    this._setter = setter
  }
  get value () {
    // 依赖收集
    track(this, 'value')
    return this.effect()
  }

  set value (val) {
    this._setter(val)
  }
}

```

