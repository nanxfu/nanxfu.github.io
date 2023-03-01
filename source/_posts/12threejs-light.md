---
title: '12threejs:light'
index_img: /img/threejs.png
date: 2023-03-01 12:04:19
tags:
- three.js
- web
- javascript
---

增加与上个文章相同的场景
<div align="center"> {% asset_img scene.png 400%} </div>
然后删除 AmbientLight 和 PointLight，当你移除灯光后会发现场景 **一片漆黑**，是因为 **MeshStandarMaterial** 需要灯光才能看见物体

## 1.AmbientLight

**AmbientLight** 应用了 Omni-directional lighting 效果，照来的灯光，所有面光照效果相同。没有阴影明暗灰度灯光反射变化

**AmbientLight具有两个属性**

- **color**
- **intensity**

> 注意添加灯光后需要将灯光添加到场景中

<div align="center"> {% asset_img ambient.png 400%} </div>

## 2.DirectionalLight

**DirectionalLight** 会模拟太阳光照效果（平行光）

它也有两个属性：color 和 intensity

<div align="center"> {% asset_img direc.png 400%} </div>

方向性光源的光会使得物体在灯光下有了明暗阴影变化。DirectionalLight 的方向是从光源位置指向世界坐标原点的

> 灯光的距离不会引起明暗阴影变化

## 3.HemisphereLight

HemisphereLight 与 AmbientLight类似。但比后者多了 groundColor 属性，这个光代表从**地面**射出来的光

<div align="center"> {% asset_img hemi.png 400%} </div>

**HeimisphereLight** 三个属性
- color(skyColor)
- groundColor
- intensity

<div align="center"> {% asset_img hemilight.png 400%} </div>

## 4.Point Light

PointLight 最接近现实灯光的照射效果。同时也比较费性能

<div align="center"> {% asset_img pointLight.png 400%} </div>

点光源会从一个光源发射光线，使得物体拥有明暗变化。它继承于 Object3D 对象，可以移动位置

默认情况下，光强不会随着距离衰减。我们可以通过 distance 和 decay 属性控制光线衰减效果。在 distance 距离内，光线强度一样。在distance距离之外，会以decay为单位开始衰减

## 5.Rect Area Light

**Rect Area Light** 会像摄影灯光一样在一个矩形内发光。它是方向性光源(directional Light)与漫射光(diffuse light)的混合
<div align="center"> {% asset_img rect.png Rect2.png 400%} </div>

**Rect Area Light** 具有以下属性
- color
- intensity
- width
- height

RectAreaLight 仅对 MeshStandardMaterial 和 MeshPhysicalMaterial 有效

你可以旋转或移动 RectAreaLight。也可以用 lookAt(...) 使灯光朝向某个物体

## 6.SpotLight

SpotLight 像一个手电筒，它会形成一个光锥照射物体。具有以下属性

- color
- intensity
- distance
- angle
- penumbra
- decay

angle 是灯光照射范围，penumbra控制灯光边缘模糊程度，如果为0则会很尖锐

<div align="center"> {% asset_img spot.png 400%} </div>

如果需要旋转 SpotLight，我没需要添加一个 target 属性在场景中，然后移动它。SpotLight 会朝向 target(Object3D)

```javascript
scene.add(spotLight.target)
spotLight.target.position.x = -0.75
```

灯光都会耗费很多的性能，所以确保场景中只有尽可能少的灯光。

**最少性能消耗的灯光：**
- AmbientLight
- HemisphereLight
**中等程度消耗的灯光：**
- DirectionalLight
- PointLight
**最耗性能的灯光：**
- SpotLight
- RectAreaLight

## 7.BAKING
当需要很多的灯光效果时可以采用烘焙技术减少性能消耗。这个效果可以在3D软件中实现。

这样做的缺点就是我没不能移动灯光，而且要加载很多很大的材质

## 8.helpher

定位灯光可以采用helper辅助我们调整灯光效果

```javascript
const hemisphereLightHelper = new THREE.HemisphereLightHelper(hemisphereLight, 0.2)
scene.add(hemisphereLightHelper)
```

> SpotLightHelper 没有 ```size``` ，当我没改变了target的位置后需要在下一帧之前调用 update(...)函数
```javascript
const spotLightHelper = new THREE.SpotLightHelper(spotLight)
scene.add(spotLightHelper)
window.requestAnimationFrame(()=>{
    spotLightHelper.update()
})
```

RectAreaLightHelper 不是THREE变量下的一个成员，所以我们需要导入它

```javascript
import {RectAreaLightHelper} from 'three/examples/jsm/helpers/RectAreaLightHelper.js '
```

导入后还需要手动的更新 position 和 rotation 属性，并调用 update函数

```javascript
window.requestAnimationFrame(()=>{
    rectAreaLightHelper.position.copy(rectAreaLight.position)
    rectAreaLightHelper.quaternion.copy(rectAreaLight.quaternion)
    rectAreaLightHelper.update()
})
```