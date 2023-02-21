---
title: uikit下img组件height与width属性
tags: []
id: '69'
categories:
  - - 网站建设
  - - 视觉设计
date: 2020-03-07 02:27:00
---

在使用uikit的img组件时我发现指定的height属性没用生效。width与height同时指定为100（原图为316X256）。 但在页面上实际渲染效果却只有100X81 ![QQ截图20200306181300.png](http://nanxfu.cn/wp-content/uploads/2020/03/1871535963.png "QQ截图20200306181300.png") ![QQ截图20200306181300.png](http://nanxfu.cn/wp-content/uploads/2020/03/2727641762.png "QQ截图20200306181300.png")

* * *

于是我打开chrome的调试工具检查了样式，发现原来时**UIKIT自带的样式把height指定为了auto**

![QQ截图20200306181523.png](http://nanxfu.cn/wp-content/uploads/2020/03/1711722119.png "QQ截图20200306181523.png")

经过我不断的实验，最终得知height的auto值是按width的比例换算。换算下来刚好为81px。 所以只要在标签里内联一个样式指定height样式就行了`style=height:100px`

因为css的Specificity效果，内联样式就会覆盖外联样式 嗐 也怪我学艺不精(_/ω＼_)