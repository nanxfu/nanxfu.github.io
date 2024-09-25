---
title: 读书管理后台开发项目-3.Nest框架搭建
date: 2024-09-23 22:01:09
tags:
  - web
  - Typescript
  - Nest.js
---
# 1.Nest框架搭建
使用官方推荐的CLI搭建
```shell
npm i -g @nestjs/cli
nest new imooc-nest-admin
```

# 2.Nest实现Restful API
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924153648.png)
## 2.1 Get方法传参方式：Param
获得Get方法中的参数如果使用Param装饰器则对应获取restful Api参数
使用方法是直接在get方法函数中@param
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154059.png)
级联参数同样使用@param
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154229.png)

## 2.1 Post方法传参方式：Body
获取Post方法中的参数可以使用Body装饰器获取如：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154409.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154425.png)
同样，Post方法也可以在URL中添加Query参数
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154510.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20240924154545.png)

## 2.2 Put方法传参方式
Put方法和Post方法使用一致，可以用@Body接收数据，也可以在URL中Query参数

# 3.Provider
保存了服务的实例化的容器，如AppService中会有一个Injectable装饰器标识这个类可以被注入。这样controller可以引入服务

# 4.异常处理
当访问接口时，若抛出异常。使用自定义一茶馆可以用装饰器：@UseFilters()