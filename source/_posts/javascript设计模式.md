---
title: javascript设计模式
date: 2022-03-02 22:26:48
tags: javascript基础
keywords: 设计模式
categories: 笔记
---
# javascript 设计模式

## 工厂模式
### 简单工厂

​		概念：将创建对象的过程单独封装

​		场景：构造函数

​		构造器例子：员工信息生成

```javascript
function User(name , age, career) {
  this.name = name
  this.age = age
  this.career = career 
}
const user = new User(name, age, career)
```

​		简单工厂例子：不同工种分配职责说明

```javascript
function User(name , age, career, work) {
    this.name = name
    this.age = age
    this.career = career 
    this.work = work
}

function Factory(name, age, career) {
    let work
    switch(career) {
        case 'coder':
            work =  ['写代码','写系分', '修Bug'] 
            break
        case 'product manager':
            work = ['订会议室', '写PRD', '催更']
            break
        case 'boss':
            work = ['喝茶', '看报', '见客户']
        case 'xxx':
            // 其它工种的职责分配
            ...
    }        
    return new User(name, age, career, work)
}
const user = new Factory(name, age, career)
```

### 抽象工厂

开放封闭原则： 对扩展开放，对修改封闭。软件实体（类、模块、函数）可以扩展，但是不可以修改。

>创建一个手机生产流水线
>
>1. 手机需要操作系统和硬件
>2. 操作系统- android 和 ios; 硬件系统-高通和小米

创建手机工厂

​		手机抽象工厂

```javascript
class MobilePhoneFactory {
  // 提供操作系统的接口
  createOS() {
    throw new Error("抽象工厂方法不允许直接调用，你需要将我重写！")
  }
  // 提供硬件接口
  createHardWare() {
    throw new Error("抽象工厂方法不允许直接调用，你需要将我重写！")
  }
}
```

​		操作系统抽象工厂

```javascript
// 定义操作系统这个类产品的抽象类
class OS {
  controlHardWare() {
    throw new Error('抽象产品方法不允许直接调用，你需要将我重写！')
  }
}
```

​		硬件系统抽象工厂

```javascript
// 定义手机硬件这类产品的抽象产品类
class HardWare {
  // 手机硬件的共性方法
  operateByOrder() {
    throw new Error('抽象产品方法不允许直接调用，你需要将我重写！')
  }
}
```

手机流水线必须先有具体的**操作系统**和**硬件**

​		定义具体操作系统的具体产品类

```javascript
class AndroidOS extends OS {
  controlHardWare() {
    console.log('我会用安卓的方式去操作硬件')
  }
}

class AppleOS extends OS {
  controlHardWare() {
    console.log('我会用苹果的方式去操作硬件')
  }
}

```

​		定义具体硬件的具体产品类

```javascript
class QualcommHardWare extends HardWare {
  operateByOrder() {
  	console.log('我会用高通的方式去运转')
  }
}

class MiWare extends HardWare {
  operateByOrder() {
    console.log('我会用小米的方式去运转')
  }
}
```

手机流水线具体类

```javascript
// 某一类的手机 - 手机使用 安卓系统➕高通硬件
class FakeStartFactory extends MobilePhoneFactory {
  createOS() {
    // 提供安卓系统实例
    return new AndroidOS()
  }
  createHardWare() {
    // 提供高通硬件实例
    return new QualcommHardWare()
  }
}
```

生产手机的流程

```javascript
const myPhone = new FakeStartFactory()
// 让他拥有操作系统
const myOS = myPhone.createOS()
// 让他拥有硬件
const myHardWare = myPhone.createHardWare()
// 启动操作系统(输出‘我会用安卓的方式去操作硬件’)
myOS.controlHardWare()
// 唤醒硬件(输出‘我会用高通的方式去运转’)
myHardWare.operateByOrder()
```

假如有一天，FakeStar过气了，我们需要产出一款新机投入市场，这时候怎么办？我们是不是**不需要对抽象工厂MobilePhoneFactory做任何修改**，只需要拓展它的种类.

```javascript
class newStarFactory extends MobilePhoneFactory {
  createOS() {
    // 操作系统实现代码
  }
  createHardWare() {
    // 硬件实现代码
  }
}
```

## 单例模式

**保证一个类仅有一个实例，并提供一个访问它的全局访问点**。

构造函数实现单例模式

```javascript
class SingleDog {
  show() {
    console.log('我是一个单例对象')
  }
  // 创建静态方法获取 类的实例
  static getInstance() {
    // 判断这个类是否已经存在实例
    if(!SingleDog.instance) {
      // 若是这个实例不存在，那先创建实例
      SingleDog.instance = new SingleDog()
    }
    return SingleDog.instance
  }
}

const s1 = SingleDog.getInstance()
const s2 = SingleDog.getInstance()

// true
s1 === s2
```

闭包实现单例：外层被内存函数使用的变量会保存在内存中不会被销毁

```javascript
SingleDog.getInstance = (function () {
  // 定义自由变量instance
  let instance = null
  return function () {
    // 判断自由变量是否为null
    if(!instance) {
      // 如果为null则new出的唯一实例
      instance = new SingleDog()
    }
    return instance
  }
})()
```

应用实例：vuex， 封装单实例方法，全局弹窗等...

## 装饰器模式

在不改变原对象的基础上，通过对其进行包装拓展，使原有对象可以满足用户的更复杂需求

ES7 中的装饰器写法

```javascript
// 类装饰器
function classDecorator(target) {
  target.hasDecorator = true
  return target
}

// 类中的函数装饰器
function funcDecorator(target, name,descriptor) {
  let originalMethod = descriptor.value
  descriptor.value = function() {
    console.log('我是Func的装饰器逻辑')
    return originalMethod.apply(this, arguments)
  }
  return descriptor
}
// 将装饰器安装到类上
@classDecorator
class Button {
  @funcDecorator
  onClick() { 
    console.log('我是Func的原有逻辑')
  }
}
// 验证装饰器是否生效
console.log(Button.hasDecorator)
button.onClick()
```

语法糖的背后

1. 函数传参

    1. 类装饰器

       `target` 就是被装饰的类本身

    2. 类方法装饰器

       target 变成了`Button.prototype`，即类的原型对象。类中的方法依附于实例，修饰方法就是修饰它的实例。但是装饰器函数执行的时候，类的实例还并不存在。为了确保实例生成后可以顺利调用被装饰好的方法，装饰器只能去修饰类的原型对象。

2. 将“属性描述对象”给装饰器

   编写类装饰器时，一般一个`target` 参数就够了。但是在编写方法装饰器时，往往需要三个参数。

    * target
    * name: 修饰的目标属性属性名
    * descriptor: 属性描述对象
        * 数据描述符：包括 value（存放属性值，默认为默认为 undefined）、writable（表示属性值是否可改变，默认为true）、enumerable（表示属性是否可枚举，默认为 true）、configurable（属性是否可配置，默认为true）。
        * 存取描述符：包括 `get` 方法（访问属性时调用的方法，默认为 undefined），`set`（设置属性时调用的方法，默认为 undefined ）

应用： React中的装饰器：HOC

## 适配器模式

**把一个类的接口变换成客户端所期待的另一种接口**，可以帮我们解决**不兼容**的问题。

应用： axios请求-node环境和浏览器环境



## 代理模式

在某些情况下，出于种种考虑/限制，一个对象**不能直接访问**另一个对象，需要一个**第三者**（代理）牵线搭桥从而间接达到访问目的。

ES6 Proxy

应用实践：

1. 事件代理

   需求： 鼠标点击列表的时候都弹出弹窗

   实现：

   	1. 每个元素绑定点击事件
   	2. 根据事件冒泡，给列表父元素绑定事件 再识别目标子元素（很大程度提高代码性能

2. 虚拟代理

   图片预加载

```javascript
   class PreLoadImage {
     constructor(imgNode) {
       // 获取真实的DOM节点
       this.imgNode = imgNode
     }
   
     // 操作img节点的src属性
     setSrc(imgUrl) {
       this.imgNode.src = imgUrl
     }
   }
   
   class ProxyImage {
     // 占位图的url地址
     static LOADING_URL = 'xxxxxx'
   
   	constructor(targetImage) {
     // 目标Image，即PreLoadImage实例
     	this.targetImage = targetImage
   	}
   
   // 该方法主要操作虚拟Image，完成加载
   	setSrc(targetUrl) {
       // 真实img节点初始化时展示的是一个占位图
       this.targetImage.setSrc(ProxyImage.LOADING_URL)
       // 创建一个帮我们加载图片的虚拟Image实例
       const virtualImage = new Image()
       // 监听目标图片加载的情况，完成时再将DOM上的真实img节点的src属性设置为目标图片的url
       virtualImage.onload = () => {
         this.targetImage.setSrc(targetUrl)
       }
     	// 设置src属性，虚拟Image实例开始加载图片
     	virtualImage.src = targetUrl
   	}
   }
```

在这个实例中，`virtualImage` 这个对象是一个“幕后英雄”，它始终存在于 JavaScript 世界中、代替真实 DOM 发起了图片加载请求、完成了图片加载工作，却从未在渲染层面抛头露面。因此这种模式被称为“虚拟代理”模式。

3. 缓存代理

   参数求和

```javascript
    // addAll方法会对你传入的所有参数做求和操作
    const addAll = function() {
      console.log('进行了一次新计算')
      let result = 0
      const len = arguments.length
      for(let i = 0; i < len; i++) {
        result += arguments[i]
      }
      return result
    }
    
    // 为求和方法创建代理
    const proxyAddAll = (function(){
      // 求和结果的缓存池
      const resultCache = {}
      return function() {
        // 将入参转化为一个唯一的入参字符串
        const args = Array.prototype.join.call(arguments, ',')
    
        // 检查本次入参是否有对应的计算结果
        if(args in resultCache) {
          // 如果有，则返回缓存池里现成的结果
          return resultCache[args]
        }
        return resultCache[args] = addAll(...arguments)
      }
    })()
```

## 策略模式

> 定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

## 状态模式

> 允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

> 状态模式主要解决的是当控制一个对象状态的条件表达式过于复杂时的情况。把状态的判断逻辑转移到表示不同状态的一系列类中，可以把复杂的判断逻辑简化。

## 观察者模式

> 观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个目标对象，当这个目标对象的状态发生变化时，会通知所有观察者对象，使它们能够自动更新。 —— Graphic Design Patterns


