---
title: 'threejs-16:galaxy-generate'
index_img: /img/threejs.png
date: 2023-04-01 17:04:45
tags:
- three.js
- web
- javascript
---

## 1.准备工作
创建一个星系工厂函数
```javascript
/*
Galaxy generate
 */
const generateGalxy = () => {
    console.log("generate the galaxy")
}

```

创建一个 ```parameter``` 对象，将会包含星系所有的参数

```javascript
const parameter = {}

parameter.count = 1000 //1000个粒子
```

首先创建随机粒子
```javascript
const generateGalxy = () => {
    const geometry = new THREE.BufferGeometry()
    const position = new Float32Array(parameter.count * 3)
    
    for(let i = 0; i < parameter.count ; i++){
        const i3 = i * 3
        position[i3] = (Math.random() - 0.5) * 3
        position[i3 + 1] = (Math.random() - 0.5) * 3
        position[i3 + 2] = (Math.random() - 0.5) * 3
    }
    geometry.setAttribute('position', new THREE.BufferAttribute(position, 3))
}
```

创建 ```PointsMaterial``` 类并添加 ```size``` 属性

```javascript
parameter.size = 0.02
const generateGalxy = () => {
    //...
    const material = new THREE.PointsMaterial({
        size: parameter.size,
        sizeAttenuation: true,
        depthWrite: false,
        blending: THREE.AdditiveBlending
    })
    
    /*
    创建 Points
     */
    const points = new THREE.Points(geometry, material)
    scene.add(points)
}
```

<div align="center"> {% asset_img particles.png 400 %} </div>

为 tweaks 添加调试
```javascript
gui.add(parameters, 'count').min(100).max(100000).step(100).onFinishChange(DreamFusion)
gui.add(parameters, 'size').min(0.001).max(0.1).step(0.001).onFinishChange(DreamFusion)
```
每次更新tweaks时会使粒子重复生成造成内存飙升，所以我们需要处理一下这个问题：将 ```geometry```,```material```,```points``` 变量移到 ```generateGalaxy```函数外面

```javascript
let geometry = null
let material = null
let points = null

const generateGalaxy = () => {
    //...
    geometry = new THREE.BufferGeometry()
    
    //...
    
    material = new THREE.PointsMaterial({
        size: parameter.size,
        sizeAttenuation: true,
        depthWrite: false,
        blending: THREE.AdditiveBlending
    })
}
```

然后在赋值之前，我们先测试它们是否存在，若存在则使用 ```dispose()``` 方法销毁几何体和材质，并用 ```remove()``` 方法把点从场景中移除

```js
const generateGalaxy = () => {
    if(points != null) {
        geometry.dispose()
        material.dispose()
        scene.remove(points)
    }
}
```

<div align="center"> {% asset_img tw.png 400 %} </div>

## 2.造型

接下来创造一个螺旋星系
<div align="center"> {% asset_img galaxy.png 400 %} </div>

添加 ```radius``` 属性

```js
parameter.radius = 5    //星系半径
//..
gui.add(parameters, 'radius').min(0.001).max(20).step(0.001).onFinishChange(DreamFusion)
```

先从简单的开始：创建从中心向外的几条射线
```js
    for (let i = 0; i < parameter.count; i++) {
    const i3 = i * 3
    const radius = Math.random() * parameter.radius
    position[i3] = radius
    position[i3 + 1] = 0
    position[i3 + 2] = 0
}
```

<div align="center"> {% asset_img line.png 400 %} </div>

然后创建星系的其它分支

```js
//...
parameter.branches = 3
gui.add(parameters, 'branches').min(2).max(20).step(1).onFinishChange(DreamFusion)
//...
```

将粒子放在分支上

```js
    for (let i = 0; i < parameter.count; i++) {
    const i3 = i * 3
    const radius = Math.random() * parameter.radius
    const branchAngle = (i % parameter.branches) / parameter.branches * Math.PI * 2 
    position[i3] = Math.cos(branchAngle) * radius
    position[i3 + 1] = 0
    position[i3 + 2] = Math.sin(branchAngle) * radius
}
```

<div align="center"> {% asset_img branch.png 400 %} </div>

## 3.旋转星系

添加 ```spin``` 属性

```js
parameter.spin = 1
gui.add(parameters, 'spin').min(-5).max(5).step(0.01).onFinishChange(DreamFusion)
```
