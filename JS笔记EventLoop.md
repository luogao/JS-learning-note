JS Event Loop 笔记

## 最前

**感谢大佬[Pomy](https://github.com/dwqs)分享的[《从一道题浅说 JavaScript 的事件循环》](https://github.com/dwqs/blog/issues/61)**

ps:此笔记内容基本于大佬分享的专题一样，仅供个人记录笔记

**侵删**




``` javascript
new Promise(resolve =>{
    resolve(1)
    Promise.resolve().then(() => console.log(2))
    console.log(4)
}).then(t => console.log(t))
console.log(3)

// output:
// 4
// 3
// 2
// 1
```



## 事件循环

- **js 是单线程** 同一时间只能做一件事
- 为了防止主线程不阻塞，产生了Event Loop 方案



## 任务队列

> 一个Event Loop 中可以有一个或者多个任务队列，一个任务队列便是一系列有序任务的集合；根据每个任务的任务源不同放入不同的队列中。相同任务源放入相同队列

在事件循坏中，每进行一次循环操作称为tick，关键步骤：

- 在此次tick中选择最先进入队列的任务（oldest task），如果有则执行（一次）
- 检查是否存在Microtasks，如果存在则不停的执行，直到清空Microtasks Queue
- 更新render
- 主线程重复执行上述步骤

**异步任务可以分为`task`和`microtask`两类**

- (macro)task主要包含：
  - script(整体代码)
  - setTimeout
  - setInterval
  - I/O
  - UI交互事件
  - postMessage
  - MessageChannel
  - setImmediate(Node.js 环境)
- microtask主要包含：
  - Promise.then
  - MutaionObserver
  - process.nextTick(Node.js 环境)

> 在 Node 中，会优先清空 next tick queue，即通过process.nextTick 注册的函数，再清空 other queue，常见的如Promise；此外，timers(setTimeout/setInterval) 会优先于 setImmediate 执行，因为前者在 `timer` 阶段执行，后者在 `check` 阶段执行

![image-20180408105754608](/var/folders/qj/nsqr8py16t93kksckqdqrrc00000gn/T/abnerworks.Typora/image-20180408105754608.png)

## 示例

``` javascript
console.log('script start');

setTimeout(function() {
  console.log('timeout1');
}, 10);

new Promise(resolve => {
    console.log('promise1');
    resolve();
    setTimeout(() => console.log('timeout2'), 10);
}).then(function() {
    console.log('then1')
})

console.log('script end');
```

### 分析

- 事件循环从宏任务开始，因只有一个script即整体代码，当遇到任务源时则会分发任务到对应的任务队列中
- 遇到`console.log`，直接输出。接着继续往下执行，遇到`setTimeout`作为一个宏任务源，则会先将其分发到对应的队列中
- 遇到`Promise`实例执行构造函数参数中的代码，后续的`.then`则分发到microtask的`Promise`队列中去。所以先输出`promise1`，然后执行`resolve`
- 构造函数继续往下执行，碰到`setTimeout`,则将对应的任务分配到对应的队列中
- script任务继续往下执行，最后输出一句`script end`至此，全局任务执行完毕

综上，每次执行完一个宏任务之后，会去检查是否存在Microtasks；如果有，则执行Microtasks直至清空Microtasks Queue

于是，script任务执行完毕，开始清空微任务队列。此时只有一个`then`任务，因此直接执行。当所有的`microtast`执行完毕之后，表示第一轮循环就结束了。

于是开始第二轮循环，循环从宏任务开始，此时宏任务有两个：`timeout1` 和 `timeout2`

执行完`timeout1` ，此时微任务队列中没有可以执行的任务，直接开始第三轮循环

第三轮循环从宏任务开始，即只有剩下的`timeout2`，取出输出即可

至此，宏任务和微任务队列中都没有任务了，代码不会再输出其他东西。所以示例的结果如下：

```javascript
script start
promise1
script end
then1
timeout1
timeout2
```



## 最后

至此JS Event Loop笔记完成。



## 参考资料

[《从一道题浅说 JavaScript 的事件循环》](https://github.com/dwqs/blog/issues/61)