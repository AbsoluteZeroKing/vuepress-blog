---
title: 女朋友的Web课程填空篇
date: 2021-06-16 19:33:49.468
updated: 2022-01-16 18:31:08.455
url: https://leesdog.space/archives/zqy2
categories: 
tags: 
- 前端
---

# 网络技术及应用复习资料-填空篇




## 填空题

#### 1. 如何设定HTML文档的编码为utf-8？

将下面这行代码放到 \<head\> 标签之中：

```html
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
```

例如：

![image-20210616184114432](https://image.leesdog.space/zqyh10.png)

#### 2. 进入浏览器调试是按什么功能键？

F12

#### 3. 超链接标签

\<a\>,常和href或者name属性一起用



#### 4. 图像标签必须定义的属性是什么？

src



#### 5. 如何制作顺序列表和项目列表？

项目列表分为多种

- 有序列表:

  ```html
  <ol>
  	<li>郑巧怡</li>
      <li>Flower</li>
  </ol>
  ```

  显示效果如下：

  <ol>
  	<li>郑巧怡</li>
      <li>Flower</li>
  </ol>

- 无序列表：

  ```html
  <ul>
      <li>郑巧怡</li>
      <li>Flower</li>
  </ul>
  ```

  显示效果如下：

  <ul>
      <li>郑巧怡</li>
      <li>Flower</li>
  </ul>

  

- 自定义列表：

  ```html
  <dl>
      <dt>郑巧怡</dt>
      <dt>Flower</dt>
  </dl>
  ```

  显示效果如下：

  <dl>
      <dt>郑巧怡</dt>
      <dt>Flower</dt>
  </dl>



#### 6. 制作下拉列表所使用的标签

主要需要使用select和option标签，如下：

```html
<select> 
<option value="1">我是下拉项目1</option> 
<option value="2">我是下拉项目2</option> 
<option value="3">我是下拉项目3</option> 
<option value="4">我是下拉项目4</option> 
</select>
```

显示效果如下：

<select> 
<option value="1">我是下拉项目1</option> 
<option value="2">我是下拉项目2</option> 
<option value="3">我是下拉项目3</option> 
<option value="4">我是下拉项目4</option> 
</select>



#### 7. 四种基本选择器

1. **标签选择器**:通过选择标签来进行选择，直接指明标签：

   例如有一个情景是这样：

   ```html
   <p>标签选择器</p>
   ```

   那么你就可以直接在css中指定该标签然后修改一些style：

   ```css
   <style type="text/css">
   p{
   font-size:40px;
   }
   </style>
   ```

   

2. **id选择器**:通过选择id来进行选择，符号为#

   例如有一个情景是这样：

   ```html
   <h2 id="good">id选择器</h2>
   ```

   那么你就可以使用#来指定id然后修改一些style：

   ```css
   <style type="text/css">
   #good{
   font-size:40px;
   }
   </style>
   ```

   

3. **类选择器**:通过选择class来进行选择，符号为.

   例如有一个情景是这样：

   ```html
   <p class="good">类选择器</h2>
   ```

   那么你就可以使用.来指定class然后修改一些style：

   ```css
   <style type="text/css">
   .good{
   font-size:40px;
   }
   </style>
   ```

   

4. **通配符\***：匹配所有标签：

   定义方式就是直接在style中写相应值即可：

   ```css
   <style type="text/css">
   font-size:40px;
   </style>
   ```

   

#### 8. 后代选择器与子选择器的关系如何？

后代选择器可以选择作为某元素后代的元素；而子元素选择器只能选择作为某元素子元素的元素。



这里给一个具体例子进行分析：

里面的nav ul li ul便是后代选择器；

里面的nav>ul>li和}nav>ul>li>ul>li是子选择器；

　1、nav>ul只选择nav下一级里面的ul元素，例如上面dom结构里id为a的ul。
　2、nav ul选择nav内所包含的所有ul元素，例如上面dom结构里面id为a、b、c的全部ul。
　3、nav>ul比nav ul限定更严格，必须后面的元素只比前面的低一个级别。

```html
<style>
        *{
            margin:0;
            padding:0;
            list-style:none;
        }nav ul li ul{
            display:none;
        }nav>ul>li{
            float:left;
            padding:10px;
            border:1px solid blue;
        }nav>ul>li>ul>li{
            padding:10px;
            border-bottom:1px solid #ccc;
        }
</style>
<!--
    >是指只能一代接一代，比如： nav>ul>li>ul>li，必须是下面这样的
-->
<nav>
    <ul>
        <li>
            <ul>
                <li></li>
            </ul>
        </li>
    </ul>
</nav>
<!--
  然后nav ul li ul只要求后面的元素是在nav标签下的顺序即可，对中间隔了几层不敏感，比如：
-->
<nav>
     <div>
         <ul>
             <div>
                 <a>
                     <li>
                         <div>
                             <ul>
                                 <li></li>
                             </ul>
                         </div>
                     </li>
                 </a>
             </div>
         </ul>
     </div>
</nav>
```



#### 9. 在JavaScript里，以匿名方式注册事件响应所使用的关键字是什么？

function前面赋值的变量

例如：

```js
var double = function(x) { return 2* x; }
```

这时注册onclick事件所需要使用的关键词就是double()



#### 10. 在页面里，通过（1）标签link引入外部样式文件（2）标签script引入外部的JavaScript脚本，分别使用什么属性？

link是html提供的标签，不仅可以加载css文件，还能定义 RSS、rel 、href属性等；

而script引入外部脚本使用的是@import，这是css中的语法规则



#### 11. Vue项目的配置信息以及所依赖的模块信息包含在哪个文件里？

配置信息在**vue.config.js**中

所依赖的模块信息在**package.json**中



#### 12. Vue项目要使用路由或Ajax功能，需要分别引入的模块名称是什么？

路由模块：vue-router

Ajax：axios

安装指令：npm install

