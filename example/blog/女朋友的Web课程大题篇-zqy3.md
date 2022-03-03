---
title: 女朋友的Web课程大题篇
date: 2021-06-20 02:04:32.426
updated: 2022-01-16 18:31:28.365
url: https://leesdog.space/archives/zqy3
categories: 
tags: 
- 前端
---

# 网络技术及应用复习资料-大题篇
> 超级超级对不起！！！李曾帅在此给你道歉了！！！这两天喝酒吃饭搞傻了，本来那天还记在心里的，后来搞忘记了/(ㄒoㄒ)/~~ 不知道还来不来得及，真诚向你道歉！！！

## 阅读理解1：
答：
1.首先运行sql文件(假设sql文件存储在E:\TestNode\memmana.sql，数据库账号为root，密码为666666，数据库名为test)，打开mysql命令行输入：
```sql
mysql -uroot -p666666 -Dtest<E:\TestNode\memmana.sql

```

2.在项目当前根目录下打开cmd，安装nodejs的http与mysql包模块：
```js
npm install http
npm install mysql

```

3.运行项目：
```js
node example7_4_2.js

```



## 阅读理解2：
主要是要通过$来获取相应对象而不是通过原来的getElementById函数
代码如下：
```html
<meta charSet="utf-8"/>
<title>jQuery用法示例</title>

<input type="button" id="btn" value="关闭窗口"/>
<script src=" http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript">
    var btn = $("#btn");  //定义jQuery对象
    $("#btn").click(function () {  //响应关闭按钮
        if(window.confirm('真的是退出吗？')){
            window.close();
        }
    });
</script>
```
最终效果：
<meta charSet="utf-8"/>
<title>jQuery用法示例</title>

<input type="button" id="btn" value="关闭窗口"/>
<script src=" http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript">
    var btn = $("#btn");  //定义jQuery对象
    $("#btn").click(function () {  //响应关闭按钮
        if(window.confirm('真的是退出吗？')){
            window.close();
        }
    });
</script>

## 程序设计题
由于我并不知道你这的第二道大题那个框架是什么，所以我也不知道应该怎么去填空，因为完成这一功能有很多实现方法；不过万变不离其宗，这里你需要理解以下几个知识点：

### 1. 如何去创建两个下拉菜单
这里主要是通过select这个标签去创建，例如我创建一个省份菜单：

```html
<select name="province">
  <option value ="hubei">湖北</option>
  <option value ="hunan">湖南</option>
  <option value="sichuan">四川</option>
  <option value="beijing">北京</option>
</select>
```

上述代码效果如下：
<select name="province">
  <option value ="hubei">湖北</option>
  <option value ="hunan">湖南</option>
  <option value="sichuan">四川</option>
  <option value="beijing">北京</option>
</select>

### 2.如何实现点击省后市出现对应数据，以及点击后的弹框显示
那么如何去实现点击这个里面某个省份另一个下拉列表里的数据动态生成呢？事实上如果用的是非vue做法，这里我们需要向后端请求对应的数据，例如我选择湖北以后应该立刻向后端发送一个数据，告诉后端我选的是湖北这个省，你需要把湖北省里有哪些市的信息发送给我，然后我再进行添加；


```html
<meta charSet="utf-8"/>
<script type="text/javascript" src=" http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
<select name="province">
  <option value ="hubei">湖北</option>
  <option value ="hunan">湖南</option>
  <option value="sichuan">四川</option>
  <option value="beijing">北京</option>
</select>

<select name="city">
</select>

<script type="text/javascript">

//这一句话的意思是当province这个列表里选择的内容发生改变的时候触发当前写的函数
  $('#province').change(function() 
  {
    url = "dataget.php"; //这个是定义的一个后端数据接口，也就是之后要把请求的命令发送给这个地址
//将当前选择的省通过ajax的post协议发送给后端，这里current:$(this).val()
//获取到的是province这个标签中目前选择的具体是哪一个标签，最终和获取到的数据在msg中
    $.post(url,{current:$(this).val()}, function(msg)
    {
      var tmp=eval(msg);
      var str='';
//这里相当于一直在获取msg中的市级信息，并且将其不断的添加option进去
      for(var i=0;i<tmp.length;i++){
        var c=tmp[i];
        str+='<option value='+c+'>'+c+'</option>';
      }
//当添加完成以后将city也就是市的下拉菜单的html内容进行改变，改成刚刚添加的那些option
      $('#city').html(str);
    });
  });

 $('#city').onblur(function(//这是定义当市这个下拉菜单失去焦点时触发的函数，这里只需要获取到当前的省市是啥就可以了
{
  var province=$('#province');
  var city=$('#city');
  window.alert('你选择了:'+province+city);
}
))
</script>
```

> 再次表达强烈的歉意！！！以后一定补偿回来！！！
