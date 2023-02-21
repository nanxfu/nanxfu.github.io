---
title: 手撕webpack
tags:
  - webpack
id: '235'
categories:
  - - 网站建设
date: 2020-03-28 07:08:37
---

console.log(this)

# 快速上手webpack\[toc\]

## webpack是什么？

> 本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle

以上就是webpack官网的概念，简单地说，webpack就是个**模块打包器**现代前端开发为了方便，会在依赖关系上大作文章，以此来提高代码的可读性以及解耦。对程序员来说这是个很好的习惯，但对浏览器来说可有点难受了。

如果一个页面需要加载十几个js，十几个css文件的话那么网站的加载速度就会拖慢。而webpack就能将数个静态资源文件打包在几个静态资源文件中，这样就减少了HTTP请求，从而加快了网站加载速度。

当然，webpack并不仅仅只是将静态资源打包在几个文件中，它也能把浏览器不能识别的静态资源（如sass，less）转换成浏览器容易识别的静态资源

![webpack工作方式](https://i.loli.net/2020/03/28/ZtLJEKXfYbTDISW.png)

### 快速使用webpack

webpack包是托管在npm上的，所以我们需要用到npm包管理器，为了防止安装过程因网速导致一大堆ERROR，请在安装之前将npm镜像设置为淘宝镜像。当然也可以使用yarn安装。