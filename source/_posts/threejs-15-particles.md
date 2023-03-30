---
title: 'threejs-15:particles'
index_img: /img/threejs.png
date: 2023-03-30 20:57:38
tags:
- three.js
- web
- javascript
---

# 1.创建粒子

创建粒子就像创建一个 ```Mesh``` ，需要有 **geometry(BufferGeometry)、material(PointsMaterial) 和 Points 实例(而不是一个 Mesh 实例)**

我们先创建一个 **sphereBufferGeometry 和 PointsMaterial**， **geometry**的每个坐标都会变成一个粒子

```javascript
const particlesGeometry = new THREE.SphereBufferGeometry(1, 32, 32)
const particlesMaterial = new THREE.PointsMaterial({
    size: 0.02,
    sizeAttenuation:true
})

const particles = new THREE.Points(particlesGeometry, particlesMaterial)
scene.add(particles)
```

通过设置 **PointsMaterial** 的 **size** 属性改变粒子的大小。通过设置 **sizeAttenuation** 属性决定粒子的透视关系（是否通过摄像机与粒子的距离实现远大近小）

<div align="center"> {% asset_img spoints.png 400 %} {% asset_img spoints2.png 400 %}</div>

## 2.自定义坐标

跟前几章一样，我们可以用 **BufferGeometry**上的 **position** 属性生成粒子,这些数据将会被送进gpu处理

```javascript
const particlesGeometry = new THREE.BufferGeometry()
const count = 500

const positions = new Float32Array(count * 3) //坐标是三个一组（xyz)

//填充数组

for(let i = 0; i < count * 3; i++){
    positions[i] = (Math.random() - 0.5 ) * 10
}

particlesGeometry.setAttribute(
    'position',
    new THREE.BufferAttribute(positions, 3)
)
const particlesMaterial = new THREE.PointsMaterial({
    size: 0.02,
    sizeAttenuation:true
})

const particles = new THREE.Points(particlesGeometry, particlesMaterial)
scene.add(particles)
```

效果如下。第二张图是 ```positions[i] = Math.random()``` 的效果
<div align="center"> {% asset_img dypoint.png 400 %} {% asset_img dypo.png 400 %}</div>

## 3.颜色

为粒子添加颜色只需改变材质的color属性

```javascript
particlesMaterial.color = new THREE.Color('#ff88cc')
```

<div align="center"> {% asset_img color.png 400 %}</div>

## 3.材质

我们同样可以给material添加材质

```javascript
const textureLoader = new THREE.TextureLoader()
const particleTexture = textureLoader.load('/texture/particles/2.png')

//...
particlesMaterial.map = particleTexture
```

<div align="center"> {% asset_img texture.png 400 %}</div>
<div align="center"> {% asset_img effect.png 400 %}</div>

要获取更多的粒子材质可以去 [kenney](https://www.kenney.nl/) 处获取

当你仔细看前景的粒子时，会发现前景粒子会以矩形的形式遮挡后面的粒子不是不规则形状遮挡,且内部区域应当透明的地方没有透明。而，所以我们需要添加 alphaMap 来设定透明区域

<div align="center"> {% asset_img hidding.png 400 %}</div>
```javascript
particlesMaterial.transparent = true
particlesMaterial.alphaMap = particleTexture
```

在使用透明贴图后仍会发现部分边缘区域会出现bug，这是因为创建粒子时他们重叠了导致的。解决方法也有很多

1.使用 **alphatest**
alphatest 是一个从0到1的值，它让 webgl 根据像素的透明度，什么时候不应该渲染该像素。默认情况下它的值为0，意味着该像素永远都会被绘制
我们使用0.001来测试

```javascript
particlesMaterial.alphaTest = 0.01
```

设置 alphatest 后看起来效果好多了，但某些情况下任然会很怪

<div align="center"> {% asset_img alphatest.png 400 %}</div>

2.depth test

depth test