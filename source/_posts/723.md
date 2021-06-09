---
title: JavaScript函数传参时的值传递与引用传递
tags:
  - javascript
id: '319'
categories:
  - - 网站建设
date: 2020-05-19 23:02:02
---

> ECMAScript中的所有参数传递的都是值，不可能通过引用传递参数

一开始对这句话没有什么概念，只是简单的当成与其它语言差不多的“Feature”。但后来在实际开发中踩了个大坑才对这个问题深究。

先来看一段简单的代码

```
function changeColor(color){
    color = "blue"
}
var Mycolor = "green"
changeColor(Mycolor)
console.log(Mycolor)
```

![图片.png](https://sm.ms/image/pea53jUxZlRgEd1)

毫无意外，输出结果是“green”。  
  
在其它编程语言中也是一样的，函数传参时默认只传值。

但如果我们传入一个**对象**呢？

```
function changeColor(_obj){
    _obj.color = "blue"
}
var Mycolor = {color:"green"}
changeColor(Mycolor)
console.log(Mycolor.color)
```

![图片.png](https://sm.ms/image/FaejLZhXbgUzuHv)

然而，奇怪的是输出的Mycolor.color被改变了（从green - > blue）。

那是否可以认为调用函数传参的过程中传递了Mycolor的地址呢？但这不就与开头定义相悖了吗？

其实并不是，在上面一段代码中函数传递的依然是值，只不过传的值不是“Mycolor”，而是Mycolor引用（地址）的拷贝。

也就是拷贝了一份Mycolor的引用传进去。不信?来看下一段代码

```
function changeColor(_obj){
    _obj.color = "blue"
    _obj = new Object()
    _obj.color = "white"
}
var Mycolor = {color:"green"}
changeColor(Mycolor)
console.log(Mycolor.color)
```

上面的代码中只改变了两行，就是给\_obj赋予了一个新的空Object，并且增加了color属性，赋值为“white”。

![图片.png](https://sm.ms/image/qcVtwJejFfBnprZ)

再来看看输出结果  

输出依然是“blue”，那么这也就可以证明了传递的其实是**Mycolor的地址的拷贝**，并不是将Mycolor的引用（地址）传递了过去，

形参\_obj和Mycolor指向的同一个Object地址，所以改变 \_obj属性的值，Mycolor的值也会改变，但如果给 \_obj赋予一个新的值，那 \_obj就不会与Mycolor关联了。

手码不易，点个赞👍吧

参考资料：