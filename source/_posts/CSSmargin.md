---
title: 'CSSmargin:auto踩坑'
tags:
  - CSS
  - margin
id: '296'
categories:
  - - 网站建设
date: 2020-03-29 18:10:52
---

css中得margin如果指定左右auto的话就能就能实现元素居中效果

![](https://i.loli.net/2020/03/29/1KGB9cfxNd5EFQk.png)

闲着的时候做的一个书单

但是，auto居中效果前提是必须**给该元素指定一个宽度**才能生效。比如width:50%

给行内元素(inline)设置上下margin时即使页面上会显示margin的存在，但是**不会**有效果。 因为行内元素只支左右的margin，而不支持上下的

* * *

给margin的左右赋值auto后，该元素就不能使用float了，不然会产生逻辑错误