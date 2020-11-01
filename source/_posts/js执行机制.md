---
title: js执行机制
date: 2020-11-01 21:21:51
author: 郭治涛
tags: js
cover: true
---
@[toc]
# js的运行机制

## JavaScript为什么是单线程的?

因为现在如果有两个任务一个是删除DOM节点，一个是增加DOM节点，浏览器该如何执行？所以JavaScript是单线程

## 为什么需要异步?

如果JavaScript中不存在异步,由于它是单线程只能自上而下执行,如果上一行解析时间很长,那么下面的代码就会被阻塞，不向下执行。
 页面出来，用户看到觉得是“卡死了”，所以需要异步。

## JavaScript单线程又是如何实现异步的呢?

是通过的事件循环(event loop)实现异步的。

![avatar](https://img-blog.csdnimg.cn/20191102110300632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Njk4MTYx,size_16,color_FFFFFF,t_70)

```javascript

let data = [];
$.ajax({
    url:www.javascript.com,
    data:data,
    success:() => {
        console.log('发送成功!');
    }
})
console.log('代码执行结束');
```

- ajax进入Event Table，注册回调函数success。
- 执行console.log('代码执行结束')。
- ajax事件完成，回调函数success进入Event Queue。
- 主线程从Event Queue读取回调函数success并执行。

## setTimeout

```javascript
function calculationS(duration) {
  return `${duration/1000} s`
}
let startTime = new Date().getTime();
let endTime = null;
setTimeout(function() {
  endTime = new Date().getTime()
  console.log('setTimeout的执行时间： ' +  calculationS(endTime - startTime))
}, 300)
console.log("先执行这个")
console.log(sum(1000))
//    写一个函数 让他 在主线程多花点时间
function sum(n) {
  let sum = 0;
  for(i = 0; i<=n; i++ ) {
      for(j = 0; j < n-1; j ++) {
          for(k = 0; k < n-2; k ++) {
            sum = i  + j + sum + k 
          }
      }
  }
  return sum;
}
// 主线程的执行时间
console.log('主线程的执行时间： ' + calculationS(new Date().getTime()- startTime))
```

- setTimeout()进入Event Table并注册,计时开始。
- 执行sum函数，很慢，非常慢，计时仍在继续。
- 0.3秒到了，计时事件timeout完成，setTimeout内的代码进入event loop但是sum也太慢了吧，还没执行完，只好等着。
- sum终于执行完了，setTimeout内的代码终于从Event Queue进入了主线程执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162327599.png#pic_center)

**可以看到主线程 跟 setTimeout 的执行时间 相差仅仅0.001s，而我们的延时器setTimeout的延时设置的0.3s，由此可以看出 setTimeout在进入 Event Table的时候便开始计时，计时结束之后便被推入Event Queue中，此时如果主线程为空就会立即执行，若不为空则等待主线程任务执行完毕之后才执行**

## setInterval

setInterval会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。
```javascript
function calculationS(duration) {
  return `${duration/1000} s`
}
let startTime = new Date().getTime();
let endTime = null;
let duration = setInterval(function() {
  endTime = new Date().getTime()
  console.log('setInterval的执行时间： ' +  calculationS(endTime - startTime))
  if ((endTime - startTime) > 30 * 1000) {
    clearInterval(duration)
  }
}, 100)
console.log("先执行这个")
console.log(sum(2000))
//    写一个函数 让他 在主线程多花点时间
function sum(n) {
  let sum = 0;
  for(i = 0; i<=n; i++ ) {
      for(j = 0; j < n-1; j ++) {
          for(k = 0; k < n-2; k ++) {
            sum = i  + j + sum + k 
          }
      }
  }
  return sum;
}
// 主线程的执行时间
console.log('主线程的执行时间： ' + calculationS(new Date().getTime()- startTime))
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162344148.png#pic_center)


唯一需要注意的一点是，对于setInterval(fn,ms)来说，我们已经知道不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了，**并不是意味着同时执行两次，而是说 代码执行的时间 大于定时器的时间，所以定时器实际时间间隔会大于设定的值**。

```javascript
function calculationS(duration) {
  return `${duration/1000} s`
}
let startTime = new Date().getTime();
let endTime = null;
let duration = setInterval(function() {
  endTime = new Date().getTime()
  console.log('setInterval的执行时间： ' +  calculationS(endTime - startTime))
  if ((endTime - startTime) > 30 * 1000) {
    clearInterval(duration)
  }
}, 100)
console.log("先执行这个")
console.log(sum(2000))
//    写一个函数 让他 在主线程多花点时间
function sum(n) {
  let sum = 0;
  for(i = 0; i<=n; i++ ) {
      for(j = 0; j < n-1; j ++) {
          for(k = 0; k < n-2; k ++) {
            sum = i  + j + sum + k 
          }
      }
  }
  return sum;
}
// 主线程的执行时间
console.log('主线程的执行时间： ' + calculationS(new Date().getTime()- startTime))
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162405392.png#pic_center)




## promise、**process.nextTick(callback)**、setImmediate（）: 功能：

**process.nextTick(callback):** 在事件循环的下一次循环中调用 callback 回调函数。效果是将一个函数推迟到代码书写的下一个**同步方法执行完毕时**或**异步方法的事件回调函数开始执行时**。

**process.nextTick**方法可以在当前"执行栈"的尾部==下一次Event Loop（主线程读取"任务队列"）之前==触发回调函数。也就是说，**它指定的任务总是发生在所有异步任务之前**。

**setImmediate**方法则是在当前"任务队列"的尾部添加事件，也就是说，**它指定的任务总是在下一次Event Loop时执行**。

```javascript
new Promise(function(resolve) {
  console.log('7');
  resolve();
}).then(function() {
  console.log('8')
})
console.log(9)
```

```javascript
process.nextTick(function() {
  console.log('6');
})
new Promise(function(resolve) {
  console.log('7');
  resolve();
}).then(function() {
  console.log('8')
})
console.log(9)
```

```javascript
setTimeout(function timeout() {
  console.log('setTimeout');
}, 0)
process.nextTick(function A() {
  console.log(1);
  process.nextTick(function B(){console.log(2);});
});
```

==奇怪的**setImmediate**==

```javascript
setImmediate(function A() {
  console.log(1);
  setImmediate(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('setTimeout');
}, 0);
```

这段代码的执行结果可能是这样的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162449125.png#pic_center)


也有可能是这样的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162519120.png#pic_center)


仅为猜测： 

​	第一种情况： setImmediate 最后执行，优先执行 setTimeout

​	第二种情况：当前执行状态正在 最后一个异步任务 故 setImmediate 执行，第二轮的时候才执行 setTimeout 和 setImmediateB

**[nodeJs setImmediate 源码]**（https://cloud.tencent.com/developer/article/1404691)

## 宏任务和微任务

- macro-task(宏任务)：包括整体代码script，setTimeout，setInterval；setImmediate ()==只有一个的时候永远最后执行，多个的时候每次循环只执行一个==
- micro-task(微任务)：Promise的回调，process.nextTick() ==永远最先执行，存在多个的时候先进先出==；

**单个setImmediate：**

```javascript
setTimeout(function timeout() {
  console.log('setTimeout');
}, 0);
setImmediate(function A() {
  console.log(1);
});
process.nextTick(function A() {
  console.log(2);
});
new Promise(function(resolve) {
  console.log('3');
  resolve();
}).then(function() {
  console.log('4')
})

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162539679.png#pic_center)

1. setTimeout异步任务 进入event table
2. setImmediate 异步任务 进入 event table
3. process.nextTick 异步任务 进入 event table
4. promise 同步任务执行 console.log(3), then回调异步任务 进入event
5. 此时同步任务执行完毕，优先执行 微任务 process.nextTick 该任务被推入 event loop ，经判断 主线程为空，则进入主线程执行
6. then()回调函数 为微任务 等待 process.nextTick 结束 执行
7. 执行宏任务 setTimeout
8. setImmediate 最后执行

**多个setImmediate：**

```javascript


setTimeout(function timeout() {
  console.log('setTimeout');
  setImmediate(function() {
    console.log(9);
  });
}, 0);
setImmediate(function() {
  console.log(1);
  setImmediate(function() {
    console.log(7);
  });
});
process.nextTick(function() {
  setImmediate(function() {
    console.log(6);
  });
  console.log(2);
});
new Promise(function(resolve) {
  console.log('3');
  resolve();
}).then(function() {
  console.log('4')
})
setImmediate(function() {
  console.log(5);
});

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920162552697.png#pic_center)


1. setTimeout宏任务进入 event table 记作**setTimeout1**
2. setImmediate 微任务 进入 event table 记作**setImmediate1**
3. process.nextTick 微任务 进入 event table 记作 **process1** 
4. promise 执行console.log(3) then()回调 微任务 进入 event table记作 **promise1**
5. setImmediate 微任务 进入 event table 记作**setImmediate2**

**event table:** 

| 宏任务          | 微任务            |
| --------------- | ----------------- |
| **setTimeout1** | **setImmediate1** |
|                 | **process1**      |
|                 | **promise1**      |
|                 | **setImmediate2** |

==第一轮的执行 结果为 3 进入第二轮==

1. process.nextTick 在第一轮执行过程中已经 存在 event table了，由于它在循环开始执行，故优先执行 **process1**
   1. setImmediate 微任务进入 event table记作**setImmediate3**
   2. 执行 console.log(2)
2. 执行当前event table中的 微任务 **promise1**的then()回调函数console.log(4)
3. 执行当前 event table中的 宏任务 **setTimeout1**
   1. 执行 console.log('setTimeout')
   2. setImmediate 进入event lable 记作**setImmediate 4**
4. setImmediate最后执行 故 执行**setImmediate1**
   1. 执行console.log(1)
   2. setImmediate 进入event lable 记作 **setImmediate 5**
5. event table 中还有   **setImmediate2** 执行 console.log(5)

**event tabel:**

| 宏任务 | 微任务             |
| ------ | ------------------ |
|        | **setImmediate3**  |
|        | **setImmediate 4** |
|        | **setImmediate 5** |

==第二轮 执行结束 结果为： 2 4 setTimeout 1 5; 进入第三轮==

1. 执行 **setImmediate3** 中 console.log(6)
2. 执行 **setImmediate4** 中 console.log(9)
3. 执行 **setImmediate5** 中 console.log(7)

==第三轮 的执行结果为 6 9 7 至此 该程序执行完毕==

### 执行顺序

不同类型的任务会进入对应的Event Queue，比如setTimeout和setInterval会进入相同的Event Queue。

事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-fvuq2KG5-1600590065788)(C:\Users\郭治涛\Desktop\tassk.jpg)]

```javascript
console.log('1');
setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
  })
  new Promise(function(resolve) {
      console.log('4');
      resolve();
  }).then(function() {
      console.log('5')
  })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
  console.log('9');
  process.nextTick(function() {
      console.log('10');
  })
  new Promise(function(resolve) {
      console.log('11');
      resolve();
  }).then(function() {
      console.log('12')
  })
})
```

第一轮事件循环流程分析如下：

- 整体script作为第一个宏任务进入主线程，遇到console.log，输出1。
- 遇到setTimeout，其回调函数被分发到宏任务Event Table中。我们暂且记为**setTimeout1**。
- 遇到process.nextTick()，其回调函数被分发到微任务Event Table中。我们记为**process1**。
- 遇到Promise，new Promise直接执行，输出7。then被分发到微任务Event Table中。我们记为**then1**。
- 又遇到了setTimeout，其回调函数被分发到宏任务Event Table中，我们记为**setTimeout2**。

| 宏任务          | 微任务       |
| --------------- | ------------ |
| **setTimeout1** | **process1** |
| **setTimeout2** | **then1**    |



- 上表是第一轮事件循环宏任务结束时各Event Table的情况，此时已经输出了1和7。
- 我们发现了process1和then1两个微任务。
- 执行process1,输出6。
- 执行then1，输出8。

好了，第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮时间循环从setTimeout1宏任务开始：

- 首先输出2。接下来遇到了process.nextTick()，同样将其分发到微任务Event Table中，记为**process2**。new Promise立即执行输出4，then也分发到微任务Event Table中，记为**then2**。

| 宏任务          | 微任务       |
| --------------- | ------------ |
| **setTimeout2** | **process2** |
|                 | **then2**    |



- 第二轮事件循环宏任务结束，我们发现有process2和then2两个微任务可以执行。
- 输出3。
- 输出5。
- 第二轮事件循环结束，第二轮输出2，4，3，5。
- 第三轮事件循环开始，此时只剩setTimeout2了，执行。
- 直接输出9。
- 将process.nextTick()分发到微任务Event Table中。记为**process3**。
- 直接执行new Promise，输出11。
- 将then分发到微任务Event Table中，记为**then3**。

| 宏任务 | 微任务       |
| ------ | ------------ |
|        | **process3** |
|        | **then3**    |



- 第三轮事件循环宏任务执行结束，执行两个微任务process3和then3。
- 输出10。
- 输出12。
- 第三轮事件循环结束，第三轮输出9，11，10，12。

整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。



## 总结

1. **js 为解决单线程 任务阻塞 问题 使用了同步和异步任务**

2. **同步和异步任务 通过 event loop 实现**

3. **事件循环中 异步任务 又被分为宏任务 和微任务**

4. **异步任务中的执行顺序如下：**

   ==A== : 微任务（微任务中 的 process.nextTick()总是优先执行，多个process.nextTick()按顺序执行，其余微任务按顺序执行）

   ==B==:  宏任务（宏任务中setImmediate()总是 最后执行 多个setImmediate()按顺序执行，其余宏任务按顺序执行 ）

   ```mermaid
   graph TD
      A --> B
   ```

   