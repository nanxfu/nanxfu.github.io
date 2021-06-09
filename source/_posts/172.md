---
title: PHP中$this和self的区别
tags:
  - php
id: '272'
categories:
  - - 网站建设
date: 2020-03-28 16:37:24
---

## $this

\[toc\]

$this提供了对自身类的引用，可以访问自身类下的所有的成员。属性/方法

## self

self和$this一样，保存了自身类的引用，也可以访问自身类下的所有成员。

## 对象操作符

"->"是php中内置的符号，它和$this一起使用来访问类的成员（属性或方法）

"::"是访问类中静态属性的操作符，可以和self结合使用访问类的静态成员

## this和self的区别：

```php
使用 $this -> member 来访问自身类下的非静态成员
使用 self::$member 来访问自身类下的静态成员
```