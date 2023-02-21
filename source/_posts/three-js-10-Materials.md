---
title: three.js_10-Materials
date: 2023-02-20 16:17:37
tags:
- three.js
- web
- javascript
index_img: /img/threejs.png
---
## 1.什么是 Materials
**Materials**用来给几何体上每个可见的像素染色。这种染色算法在程序里叫做 **Shaders**. 我们不需要自己手写 **Shaders** 可以使用内置的 materials

## 2.使用 Materials

### 2.1 创建场景

我们创建三种不同的几何体(spehre, plane, torus)来练习Materials的使用
```javascript

const material = new THREE.MeshBasicMaterial({color: 0xff0000})

const sphere = new THREE.Mesh(
    new THREE.SphereBufferGeometry(0.5, 16, 16),
    material
)
sphere.position.x = - 1.5

const plane = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(1, 1),
    material
)


const torus = new THREE.Mesh(
    new THREE.TorusBufferGeometry(0.3, 0.2, 16, 32),
    material
)
torus.position.x = 1.5

scene.add(sphere, plane, torus)
```
以上代码创建了三个几何体，并设置了color。效果如图

<div align="center"> {% asset_img material.png 400 %} </div>

接下来，我们让几何体自己旋转以便观察到各个部位

```javascript
const clock = new THREE.Clock()
const tick =() =>{
    const elapsedTime = clock.getElapsedTime()
    sphere.rotation.y = 0.1 * elapsedTime
    plane.rotation.y = 0.1 * elapsedTime
    torus.rotation.y = 0.1 * elapsedTime

    sphere.rotation.x = 0.15 * elapsedTime
    plane.rotation.x = 0.15 * elapsedTime
    torus.rotation.x = 0.15 * elapsedTime
}
```

然后加载纹理
```javascript
const textureLoader = new THREE.TextureLoader(loadingManager)
const doorColorTexture = textureLoader.load('/textures/door/color.jpg')
const doorAlphaTexture = textureLoader.load('/textures/door/alpha.jpg')
const doorColorTexture = textureLoader.load('/textures/door/color.jpg')
const doorColorTexture = textureLoader.load('/textures/door/color.jpg')
const matcapTexture = textureLoader.load('/textures/matcaps/1.png')
const gradientTexture = textureLoader.load('/textures/gradients/1.png')
```
### 2.2 MeshBasicalMaterial
大多数的 **Material** 属性都能用两种设置，例如：
```javascript
//实例化时声明
const material = new THREE.MeshBasicMaterial({map: doorColorTexture})

//实例化后声明
const material = new THREE.MeshBasicMaterial()
material.map = doorColorTexture
```
<div align="center"> {% asset_img setPro.png 400 %} </div>

 1. **map** 属性会将 **texture** 设置在几何体的表面.

 2. **color** 属性会给几何体的表面设置颜色，一旦被实例化后。 **color** 属性就变为 ```Color```**``` 类型，所以如果在实例化 **Material**  后指定物体的颜色可采用以下两种方法

```javascript
// 1.采用Color.set 方法
material.color.set('#ff00ff')

// 2.实例化一个 Color 类
material.color = new THREE.color('#ff00ff')
```
 3. **wireframe** 属性会展示组成几何体的三角形
 4. **opacity** 控制几何体的透明度。在这之前我们需要设置 ```transparent = true``` 
 5. **alphaMap** 控制材质的透明度。
```javascript
material.transparent = true
material.alphaMap = doorAlphaTexture
```
<div align="center"> {% asset_img alpha.png 400 %} </div>

 6. **side** 可以让你决定哪个面能够显示
    - THREE.FrontSide(default)
    - THREE.BackSide
    - THREE.DoubleSide

```javascript
material.side = THREE.DoubleSide //会造成GPU压力增加
```

### 2.3MeshNormalMaterial

**MeshNormalMaterial** 会展示一个非常炫酷的紫色效果
> normals n.法线

```javascript
const material = new THREE.MeshNormalMaterial()
```

<div align="center"> {% asset_img normal.png 400 %} </div>

实际上 **Normal** 并不仅仅是给几何体添加上了炫彩的颜色，而是给 **Geometry** 自动添加了法线方向属性，以供于光照、反射的计算

**MeshNormalMaterial** 与 **MeshBasicMaterial** 一样拥有 ```wireframe, transparent, opacity, side``` 属性。但会额外多一个 ```flatShading``` 属性。

**flatShading（平坦着色）** 是指通过三角形三顶点的坐标计算出整个三角形的法向量。 这样就导致相邻两个三角形的法向量差别很大，所以就能看到明显的三角形的边。
```javascript
material.flatShading = true
```

<div align="center"> {% asset_img flat.png 400 %} </div>

与之相对应的是“Smooth Shading”：先计算三角形三顶点处的法向量，然后将这三个法向量线性插值的结果作为整个三角形的法向量。这样相邻两个三角形的法向量差别就会小很多。

来两张图感受一下两种着色,前者为 flat 着色，后者为 smooth 着色
<div align="center">{% asset_img flat.bmp 400 %} {% asset_img smoth.bmp 400 %}</div>
图源：

[平坦着色（Flat Shading）和平滑着色（Smooth Shading)](https://blog.csdn.net/libing_zeng/article/details/60760296)

**MeshNormalMaterial**通常是用来调试法线的，且它可以直接用在你的工程中

### 2.4 MeshMatcapMaterial
**MeshMatcapMaterial** 会使用法线作为计算应用在几何体上matcap材质正确着色的参考。
<div align="center">{% asset_img matcap.png %}</div>

```javascript
//设置matcap属性应用matcap
const material = new THREE.MeshMatcapMaterial()
material.matcap = matcapTexture
```
注意设置mapcap后几何体有光照阴影效果
<div align="center">{% asset_img matcaps.png %}</div>

[下载matcaps](https://github.com/nidorx/matcaps)