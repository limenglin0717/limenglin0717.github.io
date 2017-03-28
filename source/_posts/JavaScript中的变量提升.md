---
title: JavaScript中的变量提升
date: 2017-01-26 10:13:23
tags: JavaScript
categories: JavaScript
---
我们先看一段很简单的Code：
```
var v='Hello World'; 
alert(v); //没有疑问输出 "Hello World"
```
这很简单。

我们在看一段Code：
```
var str='Hello World'; 
;(function(){ 
    alert(str); 
})()
//弹出 "Hello World"
```
经过运行之后，我们发现，还是和我们预期的一样，弹出了“Hello World”。

然而，今天的重点就要到来了,睁大眼睛看下面这一段代码：
```
var str='Hello World'; 
;(function(){ 
    alert(str); 
    var str='new string'; 
})() 
//输出undefined
```
竟然输出了 *"undefined"* 现在肯定心里一万只草泥马奔腾而过，WTF!

OK,这就是JavaScript中的**变量提升**：

>在js中，变量的声明会被解析器悄悄的提升到方法体的最顶部，但是需要注意的是，提升的仅仅是变量的声明，变量的赋值并不会被提升。

对于这个问题，简单粗暴的解释就是：在编写程序的时候用**var**定义变量，然后程序在**预编译**的时候
会对整个脚本文件进行解析，遇到var定义的变量时会提升到**块级作用域**顶部。(你要先了解作用域)

**在JavaScript中除了函数代码块外，if、for等结构也属于块级作用域**

所以上面的程序变成了：
```
var str='Hello World'; 
;(function(){ 
    var str;
    alert(str); 
    str='new string'; 
})() 
//输出undefined
```
到这里，大家的疑惑应该就豁然开朗了，所以在写程序的时候，建议大家把定义变量都写在作用域的顶部。