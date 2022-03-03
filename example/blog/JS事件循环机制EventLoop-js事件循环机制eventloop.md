---
title: JS事件循环机制EventLoop
date: 2021-12-29 19:08:47.0
updated: 2022-01-13 17:49:39.498
categories: 
tags: 
- 前端
- JS
- 异步同步
- Eventloop
- 事件循环机制
summary: JS是一门单线程的非阻塞的脚本语言，即使在后来引入了Html5引入了Web Worker概念使其能在另外一个线程

---

### 前言
JS是一门**单线程的非阻塞**的脚本语言，即使在后来引入了Html5引入了Web Worker概念使其能在另外一个线程中执行计算密集的JS代码而不引起页面卡死，但为了保证线程安全，所有新线程都受主线程的完全控制，worker代码无法访问DOM，所以JS内部其实是利用了事件循环机制来实现的其非阻塞特性，其实这一点和操作系统的时间片轮询很相似，其本质还是一个串行系统，只是利用了分时操作让用户感觉在并发

### JS EventLoop理解
那么JS内部究竟是如何实现事件循环的呢？我翻阅了相关资料，许多人讲的也有些一知半解，最终我综合了一下这其中的思想，想尽可能简单的将其表述出来

为了方便理解我们认为JS线程内部是有三个数据结构做驱动和调度的，分别是
- **调用栈：和函数调用栈类似，每一个函数操作(帧)都在此入栈出栈**
- **消息队列**
- **微任务队列**

虽然js引擎只维护一个主线程用来解释执行JS代码，但实际上浏览器环境中还存在其他的线程，例如处理**AJAX，DOM，定时器**等，我们可以称他们为**工作线程**；同时浏览器中还维护了一个**消息队列**，主线程会将执行过程中遇到的**异步请求**发送给这个消息队列，等到主线程空闲时再来执行消息队列中的任务。


<span style="float: left">
<img src="http://leesdog.oss-cn-shanghai.aliyuncs.com/QQ%E6%88%AA%E5%9B%BE20220112185731_1641985126416.png" style='width:auto;height:300px' alt="js运行态"/>
</span>

<img src="http://leesdog.oss-cn-shanghai.aliyuncs.com/image_1641985696925.png" style='width:auto;height:300px' alt="v8引擎"/>


我们经常使用的一些API并不是js引擎中提供的，例如setTimeout。它们其实是在浏览器中提供的，也就是运行时提供的，因此，实际上除了JavaScript引擎以外，还有其他的组件。其中有个组件就是由浏览器提供的，叫Web APIs，像DOM，AJAX，setTimeout等等。

诸如promise/then，async/await创建的异步操作的**回调消息加入到微任务队列**中，微任务队列中的消息会在**主线程上调用栈无法加入新的函数(script被弹出)为空后立即执行，且在执行期间新加入的微任务也会被执行**；
主线程在执行过程中遇到了异步任务，就发起函数或者称为注册函数，通过event loop线程通知相应的工作线程（如ajax，dom，setTimout等），同时主线程继续向后执行，不会等待。等到**工作线程完成了任务(例如setTimeout计时器线程计时结束)**，eventloop线程会将**回调消息添加到消息队列**中，如果此时**主线程上调用栈无法加入新的函数（script被弹出）且为空且微任务队列为空**就执行消息队列中排在最前面的消息，依次执行。
新的消息进入队列的时候，会自动排在队列的尾端。


> 这里为了辨析两个概念，我们用一种可能并不标准的说法来区分下面两种情况，一个是**广义回调函数**，一个是**回调函数内容**
>- 举个例子
setTimeout(()=>code,0);
其中
广义回调函数是箭头函数()=>code这一个整体
回调函数内容指的是里面的code；
>- 另一个例子
let p=new Promise((res,rej)=>{
othercode
resolve(code)
});
广义回调函数是(res,rej)=>{
resolve(code)
}整个整体
而回调函数内容是只指里面的resolve(code);
如果接上p.then(function(){ 
    code
});
此时广义回调函数是function(){ 
    code
}整个整体
而回调函数内容是里面的code
>- 再举个例子
async func()
{
	await func1();
	code
}
广义回调函数指的是await func1()，而后面的code是回调函数内容

>然后我们将上述语句表述的再通俗一些，意思就是：
**JS异步操作**(promise/then、async/await)的**回调函数内容**会随着执行被排入**微任务队列**中；这里需要注意的是，**广义回调函数本身**是被压入**主调用栈**的

>**工作线程**(WebAPI)执行完后的**回调函数内容**，将会随着执行被排入**消息队列**中，这里需要注意的是，**广义回调函数本身**是被压入**主调用栈的**
当主线程调用栈为空的时候会首先立即执行微任务队列中的操作，依次出队直至队空；
当主线程调用栈和微任务队列均为空后会依次执行消息队列中的消息，依次出队直至队空；
可以这么理解，调用栈优先级>微任务队列优先级>消息队列优先级


- 针对上述描述我们举一个简单的例子
```javascript
console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(function () {
    console.log('promise1');
  })
  .then(function () {
    console.log('promise2');
  });

console.log('script end');
//输出script start, script end, promise1, promise2, setTimeout

```
过程如下：
![简单的例子](http://leesdog.oss-cn-shanghai.aliyuncs.com/image_1642066200864.png)

一些浏览器的打印的顺序是 script start, script end, setTimeout, promise1, promise2。它们在setTimeout之后运行promise回调。很可能他们调用promise回调是作为新任务的一部分，而不是作为一个微任务。

这也是可以理解的，因为promise来自 ECMAScript 而不是 HTML。ECMAScript 有“作业”的概念，类似于微任务，但是除了模糊的邮件列表讨论之外，这种关系并不明确。然而，普遍的共识是，promise应该是微任务队列的一部分并且有充足的理由。

将promise 看作任务会导致性能问题，因为回调没有必要因为任务相关的事（比如渲染）而延迟执行。它还会由于与其他任务源的交互而导致非确定性，并可能中断与其他api的交互


- 我们这里为了理解更加清晰，再举一个例子：
```JavaScript
async function async1() {
    console.log('async1 start'); // 主4
    await async2(); // 主5
    console.log('async1 end'); // 微1
}
async function async2() {
    console.log('async2'); // 主6
}

console.log('script start'); // 主1

setTimeout(function() { // 主2
    console.log('setTimeout'); // 队1
}, 0)

async1(); // 主3

new Promise(function(resolve) { // 主7
    console.log('promise1'); // 主8
    resolve('resolve'); // 主9
}).then(function(res) { // 主10
    console.log('promise2'); // 微3
    console.log(res)	     //微4
});
console.log('script end'); // 主11


// 按 主1-11，微1-4，队1 执行顺序，输出如下
// script start -> 主1
// async1 start -> 主4
// async2       -> 主6
// promise1     -> 主8
// script end   -> 主11
// async1 end   -> 微1
// promise2     -> 微3
// resolve      -> 微4
// setTimeout   -> 队1
```
上面的过程表述起来就是：
主0（压入script，准备执行script代码）
主1（直接入栈后出栈输出）
主2（计时器线程在0s后将console消息放入消息队列1）
主3（async1压栈，保存当前指针位置信息，此时进入该函数内部）
主4（直接入栈后出栈输出）
主5（await后面的函数async2入栈，其后面的函数内容(本质上也是个回调)async1 end被阻塞放在微任务队列1中，此时进入async2内部）
主6（直接入栈后出栈输出）
此时async2出栈
主7（function（resolve）入栈，其回调函数内容resolve被阻塞放在微任务队列2中，此时进入function（resolve）该函数内部）
主8（直接入栈后出栈输出）
主9（直接入栈后调用resolve出栈，此时resolve回传的值为'resolve'）
主10（then入栈，then的回调函数内容promise2被阻塞放在微任务队列3中）
主11（直接入栈后出栈输出）
此时script出栈，主调用栈为空，此时开始立即执行微任务队列中的任务
微1 微2 微3 微4
此时微任务队列为空，开始执行消息队列中的任务
队1


- 最后再举一个理解上容易出问题的例子
```javascript
async function cv() {
    new Promise((res, rej) => {
        setTimeout(() => {
            console.log('0');
        })
        setTimeout(() => {
            res('1');
            console.log('2');
        })
    }).then((res) => {
        console.log(res);
    })
    console.log('3');
}
cv();

//输出3 0 2 1
```
按顺序走首先new Promise入栈，此时其箭头函数整个入栈,按顺序走先将第一个setTimeout入栈，其里面的内容console.log('0')进入消息队列；接下来第二个setTimeout入栈，其里面的内容console.log('2')进入消息队列,这里**需要注意**,和上面不同，这里**不会直接进入then**，上面那个例子之所以进入了then是因为其promise里面的内容是普通函数，会直接压栈后执行，所以**promise执行完了可以进入then**；而这里promise里的内容是带setTimeout的，其**里面的函数在消息队列中还未被执行**，所以这里不会直接进入then，而是去执行下面的console.log('3')，等到消息队列中相应的函数执行完以后，then才可以被调用，里面的回调函数入栈，而回调函数内容则入微消息队列





从设计上理解，单线程意味着js任务需要排队，如果前一个任务出现大量的耗时操作，后面的任务得不到执行，任务的积累会导致页面的“假死”。这也是js编程一直在强调需要回避的“坑”。
主线程会循环上述步骤，事件循环就是主线程重复从微任务队列、消息队列中取消息、执行的过程。
需要注意的是 GUI渲染线程与JS引擎是互斥的，当JS引擎执行时GUI线程会被挂起，GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。因此页面渲染都是在js引擎主线程调用栈为空时进行的。

![JS运行时](http://leesdog.oss-cn-shanghai.aliyuncs.com/image_1641987064753.png)


> 参考资料：
[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
[2分钟了解 JavaScript Event Loop](https://www.bilibili.com/video/BV1kf4y1U7Ln)
[JavaScript中的事件循环与消息队列](https://www.jianshu.com/p/50ab99baf026)
[知乎-JS消息队列只有普通队列和延迟队列吗](https://www.zhihu.com/question/412373735)

