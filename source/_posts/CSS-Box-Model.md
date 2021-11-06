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

像上面看到的那样，文本内容会溢出到边框外。打开调试器会发现宽度为 142px 左右，而不是设定的 100px。为什么会这样呢？盒模型是 CSS 的基本概念，理解它是如何运作的。更重要的是，你学会如何利用它后才能写出看起来符合你预期的 CSS，不然就会出现各种诡异的行为（

你要记住，在网页中，所有用 CSS 显示的元素都会被浏览器解释为一个 "box"

## Content and sizing | 内容和大小 ##
根据 display 取值、设置的大小和 Box 中的内容不同，Box在布局中的显示效果也不同。
内容指的就是由子元素、或纯文本内容生成的 Box。默认情况下，这些内容会影响 Box 的大小

你可以使用用 extrinsic sizing （译为外部大小，即指 width 属性） 控制元素的 content 大小，或者让浏览器根据 content 的大小来决定 intrinsic sizing (译为固有大小) 自主决定

{% codepen abpoMBL default_tabs 600 100% web-dot-dev 33713%}

