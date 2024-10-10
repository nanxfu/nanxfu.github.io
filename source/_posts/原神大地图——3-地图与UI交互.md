---
title: 原神大地图——3.地图与UI交互
index_img: /img/yuanshen.jpg
date: 2024-10-05 14:44:42
tags:
  - Vue.js
  - web
  - Typescript
  - Vite
  - Leaflet
---

# 地名动态渲染
实现根据地图缩放大小不同，地名展示也不同。
home store中封装mapAnchorList
```typescript
    function setMapAnchorList(data: any[]) {
        mapAnchorList.value = data
    }
```
将其他组件中的mapAnchorList ref替换为store中的ref。然后渲染一级地名
显示效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241005203218.png)
接着开发动态渲染二级地名的逻辑，开发思路为监听缩放，在map construct中添加zoom的事件监听，然后重新封装renderAreaNames

```typescript
 renderAreaNames() {
    let markers: L.Marker[] = []
    if (this.map.getZoom() >= 6) {
        //渲染二级地名
        this.mapAnchorList.forEach((val) => {
            let childrenList: L.Marker[] = []
            childrenList = val.children.map((val) => {
                const {lat, lng, name} = val
                const marker = L.marker(L.latLng([lat, lng,]), {
                    icon: L.divIcon({
                        className: 'map-marker-item',
                        html: `<div class="area-mark-item">${name}</div>`
                    })
                })
                return marker
            })
            markers = markers.concat(childrenList)
        })
    } else {
        //渲染一级地名
        markers = this.mapAnchorList.map((item) => {
            const {lat, lng, name} = item
            const marker = L.marker(L.latLng([lat, lng,]), {
                icon: L.divIcon({
                    className: 'map-marker-item',
                    html: `<div class="area-mark-item">${name}</div>`
                })
            })
            return marker
        })
    }


    this.areaNameLayerGroup = L.layerGroup(markers)
    this.areaNameLayerGroup.addTo(this.map)
}
```
至此实现效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/QQ2024105-204720-HD-ezgif.com-video-to-webp-converter.webp)
有个小问题就是缩放到二级地名时，一级地名应当被清除。缩放到显示一级地名时，二级地名应当被清楚**但实际没有**。所以需要修复这个BUG
重新封装代码。新增getAreaNameMakerItem函数
```typescript
    getAreaNameMakerItem(config: AreaNameConfig){
    const {lat, lng, name} = config
    return L.marker(L.latLng([lat, lng,]), {
        icon: L.divIcon({
            className: 'map-marker-item',
            html: `<div class="area-mark-item">${name}</div>`
        })
    })
}
```
同时重构renderAreaNames
```typescript
    renderAreaNames() {
    this.areaNameLayerGroup?.clearLayers()
    let markers: L.Marker[] = []
    if (this.map.getZoom() >= 6) {
        //渲染二级地名
        this.mapAnchorList.forEach((val) => {
            let childrenList: L.Marker[] = []
            childrenList = val.children.map(this.getAreaNameMakerItem)
            markers = markers.concat(childrenList)
        })
    } else {
        //渲染一级地名
        markers = this.mapAnchorList.map(this.getAreaNameMakerItem)
    }

    this.areaNameLayerGroup = L.layerGroup(markers)
    this.areaNameLayerGroup.addTo(this.map)
}
```
到此就正常展示地名了，但是有一个性能问题，当zoom在6，7内反复时。leaflet会不断重复渲染，实际上是没必要的。所以渲染逻辑改为当zoom处于一级与二级缩放临界值时才进行重渲染。
重构渲染时机。此处可解释为：curZoom与prevZoom不在同一个区间[4,5]，[6,7]时就渲染
```typescript
        this.map.on('zoom', (event) => {
            const curZoom = this.map.getZoom()
            if (curZoom >= 6 != this.prevZoom >=6){
                console.log('render')
                this.renderAreaNames()
                this.prevZoom = curZoom
            }

        })
```
最终效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/QQ2024105-204720-HD-ezgif.com-video-to-webp-converter.webp)

# 标点动态渲染
在UI界面选择了标点后在地图上显示相应的标点
在home store中找到calcSelectedFilterItems，也即找到选择筛选标点后对于的处理事件。增加*处的代码，标记需要渲染的标点数据
```typescript
function calcSelectedFilterItems(){
    let res: any[] = []
    let pointList: any[] = []   //*
    for (let i = 0; i < filterTree.value.length; i++){
        const item = filterTree.value[i]
        const activeItems = item.children.filter((child: any)=>{
            return child.active
        })
        activeItems.forEach((item)=>{
            const points: any = item.children.map((val) => {    //*
                return {
                    ...val,
                    icon: item.icon
                }
            })
            pointList = pointList.concat(points)
        })
        res = res.concat(activeItems)
    }
    selectedFilterItems.value = res
}
```
接着创建用于通信的全局变量
```typescript
//src/ts/global-data.ts
import {MapManager} from "./map-manager.ts";

class GlobalData {
    private static instance: GlobalData;
    private constructor() {

    }
    public static getInstance(): GlobalData{
        if(!GlobalData.instance){
            GlobalData.instance = new GlobalData();
        }
        return GlobalData.instance;
    }

    public mapManager: MapManager;
}

export const globalDataInst = GlobalData.getInstance()
```
将mapManger存入global变量后再在calcSelectedFilterItems函数内合适的位置渲染标点
```typescript
        if (globalDataInst.mapManager) {
    globalDataInst.mapManager.renderPoints(pointList)
}
```
最后在renderPoints中添加```this.pointerMarkersLayerGroup?.clearLayers()```以实现取消标点选择时在地图上清空未选中的标点
最终效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241007000948.png)

# 地图缩放条功能
使用已有的插件：Leaflet.zoomslider ```npm i leaflet.zoomslider```

引入插件```this.map.addControl(new L.control.Zoomslider())```
插件兼容性比较差，遂放弃（

# 修复标点偏移BUG
地图在放大时默认情况下标点会基于左上角定位，也即是在放大时标点会一直往左上角偏移，所以我们重新定位标点的锚点
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241007004211.png)
# 地图快速定位
在LocationBtn中添加对应的事件即可
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241007010147.png)
最终效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/QQ2024107-1237-HD-ezgif.com-video-to-webp-converter.webp)

# 标点点点击选中态
首先为每个icon数据赋予唯一ID，然后根据id上是否由active类来更改样式
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241007014231.png)

然后编写样式,一个角的图片通过旋转操作变成四个角
```css
:deep(.arrow-icon){
  position: absolute;
  display: none;
  width: 10px;
  height: 10px;
  background-image: url("../assets/images/map/arrow-l.png");
}
:deep(.arrow-icon.lb){
  left: 0;
  bottom: 6px;
}
:deep(.arrow-icon.lt){
  left: 0;
  transform: scaleY(-1);
  top: -5px;
}
:deep(.arrow-icon.rb){
  right: 0;
  transform: scaleX(-1);
  bottom: 6px;
}
:deep(.arrow-icon.rt){
  right: 0;
  transform: scale(-1);
  top: -5px;
}
```
最终效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/QQ2024107-14455-HD-ezgif.com-video-to-webp-converter.webp)

# 标点点击弹窗
使用[popup](https://leafletjs.cn/reference.html#popup)实现弹窗功能
在renderPoints中添加popupHTML结构和样式
```typescript
marker.bindPopup(L.popup({
    content:
        `<div class="point-popup-container">
                        <div class="popup-title">传送锚点</div>
                        <div class="popup-pic" style="background-image: url('https://webstatic.mihoyo.com/upload/wiki-ys-map/2023/01/18/306126220/a2c132b651daf17b7444bc575f2fa04d_790168931618739974.png?x-oss-process=image%2Fresize%2Cw_600%2Fquality%2CQ_90%2Fformat%2Cwebp')"></div>
                        <div class="point-name">传送锚点（蒙德）</div>
                        <div class="contributor-container">
                            <div class="contributor-label">贡献者：</div>
                            <div class="avatar-container">
                                <div class="avatar-item" style="background-image: url('https://webstatic.mihoyo.com/upload/wiki-ys-map/2023/01/18/306126220/a2c132b651daf17b7444bc575f2fa04d_790168931618739974.png?x-oss-process=image%2Fresize%2Cw_600%2Fquality%2CQ_90%2Fformat%2Cwebp')">
                                </div>
                            </div>
                        </div>
                        <div class="point-time">更新时间：2021-09</div>
                    </div>`
}), {
    maxWidth: 375
})
```
> 接下来添加交互逻辑

首先在mock文件中添加mock数据并封装好请求接口
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241007204410.png)
然后抽离出calcPopuoContent方法，用于动态生成popupcontent HTML字符串
```typescript
    calcPopuoContent(popupData:any){
        const {correct_user_list, last_update_time, info} = popupData
        const avatarElmStr = correct_user_list.map((val) => {
            return `<div class="avatar-item" style="background-image: url(${val.img})"></div>`
        })

        return `<div class="point-popup-container">
                        <div class="popup-title">传送锚点</div>
                        <div class="popup-pic" style="background-image: url(${info.img})"></div>
                        <div class="point-name">${info.content}</div>
                        <div class="contributor-container">
                            <div class="contributor-label">贡献者：</div>
                            <div class="avatar-container">
                                ${avatarElmStr}
                            </div>
                        </div>
                        <div class="point-time">更新时间：${last_update_time}</div>
                    </div>`
    }
```
最后在marker中监听popup事件并使用```marker.setPopupContent```方法重设popup content
```typescript
marker.on('popupopen', async()=>{
    const res = await getMapPointDetail(pointId)
    marker.setPopupContent(this.calcPopuoContent(res.data))
})
```
注意优化关闭popup时的功能，地图上监听click事件，当popup关闭时，标点的激活态应该被移除
```typescript
        this.map.on('click', ()=>{
            const lastActivePoint = document.getElementById(`mapPointItem${this.lastActivePointId}`)
            lastActivePoint?.classList.remove('active')

            this.lastActivePointId = -1
        })
```
> BUG 修复： 目前的popup弹窗中左上角的标题没有被更改，在home store中的calcSelectedFilterItems需传入name: item.name，然后再在模板中渲染值

最终效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/QQ2024107-205819-HD-ezgif.com-video-to-webp-converter.webp)

# 屏幕外标点引导
新建calcOutScreenPoints函数用于渲染屏幕外标点的引导UI
```typescript
calcOutScreenPoints() {
        const guideUIAry: GuideUiItem[]
        const calcPointMap: { [key: string]: any } = {}
        const center = this.map.getCenter()
        //遍历选中的标点，为选中的标点保存是否在屏幕中的信息
        for (let i = 0; i < this.pointList.length; i++) {
            const pointItem = this.pointList[i]
            const {name, lat, lng} = pointItem

            if (!calcPointMap[name]) {
                calcPointMap[name] = {}
            }

            if (calcPointMap[name].inScreen) {
                continue
            }

            const isContain = this.map.getBounds().contains(pointItem)

            if (!isContain) {
                const dist = this.map.getCenter().distanceTo(pointItem)
                if (!calcPointMap[name].pointItem) {
                    calcPointMap[name] = {dist, pointItem, inScreen: false}
                } else {
                    const curDist = calcPointMap[name].dist
                    if (dist < curDist) {
                        calcPointMap[name] = {dist, pointItem, inScreen: false}
                    }
                }
            } else {
                calcPointMap[name].inScreen = true
            }
        }
        //将所有在屏幕外
        for (let key in calcPointMap) {
            const {inScreen, pointItem} = calcPointMap[key]
            if (!inScreen) {
                const {lat, lng, icon} = pointItem
                const directionVector = {x: lng - center, y: lat - center.lat}
                const xVector = {x: 1, y: 0}
                const angle = calcVectorAngle(xVector, directionVector)
                guideUIAry.push({angle, icon, lat, lng})
            }
        }
    }
```

搭建GuideMarker样式
```vue
<script lang="ts" setup>

</script>

<template>
  <div class="guide-marker-ui">
    <div v-for="item in 1" :key="item" class="guide-marker-item">
      <div class="marker-bg">
        <div class="arrows-icon"></div>
      </div>
      <div class="marker-img-container">
        <img :src="item.icon" alt="" class="item-icon">
      </div>
    </div>
  </div>
</template>

<style scoped>
.guide-marker-ui{
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 999;
  pointer-events: none;
}
.guide-marker-item{
  pointer-events: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 53px;
  height: 53px;
  cursor: pointer;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.marker-bg{
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
  border: 1px solid hsla(0, 0%, 100%, 0.3);
}
.arrows-icon{
  position: absolute;
  left: -19px;
  top: 15px;
  width: 19px;
  height: 26px;
  background-image: url("../assets/images/ui/guide-arrow.png");
  background-size: cover;
}
.marker-img-container{
  position: relative;
  width: 48px;
  z-index: 999;
  height: 48px;
}
.item-icon{
width: 100%;
}
</style>
```
GuideMarkerUI接收绘制UI事件
安装events库，并新建event-manager类管理事件
在calcOutScreenPoints中发布计算完毕事件
```vue
<script lang="ts" setup>
import {EventManager} from "../ts/event-manager.ts";
import type {GuideUiItem} from '../ts/map-manager.ts'
import {ref} from 'vue'

const guideUiAry = ref()

EventManager.on('RenderMapGuideUI', onRenderMapGuideUI)

function onRenderMapGuideUI(data: GuideUiItem[]) {
  guideUiAry.value = data
}
</script>

```
> 之前文章中的q部分代码有BUG，注意识别

编写guideUI位移计算函数
```typescript
function calcOffsetStyle(item: any){
  let {innerWidth, innerHeight} = window
  let {angle} = item
  let marginTop
  let marginLeft
  
  if(angle > 0){
    marginTop = innerHeight / 2
    marginLeft = marginTop / Math.tan(angle)
  }else {
    marginTop = - innerHeight / 2
    marginLeft = marginTop / Math.tan(angle)
  }
  
  let finalX = `${innerWidth / 2 + marginLeft }`
  let finalY = `${innerHeight / 2 - marginTop}`
    
    //防止在外面
  finalX = Math.max(0, finalX)
  finalY = Math.max(0, finalY)
    
  finalX = Math.min(innerWidth - 53, finalX)
  finalY = Math.min(innerHeight - 53, finalY)

  console.log(finalY)
  return {
    transform: `translate(${finalX}px,${finalY}px)`
  }
}
```
坐标计算分析简图：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/1679167bfe0e338fcdead62ad87d85d.jpg)
实现效果：
![](https://picbed-1251050137.cos.ap-nanjing.myqcloud.com/20241009205050.png)
接着旋转指针
> 以上代码依然有BUG，有时间完善