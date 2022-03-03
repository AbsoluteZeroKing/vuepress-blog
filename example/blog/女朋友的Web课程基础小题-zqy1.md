---
title: 女朋友的Web课程基础小题
date: 2021-06-16 18:30:19.116
updated: 2022-01-16 18:30:51.226
url: https://leesdog.space/archives/zqy1
categories: 
tags: 
- 前端
---

# 网络技术及应用复习资料-单选篇

> 李曾帅著



## 单项选择题

#### 1. 常用表单元素的标签名及主要属性

详见这个地址：

https://www.w3school.com.cn/tags/index.asp

里面包含了所有html元素标签与其属性，需要关注的是这几个：

1. **form**: 定义供用户输入的表单。
2. **fieldset**: 定义域。即输入区加有文字的边框。
3. **legend**：定义域的标题，即边框上的文字。
4. **label**：定义一个控制的标签。如输入框前的文字，用以关联用户的选择。
5. **input**: 定义输入域，常用。可设置type属性，从而具有不同功能。
6. **textarea**: 定义文本域(一个多行的输入控件)，默认可通过鼠标拖动调整大小。
7. **button**: 定义一个按钮。
8. **select**: 定义一个选择列表，即下拉列表。
9. **option**: 定义下拉列表中的选项。

还有几个需要注意的点：

- 首先是一个元素所包含的属性分为三块

  - 这个标签特有的属性

    - 例如下图写了input标签的相关特有属性
    - ![image-20210616170228414](https://image.leesdog.space/zqyh1.png)

  - 全局属性

    - 不是每个标签都支持全局属性的，支不支持在上面网页中国如果写了全局属性就是支持

    - 例如下图：

      ![image-20210616170532653](https://image.leesdog.space/zqyh2.png)

  - 事件属性

    - 基本同上面的全局属性，不是每个标签都支持事件属性的
    - 事件属性是当触发某些动作时所执行的，例如鼠标点击，键盘按键等等



#### 2. 常用HTML特殊字符对应的代码（空格、小于号、大于号、人民币）

| 符号 | 说明         | 编码(中间不带空格，之所以这里写的时候带是因为不带的话会被转义，答题的时候不需要写空格) | 英文（帮助记忆）               |
| ---- | ------------ | ------------------------------------------------------------ | ------------------------------ |
| &    | AND符号      | &amp ;                                                       | **amp**ersand                  |
| <    | 小于         | &lt ;                                                        | **l**i**t**tle                 |
| >    | 大于         | &gt ;                                                        | **g**rea**t**                  |
|      | 不断行的空格 | &nbsp ;                                                      | **n**um**b**er **sp**ace       |
| ?    | 问号         | &quest ;                                                     | **quest**ion                   |
| "    | 双引号       | &quot ;                                                      | **quot**e                      |
| ×    | 乘号         | &times ;                                                     | **times**                      |
| ÷    | 除号         | &divide ;                                                    | **divide**                     |
| \|   | 竖线         | &vert ;                                                      | **vert**icalLine               |
| ±    | 加减号       | &plusmn ;                                                    | **plus** **m**i**n**us         |
| ≠    | 不等号       | &ne ;                                                        | **n**ot **e**qual              |
| ≤    | 小于等于     | &le ;                                                        | **l**ess than or **e**qual to  |
| ≥    | 大于等于     | &ge ;                                                        | **g**reat than or **e**qual to |
| ¥    | 人民币       | &yen ;                                                       |                                |

如果觉得这些还不够可以去下面这个网站找到字符转义大全：

http://114.xixik.com/character/



#### 3. 如何应用CSS样式取消

#### （1）超链接默认的下划线|

如果超链接是这样的：

```html
<a href="具体连接地址">这是一个链接</a>
```

则像如下这么写就可以（主要注意下面的内容和上面的标签名是一致的，此处是a）：

```html
<style type="text/css">a {text-decoration: none}</style>
```

#### （2）项目列表符号

这里其实和上面类似，只是这里用的是list-style-type，一般要取消的项目列表标签是ul和ol，所以下面的写法是:

```html
<style type="text/css">ul,ol {list-style-type: none}</style>
```



#### 4. 并排列表项和同级Div所使用的CSS样式属性

加入display:inline即可解决实现同行并排显示div或者列表li对象，具体代码如下（注意如果是要并排div下面就写div，并排列表就用ul或者ol或者dl，如果要让多个项都实现并排，则像上面那样每个项之间加上,即可）：

```html
<style type="text/css">div{ display:inline}</style>
```

#### 5. 页面元素的水平居中方法

这个的实现方式有很多，具体看这个网页：

https://www.cnblogs.com/chengxs/p/11231906.html

#### 6. 项目列表所使用的两个HTML标签

这里严格来说分为很多种情况，因为项目列表分为多种

- 有序列表，用到的两个标签是

  ```
  <ol>
  	<li>
  ```

- 无序列表，用到的两个标签是：

  ```
  <ul>
  	<li>
  ```

- 自定义列表，用到的两个标签是：

  ```
  <dl>
  	<dt>
  ```

  

#### 7. 常用的浏览器对象与JavaScript内置对象

浏览器对象：

![image-20210616175523522](https://image.leesdog.space/zqyh5.png)

js内置对象：

![image-20210616175621502](https://image.leesdog.space/zqyh6.png)

#### 8. 浏览器对象window具有的几个重要方法

主要是这几个：

```js
window.setTimeout()
window.setInterval()
window.clearTimeout()
window.clearInterval()
window.open()
window.close()
window.alert()
window.confirm()
window.prompt()
```

至于每个方法是干什么的以及需要输入什么参数，可以参考这一篇博客：

https://blog.csdn.net/l_ppp/article/details/106590670



#### 9. 如何将超链接的内容显示在页内框架里？

其实这里他的意思就是和他的demo里一样：

![image-20210616181524568](https://image.leesdog.space/zqyh8.png)

要想实现这样一个功能需要首先在右侧这一块定义一个iframe标签用于之后显示超链接的内容，然后给这个标签一个name属性，例如kj：

```html
<iframe name="kj" width="550px" height="550px"></iframe>
```

然后在下面的超链接中加入target属性，属性的值为name的值kj即可：

```html
<a href="http://www.wustwzx.com/webfront/sy/sy1.html" target="kj">1.PHP网站开发环境</a>
```

如此便可实现点击一个超链接，然后会将超链接中的内容显示到指定iframe中（当然也可以显示到其他的属性中去）

#### 10. 弹性容器的定义、元素的水平均匀排列与垂直居中的设置方法

1. 弹性容器的定义：

   弹性容器是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式

   具体可见：https://www.runoob.com/css3/css3-flexbox.html

2. 元素水平均匀排列的方法：

   虽然说设置的方法很多，但这块应该是利用弹性容器来实现这样一个功能；具体来说就是设置弹性容器的justify-content属性为**space-between**或者**space-around**即可，两者唯一的区别是两边是否留空间，两者都是水平均匀排列

3. 元素垂直居中排列的方法：
   也是使用弹性容器来实现，具体来说就是设置弹性容器的align-items属性为center即可



















