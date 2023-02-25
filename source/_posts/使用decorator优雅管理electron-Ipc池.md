---
title: 使用decorator优雅管理electron-Ipc池
index_img: /img/electron.png
date: 2023-02-25 22:43:15
tags:
- typescript
- electron
- ipc通信
- decorator
---
## 1.decorator简介
> 要启用对装饰器的实验性支持，您必须在命令行或 tsconfig.json 中启用 experimentalDecorators 编译器选项

typescript的decorator一共有五种可被我们使用:
1. Class Decorators
2. Method Decorators
3. Accessor Decorators(访问器装饰器)
4. Property Decorators
5. Parameter Decorators
鉴于需求的原因，在本文仅介绍前两种装饰器。其它装饰器的介绍可在官网查阅
[中文手册](https://www.tslang.cn/docs/handbook/decorators.html) [官网手册](https://www.typescriptlang.org/docs/handbook/decorators.html)

### 1.1 Class Decorators

### 1.2 Method Decorators

## 2.Ipc通信

### 2.1 单向通信

### 2.2 双向通信

## 3.使用decorator管理Ipc

## 4.进阶：自动加载 Ipc handler