---
title: 原神大地图——2.UI交互层开发
index_img: /img/Vue.jpg
date: 2024-10-01 20:40:00
tags:
  - Vue.js
  - web
  - Typescript
  - Vite
  - Leaflet
---
本节将开发
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001204056.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001204116.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001204135.png)
# 整体ui搭建
首先编写左侧filter-config代码
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001204507.png)
```vue
<template>
  <div id="map-container">
    <div id="map">
      <div class="ui-layer">
        <div class="filter-container">

        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .ui-layer .filter-container{
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 415px;
    padding: 20px;
  }
  .ui-layer .filter-container .filter-content{
    position: relative;
    display: flex;
    flex-direction: column;
    background-color: #3b4354;
    width: 100%;
    height: 100%;
    border-radius: 12px;
  }
</style>
```
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001205523.png)
基本的ui框架搭建完成分析官网中的界面，可以进行组件拆分
在component下新建FilterHeader.vue组件并在filter-container中引入该组件
首先完成header部分效果
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001211122.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001211057.png)
```vue
<style scoped>
.filter-header {
  display: flex;
  align-items: center;
  padding: 12px 7px 7px 10px;
  color: #d3bc8e;
}
.header-icon {
  width: 40px;
  height: 40px;
  background-image: url("../assets/images/ui/xumi-icon.png");
  background-size: cover;
  margin-right: 5px;
}
.header-name{
  font-size: 18px;
  font-weight: bold;

  margin-right: 10px;
}
.switch-btn{
  display: flex;
  align-items: center;
  padding: 4px 8px 4px 4px;
  border-radius: 12px;
  background-color: rgb(74, 83, 102);
}
.switch-icon{
  width: 16px;
  height: 16px;
  background-image: url("../assets/images/ui/switch-btn.png");
  background-size: cover;
}
.switch-text{
  font-size: 12px;
}
</style>
```
编写关闭按钮代码和样式
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001212656.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001212754.png)
```vue
.close-btn{
  position: absolute;
  top: 32px;
  right: -44px;
  width: 64px;
  height: 40px;
  background-image: url("../assets/images/ui/close-bg.png");
  background-size: cover;
  display: flex;
  align-items: center;
  padding-left: 18px;
  box-sizing: border-box;
  z-index: 10;
}
.close-icon{
  width: 24px;
  height: 24px;
  background-image: url("../assets/images/ui/close-icon.png");
  background-size: cover;
}
```
新建LocationBtn.vue并在Filter-header上方引入。实现一个占位组件
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001213219.png)
```vue
<template>
  <div class="location-btn">

  </div>
</template>

<style scoped>
.location-btn{
  position: absolute;
  top: 84px;
  right: -30px;
  width: 40px;
  height: 40px;
  background-color: rgba(50,57,71,0.8);
  border-radius: 8px;
  cursor: pointer;
}
```
新建SelectedArea组件，内容与LocationBtn类似，调整以下top值到合适的位置即可
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001213520.png)
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001213543.png)

新建FilterMain组件，编写相应UI代码并引入
```vue
<template>
<div class="filter-main">
  <div class="filter-main-left">

  </div>
  <div class="filter-main-right"></div>
</div>
</template>

<style scoped>
.filter-main{
  flex: 1;
  display: flex;

}
.filter-main-left{
  width: 97px;
  background: #323947;
}
.filter-main-right{
  flex: 1;
}
</style>
```

最终的filter区域HTML结构：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001214042.png)
效果展示：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241001214105.png)