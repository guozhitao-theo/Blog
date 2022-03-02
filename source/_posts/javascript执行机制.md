---
layout: javascript
title: 执行机制
date: 2022-03-02 22:22:34
tags: javascript基础
keywords: 执行上下文
categories: 笔记
---
# JavaScript 执行机制

[TOC]

## 变量提升

所谓的变量提升，是指在JavaScript代码执行过程中，javaScript引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”。变量被提升后，会给变量设置默认值`undefined`。
所以我们可以在定义之前使用变量或者函数。

## JavaScript代码执行流程

变量和函数声明在代码的位置是不会改变的，而且是在变异阶段被JavaScript引擎放在内存中。一段JavaScript代码在执行之前需要被JavaScript编译，编译完成之后，才会进入执行阶段。

### 编译阶段

编译过程：生成执行上下文和可执行代码。

#### 执行上下文

执行上下文是JavaScript执行一段代码时的运行环境。执行上下文中存在一个变量环境的对象（Viriable Environment），该对象中保存了变量提升的内容。编译过程先把代码的声明部分存入对象中，如变量和函数（函数是在变量环境中创建一个属性，该属性指向堆中函数所在的位置）。

**一般来说创建执行上下文有三种情况：**
1.  当JavaScript执行全局代码的时候，会编译全局代码并创建全局执行上下文，而且在整个页面周期内，执行上下文只有一份。
2.  当调用一个函数的时候，函数体内的代码会被编译，并创建函数执行上下文，一般情况下，函数执行结束之后，创建的函数执行上下文会被销毁。
3.  当使用eval函数的时候，eval的代码也会被编译，并创建执行上下文。



#### 可执行代码

JavaScript引擎会把声明之外的代码编译为字节码。

### 执行阶段

* 当执行到函数的时候，JavaScripty引擎便开始在变量环境对象中查找该函数，由于变量环境中存在该函数的引用，所以JavaScript引擎便开始执行该函数
* 当执行到变量的时候，JavaScript引擎继续在变量环境对象中查找该对象，如果存在则继续执行，不存在则`not define`

代码出现相同的变量或者函数的时候，后面的覆盖前面的。



## 调用栈

调用栈是用来管理函数调用关系的一种数据结构。

### 函数调用

```javascript
var a = 2
function add () {
  var b = 10
  return a + b
}
add()
```

函数调用过程：

1. 在执行到函数`add`之前，JavaScript 引擎会为上面这点代码创建全局执行上下文，包含了声明的函数和变量。
2. 执行上下文准备好之后，便开始执行全局代码，当执行到add这，JavaScript判断这个是一个函数调用，就会执行一下操作：
    1. 从全局执行上下文中，取出add函数代码。
    2. 对add函数的这段代码进行编译，并创建该函数的执行上下文和可执行代码。
    3. 执行代码，输出结果。

### 什么是javaScript调用栈

在执行上下文创建好之后，JavaScript引擎会讲执行上下文压入栈中，通常把这种用来管理执行上下文的栈称为执行上下文栈，又称调用栈。

```javascript
var a = 2
function add (b, c ) {
  return b + c
}
function addAll(b, c) {
  var d = 10
  result = add(b,c)
  return result
}
addAll(3, 6)
```

1. 创建全局上下文将其压入栈底部

   ```javascript
   let stack = []
   let globalContext = {
     变量环境: {
       a = undefined
     	add = function () {...}
     	addAll = function (){}
     }
     词法环境: {
       
     }
   }
   stack.push(globalContext)
   ```

2. 执行全局代码。首先执行`a=2`的赋值操作，执行该语句会讲全局上下文变量环境中a的值设置为2

   ```javascript
   let globalContext = {
     变量环境: {
       a = 2
     	add = function () {...}
     	addAll = function (){}
     },
     词法环境: {
       
     }
   }
   ```

3. 调用`addAll`函数。当调用该函数时，JavaScript引擎会编译该函数，并为其创建一个执行上下文，最后还将该函数的执行上下文压入栈中。

   ```javascript
   let addAllContext = {
     变量环境: {
       d = undefined
       result = undefinde
     },
     词法环境: {
       
     }
   }
   stack.push(addAllContext)
   ```

4. addAll函数的执行上下文创建好了之后，进入函数代码执行阶段，`d=10`

   ```javascript
   let addAllContext = {
     变量环境: {
       d = 10
       result = undefinde
     },
     词法环境: {
       
     }
   }
   ```

5. 执行到add函数调用语句时，同样为其创建执行上下文，并将其压入调用栈

   ```javascript
   let addContext = {
     变量环境: {
       
     },
     词法环境: {
       
     }
   }
   stack.push(addContext)
   ```

   **此时的执行上下文栈：**

   ```javascript
   // stack
   [
     globalContext = {
       变量环境: {
         a = 2
         add = function () {...}
         addAll = function (){}
       },
       词法环境: {
   
       }
   	},
       
     addAllContext = {
       变量环境: {
         d = 10
         result = undefinde
       },
       词法环境: {
   
       }
     },
       
     addContext = {
       变量环境: {
   
       },
       词法环境: {
   
       }
     }
   ]
   ```

6. 当add函数返回时，该函数的执行上下文就会从栈顶弹出，并将result 的值设置为add函数的返回值9

   ```javascript
   stack.pop()
   // stack
   [
     globalContext = {
       变量环境: {
         a = 2
         add = function () {...}
         addAll = function (){}
       },
       词法环境: {
   
       }
   	},
       
     addAllContext = {
       变量环境: {
         d = 10
         result = 9
       },
       词法环境: {
   
       }
     }
   ]
   ```

7. addAll执行最后一个相加操作并返回，addAll的执行上下文也会从栈顶弹出，此时调用栈中就只剩下全局上下文了。

   ```javascript
   stack.pop()
   // stack
   [
     globalContext = {
       变量环境: {
         a = 2
         add = function () {...}
         addAll = function (){}
       },
       词法环境: {
   
       }
   	}
   ]
   ```

8. 至此，整个JavaScript流程执行结束。

所以 调用栈是JavaScript引擎追踪函数执行的一个机制，当一次有多个函数被调用时，通过调用栈就能够追踪到哪个函数正在被执行以及各个函数之间的调用关系。

### 总结

1. 每个调用函数，JavaScript引擎会为其创建执行上下文，并把该执行上下文压入调用栈，然后JavaScript引擎开始执行函数代码。
2. 如果在一个函数A中调用了另外一个函数B，那么JavaScript引擎会为B函数创建执行上下文，并将B函数的执行上下文压入栈顶。
3. 当前函数执行完毕后，JavaScript引擎会将该函数的执行上下文弹出栈。
4. 当分配的调用栈空间被占满时，会引发“堆栈溢出”问题。



## 作用域

作用域是指在程序中定义变量的区域，该位置决定了变量的生命周期。通俗地理解，作用域就是变量与函数的可访问范围，即作用域控制着变量和函数的可见性和生命周期。

### ES6之前，JavaScript只支持两种作用域

* 全局作用域

  全局作用域中的对象在代码中的任何地方都能够访问，其生命周期伴随着页面的生命周期。

* 函数作用域

  函数作用域就是在函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问。函数执行结束之后，函数内部定义的变量会被销毁。

#### 变量提升的问题

由于变量无论在哪里声明，在编译阶段都会被提取到执行上下文的变量环境中，所以这些变量在整个函数体内部的任何地方都能够被访问。

1. 变量容易在不被察觉的情况下被覆盖掉。

   ```javascript
   var myname = "小明"
   function showName () {
     console.log(myname)// undefined 而非小明
     if(0) {
       var myname = "小红"
     }
     console.log(myname)// undefined 而非小明
   }
   showName()
   ```

2. 本应该销毁的变量没有被销毁。

   ```javascript
   function foo() {
     for (var i = 0; i < 7; i++) {
       
     }
     console.log(i) // 7
   }
   foo()
   // 在创建执行上下文的时候，变量i已经被提升了，所以当for循环结束之后，变量i并没有被销毁
   ```

### ES6块级作用域

为了解决变量提升带来的一系列问题，ES6引入了`let`和`const`关键字，从而使JavaScript拥有了块级作用域。

#### JavaScript 是如何支持块级作用域的

##### 1. 编译并创建执行上下文

从执行上下文中来看，在创建执行上下文的时候执行流程：

* 函数内部通过`var` 声明的变量，在编译阶段全都被存放在**变量环境**里面了。
* 通过`let `o r`const`声明的变量，在编译阶段会被存放到**词法环境**（Lexical Environment）中。
* 在函数的作用域块内部，通过`let`声明的变量并没有被存放到词法环境中。



##### 2. 执行代码

当进入作用域块的时，作用域块中通过`let`声明的变量，会被存放在词法环境的一个单独区域中，这个区域中的变量并不影响作用域块外面的变量。

词法环境内部，维护了一个小型栈结构，栈底是函数最外层变量，进入作用域块后，就会把该作用域块内部的变量压到栈顶；当作用域执行完成之后，该作用域的信息就会从栈顶弹出。（这里的变量是指通过`let`或`const`声明的变量）

当使用到变量的时候，javaScript引擎需要在词法环境和变量环境中查找被使用变量的值，具体查找方法：**沿着词法环境的栈顶向下查询，如果在词法环境中的某个块查找到了，就直接返回给JavaScript引擎，如果没有查找到，那么继续在变量环境中查找。**

块级作用域执行结束后，其内部通过`let`或`const`声明的变量就会从词法环境的栈顶弹出。



### 总结

**块级作用域就是通过词法环境的栈结构来实现的，而变量提升是通过变量环境实现，通过这两者的结合，JavaScript引擎就同时支持了变量提升和块级作用域了。**



## 作用域链和闭包

### 词法作用域

词法作用域就是值作用域是有代码中函数声明的位置来决定的。所以词法作用域是静态的作用域，通过它能够预测代码在执行过程中如何查找标识符。

**词法作用域是代码编译阶段就决定好的，和函数怎么调用的没有关系。**

### 作用域链

每个执行上下文的变量环境中，都包含了一个外部引用，用来指向外部的执行上下文，这个外部引用称作`outer`。

当一段代码使用了一个变量时，JavaScript引擎首先会在“当前的执行上下文”中查找该变量，如果当前的变量环境中没有查找到，JavaScript引擎会继续`outer`所指向的执行上下文中查找。这个查找的链条就称为**作用域链**。

在 JavaScript 执行过程中，其**作用域链是由词法作用域决定的**。

### 块级作用域中的变量查找

1. 先查找当前执行上下文词法环境栈顶
2. 栈顶没有一直往下查找
3. 词法环境中不存在则查找当前执行上下文中的变量环境
4. 变量环境中依旧不存在则根据`outer` （根据词法作用域规则）到外部执行上下文查找
5. 查找顺序重复1，2，3，4步骤。



### 闭包

**在JavaScript中，根据词法作用域的规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数用用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为“外部函数的闭包”**

```javascript
function foo() {
  var myName = "小明"
  let test1 = 1
  const test2 = 2 
  var innerBar = {
    getName: function () {
      console.log(test1)
      return myName
    },
    setName: function (newName) {
      myName = newName
    }
  }
  return innerBar
}
var bar = foo()
bar.setName('小张')
bar.getName()
console.log(bar.getName())
```

1. 创建全局执行上下文,并将全局执行上下文入栈

   ```javascript
   // stack = []
   let 全局执行上下文 = {
     变量环境: {
       bar = undefined
       function foo(){...}
   		outer = null
     },
     词法环境: {
       
     }
   }
   stack.push(全局执行上下文)
   
   ```

2. 执行代码调用foo函数，创建foo函数执行上下文

   ```javascript
   let foo函数执行上下文 = {
     变量环境: {
       myName = undefined
       innerBar = undefined
   		outer = 全局执行上下文
     },
     词法环境: {
       test1 = 1
       test2 = 2
     }
   }
   stack.push(foo函数执行上下文)
   ```

3. foo函数执行完成，其返回值赋值给bar。（根据 词法作用域的规则，内部函数getName和setname总是可以访问他们的外部函数foo中的变量，所以当innerBar对象返回给全局变量bar时，虽然函数foo已经执行结束，但是getName和setName函数依旧可以使用foo函数中的变量myName和test1）

   ```javascript
   // stack 
   [
     全局执行上下文 = {
       变量环境: {
         bar = innerbar
         function foo(){...}
         outer = null
       },
       词法环境: {
   
       }
     },
     foo(closure) = {
       myName = "小明",
       test1 = "1"
     }
   ]
   ```

4. 调用bar.setName('小红') 的时候，先创建setName函数的执行上下文，并将其入栈，随后开始执行内部代码，执行到`myName = 小红` 的时候开始查找myName 变量。

   ```javascript
   let bar_setName执行上下文 = {
     变量环境: {
       outer = 全局执行上下文
     },
     词法环境: {
       
     }
   }
   stack.push(bar_setName执行上下文)
   ```

5. **闭包作用域链查找**

   ```javascript
   // stack
   [
     全局执行上下文 = {
       变量环境: {
         bar = innerbar
         function foo(){...}
         outer = null
       },
       词法环境: {
   
       }
     },
     foo(closure) = {
       myName = "小明",
       test1 = "1"
     },
     bar_setName执行上下文 = {
       变量环境: {
         outer = 全局执行上下文
       },
       词法环境: {
   
       }
     }
   ]
   ```

   **JavaScript 引擎会沿着“当前执行上下文–>foo 函数闭包–> 全局执行上下文”的顺序来查找 myName 变量。**

   此时查找顺序如下：

    1. 查找当前执行上下文`setName 的执行上下文`
    2. 没有则查找闭包`foo(closure)`
    3. foo 函数的闭包中包含了变量 myName，所以调用 setName 时，会修改 foo 闭包中的 myName 变量的值。

6. 调用bar.getName()流程如上4。

#### 闭包的回收

* 如果引用闭包的函数是全局变量，那么闭包会一直存在知道页面关吧；如果这个闭包以后不在使用的话，就会造成内存泄漏。
* 如果引用闭包是个局部变量，等函数销毁后，在下次JavaScript引擎垃圾回收机制时，判断闭包这块内容如果已经不再被使用了，那么回收这块内存。

**如果该闭包会一直使用，那么它可以作为全局变量而存在；但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一个局部变量。**



## JavaScript中的this

作用域链和this是两套不同的系统，他们之间基本没有联系。

**this 是和执行上下文绑定的**，也就是说每个执行上下文中都有一个this，执行上下文主要分三种，this也分三种：

* 全局执行上下文中的this
* 函数中的this
* eval中的this



### 全局执行上下文中的this

全局执行上下文中的this是指向window对象的。这是this和作用域链的唯一交点，作用域链的最底端包含了window对象，全局执行上下文中的this也是指向window对象。

### 设置函数执行上下文中的this

1. 通过函数的call方法设置

   ```javascript
   let bar = {
     myName : "极客邦",
     test1 : 1
   }
   function foo(){
     this.myName = "极客时间"
   }
   foo.call(bar)
   console.log(bar)
   console.log(myName)
   ```

2. 通过对象调用方法设置

   ```javascript
   var myObj = {
     name : "极客时间", 
     showThis: function(){
       console.log(this)
     }
   }
   myObj.showThis()
   ```

   **使用对象来调用其内部的一个方法，该方法的this指向对象本身。**

    * 在全局环境中调用一个函数，函数内部的 this 指向的是全局变量 window。
    * 通过一个对象来调用其内部的一个方法，该方法的执行上下文中的 this 指向对象本身。

3. 通过构造函数中设置

   ```javascript
   function CreateObj(){
     this.name = "极客时间"
   }
   var myObj = new CreateObj()
   ```

   当执行 `new CreateObj()`的时候，JavaScript引擎做了四件事：

    1. 创建一个空对象` tempObj`
    2. 调用`CreateObj.call`方法，并将`tempObj`作为call方法的参数，这样当`CreateObj`的执行上下文创建时，它的this就指向了`tempObj`对象
    3. 执行`CreateObj`函数，此时的`CreateObj`函数执行上下文中的this指向了`tempObj`对象
    4. 返回`tempObj`对象

### this的设计缺陷以及应对方案

1. 嵌套函数中的this不会从外层函数中继承。
    1. 通过变量保存this，将this体系转为作用域体系。
    2. 使用ES6箭头函数解决这个问题。
2. 普通函数中的this默认指向全局对象window。
    1. 通过设置JavaScript的“严格模式”来解决。在严格模式下，默认执行一个函数，其函数的执行上下文中的this值是undefined。

### 总结

1. 函数作为对象的方法调用时，函数中的this就是该对象。
2. 当函数被正常调用时，在严格模式下，this值是undefined，非严格模式下this指向全局window。
3. 嵌套函数中的this不会继承外层函数的this值
