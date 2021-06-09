---
title: Vue笔记。
tags: []
id: '76'
categories:
  - - 网站建设
date: 2020-03-17 01:55:00
---

## Computed和methods

在一个页面中，Computed不管被调用几次，实际上都只调用一次，且计算结果会缓存，但依赖改变依然会重新计算。而每次调用methods，实际上都会重新调用，所以页面中逻辑可以尽量写在Computed里然后通过插值方法调用（意识流文章×

## Component组件

在一个Script中调用Vue.component()方法创建一个组件时此组件属于根组件，任意模板（template）中都能使用。

而如果在Component()方法中指定组件对象（compoent{}）创建组件，那么这个组件为局部组件，只能被父组件调用

```javascript
Vue.component({
  components: {
   child:{
template:&#x60;&lt;div&gt;child&lt;/div&gt;&#x60;

       }
     }
})
```

*   template中必须只能有一个根节点
*   子组件无法直接访问到父组件中的数据
*   组件中的data必须是一个函数

### 父子组件通信方式

*   【父->子】使用props属性(使用props传递逻辑值时记得在属性前加 : 例如 :myprops=true。否则js会让你知道什么叫做黑魔法QAQ)
*   【父<-子】父组件需申明一个事件并赋值，子组件再通过this.$emit(事件名)调用。例如this.$emit(get,1000)

## 生命周期

当Vue实例中的data被修改时会调用beforeupdated()和updated()。而DOM树被修改时会重新循环生命周期并，此时mounted()就有很多的作用。

## vue.config.js

*   改了配置文件一定要重启服务器

### 反向代理设置

vue中的反向代理设置实际上是通过node的中间件[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware#proxycontext-config)实现的  
![QQ截图20200319223906.png](http://nanxfu.cn/wp-content/uploads/2020/03/1487693512.png "QQ截图20200319223906.png")

\[toc\]