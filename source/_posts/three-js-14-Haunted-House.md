---
title: three.js-14-Haunted-House
index_img: /img/threejs.png
date: 2023-03-04 22:04:17
tags:
- three.js
- web
- javascript
---
在本节中，我们要造一个鬼屋。用Three.js的基本几何体。并以米作为基本单位

我们先建造基本场景：一个地板、一个球体、一些灯光、不必有阴影、一个Dat.GUI面板。[点击下载初始包](https://threejs-journey.com/assets/lessons/16/17-haunted-house.zip)
<div align="center"> {% asset_img basic.png 600 %} </div>
然后我们把球体移除

## 1.创建房子

1. 建立房子我们先创建一个Group
```javascript
const house = new THREE.Group()
scene.add(house)
```

## 2. 然后创建墙
```javascript
const walls = new THREE.Mesh(
    new THREE.BoxGeometry(4, 2.5, 4),
    new THREE.MeshStandardMaterial({ color: '#ac8e82' })
)
walls.position.y = 1.25 //避免防止墙一半在地板下面
house.add(walls)
```

## 3. 创建一个房顶

我们使用 **ConeBufferGeometry** 。调整它的细分程度即可变为金字塔性质

```javascript
const roof = new THREE.Mesh(
    new THREE.ConeGeometry(3.5, 1, 4),
    new THREE.MeshStandardMaterial({ color: '#b35f45' })
)
roof.rotation.y = Math.PI * 0.25
roof.position.y = 2.5 + 0.5
house.add(roof)
```
<div align="center"> {% asset_img roof.png 600 %} </div>
## 4. 用平面创建一个门，并设置上材质

```javascript
const door = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(2, 2),
    new THREE.MeshStandardMaterial({ color: '#aa7b7b' })
)
door.position.y = 1
door.position.z = 2 + 0.01 //注意平面竞争（z-fighting
house.add(door)
```

## 5. 添加草丛
因为草丛都是一样的材质，形状。所以我们不是创建三个草丛几何体，而是创建三个网格
```javascript
const bushGeometry = new THREE.SphereBufferGeometry(1, 16, 16)
const bushMaterial = new THREE.MeshStandardMaterial({color:'#89c854'})

const bush1 = new THREE.Mesh(bushGeometry, bushMaterial)
bush1.scale.set(0.5, 0.5, 0.5)
bush1.position.set(0.8, 0.2, 2.2)

const bush2 = new THREE.Mesh(bushGeometry, bushMaterial)
bush2.scale.set(0.25, 0.25, 0.25)
bush2.position.set(1.4, 0.1, 2.1)

const bush3 = new THREE.Mesh(bushGeometry, bushMaterial)
bush3.scale.set(0.4, 0.4, 0.4)
bush3.position.set(- 0.8, 0.1, 2.2)

const bush4 = new THREE.Mesh(bushGeometry, bushMaterial)
bush4.scale.set(0.15, 0.15, 0.15)
bush4.position.set(- 1, 0.05, 2.6)
house.add(bush1, bush2, bush3, bush4)
```

<div align="center"> {% asset_img bush.png 600 %} </div>

## 6. 添加坟墓
手动生成大量的坟墓会很耗时，所以我们就自动的生成并防止坟墓

首先我们创建一个坟墓组

```javascript
const graves = new THREE.Group()
scene.add(graves)

const graveGeometry = new THREE.BoxBufferGeometry(0.6, 0.8, 0.2)
const graveMaterial = new THREE.MeshStandarMaterial({color: '#b2b6b1'})

//范围内随机生成坟墓
for(let i = 0; i< 50; i++){
    const angle = Math.random() * Math.pi * 2  
    const radius = 3 + Math.random() * 6
    const x = Math.sin(angle) * radius
    const z = Math.cos(angle) * radius
    
    const grave = new THREE.Mesh(graveGeometry, graveMaterial)
    grave.position.set(x, 0.3, z)
    graves.add(grave)
}
```

<div align="center"> {% asset_img graves.png 600 %} </div>

然后增加点随机旋转

```javascript
//...
grave.rotation.y = (Math.random() - 0.5) * 1
grave.rotation.z = (Math.random() - 0.5) * 1
```

<div align="center"> {% asset_img rotate.png 600 %} </div>

## 7. 灯光
然后调整灯光效果，并给一种蓝色的氛围色
```javascript
const ambientLight = new THREE.AmbientLight('#b9d5ff', 0.12)

//...

const moonLight = new THREE.DirectionalLight('#b9d5ff', 0.12)
```

<div align="center"> {% asset_img dim.png 600 %} </div>

然后给门加上点光源

```javascript
//Door light

const doorLight = new THREE.PointLight('ff7d46', 1, 7)
doorLight.position.set(0, 2.2, 2.7)
house.add(doorLight)
```

<div align="center"> {% asset_img pointlight.png 600 %} </div>

## 8.雾

three.js提供了Fog类添加雾,要激活雾就在 ```scene```上添加fog属性

```javascript
//Fog
const fog = new THREE.Fog('#ff0000', 2, 6)
scene.fog = fog
```
<div align="center"> {% asset_img fog.png 600 %} </div>

雾有三个属性：
- **color**
- **near** ——雾距离相机的距离单位，开始的位置
- **far** ——雾完全透明的距离

  调整一下颜色、近端和远端
```javascript
const fog = new THREE.Fog('#262837', 1, 15)
```

<div align="center"> {% asset_img fogad.png 600 %} </div>

当镜头推远时会发现背景有很强的违和感，我们需要改变```renderer```的 clearColor 和雾一样的颜色

```javascript
renderer.setClearColor('#262837')
```

<div align="center"> {% asset_img bg.png 600 %} </div>

这样一来背景和雾的颜色就一样了

## 9.添加材质
添加所有的材质
```javascript
const doorColorTexture = textureLoader.load(' /textures/door/color.jpg' )
const doorAlphaTexture = textureLoader.load( ' /textures/door/alpha.jpg' )
const doorAmbdientocclusionTexture = textureLoader.load('/textures/door/ambientOcclusion.jpg')
const doorHeightTexture = textureLoader.load( '/textures/door/height.jpg' )
const doorNormalTexture = textureLoader.load('/textures/door/normal.jpg')
const doorMetalnessTexture = textureLoader.load('/textures/door/metalness.jpg')
const doorRoughnessTexture = textureLoader.load('/textures/door/roughness.jpg')
```

然后将他们添加到door上

```javascript
const door = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(2, 2, 100, 100),
    new THREE.MeshStandarMaterial({
        map:doorColorTexture,
        transparent: ture,
        alphaMap: doorAlphaTexture,
        aoMap: doorAmbdientocclusionTexture,
        displacementMap: doorHeightTexture,
        displaceMentScale: 0.1,
        normalMap: doorNormalTexture,
        metalnessMap:doorMetalnessTexture,
        roughnessMa:doorRoughnessTexture
        
    })
)

door.geometry.setAttribute('uv2', 
    new THREE.Float32BufferAttribute(door.geometry.attributes.uv.arry, 2)
)
```

调整门的位置

```javascript
//...
new THREE.PlaneBufferGeometry(2.2, 2.2, 100, 100)
```
<div align="center"> {% asset_img door.png 600 %} </div>

然后添加墙的材质

```javascript
const bricksColorTexture = textureLoader.load('/textures/bricks/color.jpg')
const bricksAmbientOcclusionTexture = textureLoader.load('/textures/bricks/ambientOcclus,jpg')
const bricksNormalTexture = textureLoader.load('/textures/bricks/normal.jpg' )
const bricksRoughnessTexture = textureLoader.load( '/textures/bricks/roughness.jpg')

/// Walls

const walls = new THREE.Mesh(
    new THREE.BoxBufferGeometry(4, 2.5, 4),
    new THREE.MeshStandardMaterial({
        map: bricksColorTexture,
        aoMap: bricksAmbientOcclusionTexture,
        normalMap:bricksNormalTexture,
        roughnessMap: bricksRoughnessTexture
    })
)

walls.geometry.setAttribute(
    'uv2',
    new THREE.Float32BufferAttribute(walls.geometry.attributes.uv.arry, 2)
)
```
<div align="center"> {% asset_img wall.png 600 %} </div>

给草添加材质

```javascript

```