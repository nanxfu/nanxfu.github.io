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

用 ```spin``` 乘 ```spinAngle``` 以旋转粒子

```js
    for (let i = 0; i < parameter.count; i++) {
    const i3 = i * 3
    const radius = Math.random() * parameter.radius
    const spinAngle = radius * parameter.spin
    const branchAngle = (i % parameter.branches) / parameter.branches * Math.PI * 2 
    position[i3] = Math.cos(branchAngle + spinAngle) * radius
    position[i3 + 1] = 0
    position[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius
}
```

<div align="center"> {% asset_img spin.png 400 %} </div>

## 4.添加随机性
添加 ```randomeness``` 属性
```js
parameter.randomness = 0.2
gui.add(parameters, 'randomness').min(0).max(2).step(0.001).onFinishChange(DreamFusion)
```
我们想让粒子随机的分布在分支周围，就要给每个轴创建一个随机值，并用它乘 ```radius``` 和 ```randomness```

```js
    for (let i = 0; i < parameter.count; i++) {
    const i3 = i * 3
    const radius = Math.random() * parameter.radius

    const spinAngle = radius * parameter.spin
    const branchAngle = (i % parameter.branches) / parameter.branches * Math.PI * 2

    const randomX = Math.random() * parameter.randomness * radius
    const randomY = Math.random() * parameter.randomness * radius
    const randomZ = Math.random() * parameter.randomness * radius
    position[i3] = Math.cos(branchAngle + spinAngle) * radius + randomX
    position[i3 + 1] = randomY
    position[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ
}
```
<div align="center"> {% asset_img random.png 400 %} </div>

注意，此时随机位置是从0到1，我们最好把随机值域改为[-0.5, 0.5]
```js
//...
    const randomX = (Math.random() - 0.5) * parameter.randomness * radius
    const randomY = (Math.random() - 0.5) * parameter.randomness * radius
    const randomZ = (Math.random() - 0.5) * parameter.randomness * radius
//...
```

但实际上，随机的粒子多了后会呈现整体变为一个“正方体”，我们需要的效果是中心多，四周少的效果。所以我们需要使用加权的方式获得非线性的随机数

<div align="center"> {% asset_img curve.png 400 %} </div>

添加 ```randomnessPower``` 属性

```js
parameter.randomnessPower = 3
gui.add(parameters, 'randomnessPower').min(1).max(10).step(0.001).onFinishChange(DreamFusion)
```

```js
//...
    const randomX = Math.pow(Math.random(), parameter.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)
    const randomY = Math.pow(Math.random(), parameter.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)
    const randomZ = Math.pow(Math.random(), parameter.randomnessPower) * (Math.random() < 0.5 ? 1 : -1)
//...
```

## 5.添加颜色

我们想让粒子从内到外做出一个渐变。添加 ```innerColor``` 和 ```outsideColor``` 属性

```js
parameter.innerColor = '#ff6030'
parameter.outsideColor = '#1b3984'

gui.addColor(parameter, "innerColor").onFinishChange(generateGalxy)
gui.addColor(parameter, "outsideColor").onFinishChange(generateGalxy)
```

然后再PointMaterial中开启 ```vertexColors```

```js
    material = new THREE.PointsMaterial({
        size: parameter.size,
        sizeAttenuation: true,
        depthWrite: false,
        blending: THREE.AdditiveBlending,
        vertexColors: true
    })
```
给 ```geometry``` 添加 ```color``` 属性

```js
geometry = new THREE.BufferGeometry()
const position = new Float32Array(parameter.count * 3)
const colors = new Float32Array(parameter.count * 3)

for(let i = 0; i < parameter.count ; i++){
    //...
    //color
    colors[i3] = 1
    colors[i3] = 0
    colors[i3] = 0
    geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
}

```
<div align="center"> {% asset_img red.png 400 %} </div>

然后分别给内部颜色与外部颜色建立 ```Color``` 实例

```js
const generateGalaxy = () => {
    const  colorInside = new THREE.Color(parameter.insideColor)
    const  colorOutside = new THREE.Color(parameter.outColor)
}
```

创建第三个 ```Color``` 实例，并使用 ```lerp()```进行插值
```js
const mixedColor = colorInside.clone()  //防止原色污染
mixedColor.lerp(colorOutside, radius / parameters.radius)
//...
//color
colors[i3] = mixedColor.r
colors[i3] = mixedColor.g
colors[i3] = mixedColor.b

```
<div align="center"> {% asset_img lerp.png 400 %} </div>