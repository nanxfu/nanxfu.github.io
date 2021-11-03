---
title: CSS - Box Model
date: 2021-11-03 13:26:54
tags: ['CSS']
---
*CSS 显示的所有内容都是一个'盒子'，所以了解 CSS 盒模型工作原理是 学习CSS 的基本功*

假如你写了这段 HTML
```HTML
<p> 这是一段简短的文字 </p>
```
然后添加了一段 CSS 样式
```CSS
p {
    width: 100px;
    height: 50px;
    padding: 20px;
    bordere:1px solid;
}
```
{% codepen WNRemxN default_tabs 300 100% web-dot-dev 33713%}