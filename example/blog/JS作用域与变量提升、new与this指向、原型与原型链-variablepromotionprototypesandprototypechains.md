---
title: JS作用域与变量提升、new与this指向、原型与原型链
date: 2021-12-27 18:10:44.0
updated: 2022-01-17 21:40:17.514
url: https://leesdog.space/archives/variablepromotionprototypesandprototypechains
summary: JS的作用域和Java有一定区别，虽然名字上也分为局部作用域和全局作用域
categories: 
tags:
- JS
- 作用域
- 变量提升
- new
- this
- 原型
- 原型链
---

# JS作用域与变量提升、new与this指向、原型与原型链

[TOC]

### JS作用域与变量提升

JS的作用域和Java有一定区别，虽然名字上也分为局部作用域和全局作用域

**局部作用域：**也可以理解为**函数作用域**，只有在函数重声明的变量才会成为局部变量

**全局作用域：**函数之外声明的变量均为全局变量

几个关键点需要注意：

1. 内部作用域可以访问外部作用域的变量，但外部不能访问内部(例如所有函数都可以访问window，但window不可能调用你后面声明的变量)

2. JS有变量提升机制，即会提升到目前作用域的最高处，隐式出现（var变量提升只帮你声明该变量，但提升时不赋值，值默认为undefined；而函数变量提升则会将函数默认放到最上面最先加载）

3. 如果为尚未声明的变量赋值，此变量会自动成为全局变量(例如在函数内使用 a=10，a未被声明，则自动转换到全局window.a=10)。

4. 变量的查找顺序：

   先找自己的（包含变量提升），在找外部的；

   先找自己显式声明的，再找变量提升

   先找同名函数，再找同名变量（函数和变量都还未执行需要变量提升）

   先找同名变量，再找同名函数（函数和变量都执行了的情况下）

   

   例如下面这一段代码

   ```js
   var a = 'aout';
   fun(a);
   function fun(a) {
       //隐式会按下列顺序出现这些
       // var a=参数a
       // var a='ain'
       // function a(){console.log('a()');}
   
       console.log(a);
       var a = 'ain';
       console.log(a);
       function a() { console.log('a()'); }
       function b() { console.log('b()'); }
       console.log(a);
       console.log(b);
       var b = 'b';
   }
   
   //最终的输出
   // ƒ a() { console.log('a()'); }
   // ain
   // ain
   // ƒ b() { console.log('b()'); }
   ```

### new与this指向

我们首先要理解new做了什么，翻阅MDN我们可以知道，new在底层其实做了这些：

1. 创建一个空的简单JavaScript对象（即**{}**）；
2. 为步骤1新创建的对象添加属性**__proto__**，将该属性链接至构造函数的原型对象 ；
3. 将步骤1新创建的对象作为**this**的上下文 ；
4. 返回这个对象，如果该函数没有返回对象，则返回**this**。

这里我们先不深入理解，只需要知道两点:

- new会创建一个新对象
- 新对象会成为this的指向

至于new的细节到下面的原型与原型链中我们将做深层次的分析理解；

来看下一段代码

```js
function fun() {
    this.a = '123';
    console.log(this)
    return '111'
}
console.log(fun);
console.log(fun());
console.log(new fun());
```

第一个console处的fun，由于没有调用他，这里将其当作一个普通变量来看，所以打印出的就是他本身的函数内容

> result：
>
> ƒ fun() {
>          this.a = '123';
>          console.log(this)
>          return '111'
>      }

第二个console处的fun()，将fun()当作静态函数调用，所以首先执行其内部代码，同时此处由于是在全局调用fun()，fun的this指向全局window，同时fun()返回值为111，所以最后打印结果如下

> result:
>
> Window {window: Window, self: Window, document: document, a: "123", location: Location, …}
> 111

第三个console处的new fun()，实例化了一个对象，此时将fun()作为对象挂载到了其调用的对象上(即window上)，然后执行fun()，此时fun()里面的this指向自己这个对象；然后new fun()返回的也是自己这个对象，所以console再次打印一次

> result
>
> fun {a: "123"}
> fun {a: "123"}





### 原型与原型链

其实经过上面一系列demo例子我们感觉已经快能理解这些内容了，但我们其实已经可以感觉到了，函数的局部与全局作用域，以及new和this指向，背后有一层更加奇妙的东西在做支撑，为了搞清楚这其中到底做了什么，我们可以深入理解一下new所做的步骤；不过在此之前，我们先弄清其new的对象实现的原理

我们都知道javaScript是一门支持面向对象的程序语言，但其与我们现在学习过的java，c++不同的是，JavaScript的面向对象是依靠**原型和原型链**所完成的

当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（object）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（**prototype**）。该原型对象也有一个自己的原型对象（__proto__），层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。

遵循ECMAScript标准，`someObject.[[Prototype]]` 符号是用于指向 `someObject` 的原型。从 ECMAScript 6 开始，`[[Prototype]]` 可以通过 [`Object.getPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf) 和 [`Object.setPrototypeOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 访问器来访问。这个等同于 JavaScript 的非标准但许多浏览器实现的属性 `__proto__`。

但它不应该与构造函数 `func` 的 `prototype` 属性相混淆。被构造函数创建的实例对象的 `[[Prototype]]` 指向 `func` 的 `prototype` 属性。**`Object.prototype`** 属性表示 [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) 的原型对象。

上面说的比较绕，但是可以用几句话总结与完善

1. javascript一切皆为对象

2. 对象由Function创造，而Function本身也是对象

3. 每个对象都有一个隐式原型\_proto_

   每个函数都有一个显式原型prototype

4. 对象的\_proto_自己也是一个对象，指向其构造函数原型的prototype

   函数的prototype自己也是一个对象，其有一个属性值叫constructor，指向函数本身

   如下图:

   ![未命名文件](https://image.leesdog.space/JS%E5%8E%9F%E5%9E%8B%E9%93%BE.png)

同时由于function testFunc()是函数的同时本身其实也是一个对象，所以也有一个\_proto_属性指向构造他的原型的prototype；

补充上图

![未命名文件 (1)](https://image.leesdog.space/js%E5%8E%9F%E5%9E%8B%E9%93%BE2.png)

继续套娃，由于prototype本身也是一个对象，所以它也有自己的\_proto_，指向的是构建它的原型的prototype，谁构建的它呢，答案是Object本身，所以补充上图

![未命名文件 (2)](https://image.leesdog.space/js%E5%8E%9F%E5%9E%8B%E9%93%BE3.png)

继续套娃，由于Object本身也是一个对象，所以其也有自己的proto，指向的是构造它的原型的prototype，前面已经说了，对象由Function构造，所以完善上图

![未命名文件 (3)](https://image.leesdog.space/js%E5%8E%9F%E5%9E%8B%E9%93%BE4.png)

但是Object.prototype.\_proto_较为特殊，因为Obejct.prototype已经是顶级对象的原型了，而\_proto\_ 要指向构造其原型的prototype，这里是没有的，所以指向null

同时由于函数对象都是被顶级构造函数Function所创建的，Function是被自身创建的，所以Function.\_proto_===Function.prototype

同时由于Function.prototype本质上也是一个对象，所以其proto指向创建其原型的prototype，即Object.prototype

![未命名文件 (5)](https://image.leesdog.space/js%E5%8E%9F%E5%9E%8B%E9%93%BE6.png)



接下来我们来看一下原型之间的继承关系

```js
// 让我们从一个函数里创建一个对象o，它自身拥有属性a和b的：
let f = function () {
   this.a = 1;
   this.b = 2;
}
/* 这么写也一样
function f() {
  this.a = 1;
  this.b = 2;
}
*/
let o = new f(); // {a: 1, b: 2}

// 在f函数的原型上定义属性
f.prototype.b = 3;
f.prototype.c = 4;

// 不要在 f 函数的原型上直接定义 f.prototype = {b:3,c:4};这样会直接打破原型链
// o.[[Prototype]] 有属性 b 和 c
//  (其实就是 o.__proto__ 或者 o.constructor.prototype)
// o.[[Prototype]].[[Prototype]] 是 Object.prototype.
// 最后o.[[Prototype]].[[Prototype]].[[Prototype]]是null
// 这就是原型链的末尾，即 null，
// 根据定义，null 就是没有 [[Prototype]]。

// 综上，整个原型链如下:

// {a:1, b:2} ---> {b:3, c:4} ---> Object.prototype---> null

console.log(o.a); // 1
// a是o的自身属性吗？是的，该属性的值为 1

console.log(o.b); // 2
// b是o的自身属性吗？是的，该属性的值为 2
// 原型上也有一个'b'属性，但是它不会被访问到。
// 这种情况被称为"属性遮蔽 (property shadowing)"

console.log(o.c); // 4
// c是o的自身属性吗？不是，那看看它的原型上有没有
// c是o.[[Prototype]]的属性吗？是的，该属性的值为 4

console.log(o.d); // undefined
// d 是 o 的自身属性吗？不是，那看看它的原型上有没有
// d 是 o.[[Prototype]] 的属性吗？不是，那看看它的原型上有没有
// o.[[Prototype]].[[Prototype]] 为 null，停止搜索
// 找不到 d 属性，返回 undefined
```

当访问一个对象的方法或属性时，我们会先找他自己本身的属性方法，如果没有我们会去其原型链中逐层查找其father有没有相应的方法(这和java,C++等面向对象的类十分相似)

如何实现我们熟悉的这样一种继承关系呢？假设我想让son()函数继承自father()，在js里很容易，只需要挂件的一行代码：

**son.prototype=new father()**

这时我们就可以解释上面的疑问了，即new中间到底干了啥，总结起来四个步骤：

1. 创建一个空的简单JavaScript对象（即**{}**）；
2. 为步骤1新创建的对象添加属性**__proto__**，将该属性链接至构造函数的原型对象的prototype ；
3. 将步骤1新创建的对象作为**this**的上下文 ；
4. 返回这个对象，如果该函数没有返回对象，则返回**this**。

所以我们可以手写出new这个函数的过程：



```
function Mynew(constructor, argument) {
//1.创建一个空的简单JavaScript对象（即**{}**）；
const obj = {};
//2.为步骤1新创建的对象添加属性__proto__，将该属性链接至构造函数的原型对象的prototype ；
Object.setPrototypeOf(obj, constructor.prototype);
//3.将步骤1新创建的对象作为this的上下文 ；
constructor.call(obj, argument);
//4.如果该函数没有返回对象，则返回**this**。
return obj;
}

function Father(name) {
this.name = name;
console.log(this);
}

// 替换var father = new Father('lzs')
var father = Mynew(Father, 'lzs');
console.log(father.name);
```

最终结果：

![image-20220116180817902](https://image.leesdog.space/js%E5%8E%9F%E5%9E%8B%E9%93%BEnew%E6%BC%94%E7%A4%BA.png)

