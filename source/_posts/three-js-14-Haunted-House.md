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

2. 然后创建墙
```javascript
const walls = new THREE.Mesh(
    new THREE.BoxGeometry(4, 2.5, 4),
    new THREE.MeshStandardMaterial({ color: '#ac8e82' })
)
walls.position.y = 1.25 //避免防止墙一半在地板下面
house.add(walls)
```

3. 创建一个金字塔性的房顶我们使用 **ConeBufferGeometry** 。调整它的细分程度即可变为金字塔性质

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
4. 用平面创建一个门，并设置上材质

```javascript
const door = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(2, 2),
    new THREE.MeshStandardMaterial({ color: '#aa7b7b' })
)
door.position.y = 1
door.position.z = 2 + 0.01 //注意平面竞争（z-fighting
house.add(door)
```

5. 添加草丛
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

6. 添加坟墓
手动生成大量的坟墓会很耗时，所以我们就自动的生成并防止坟墓

首先我们创建一个坟墓组

```javascript
const graves = new THREE.Group()
scene.add(graves)


```

