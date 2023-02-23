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

### 2.5 MeshDepthMaterial

**MeshDepthMaterial** 会在摄像头距离几何体近端染上白色，远端染上黑色

<div align="center">{% asset_img white0.png 400 %} {% asset_img black.png 400 %}</div>

可以用来模拟光照

### 2.6 MeshLambertMaterial

**MeshLambertMaterial** 可以被光照的效果影响到
首先增加灯光来看下效果
```javascript
const ambienLight = new THREE.AmbientLight(0xffffff, 0.5)
scene.add(ambienLight)

const pointLight = new THREE.PointLight(0xffffff, 0.5)
pointLight.position.x = 2
pointLight.position.y = 3
pointLight.position.z = 4

scene.add(pointLight)

const material = new THREE.MeshLambertMaterial()

```

左图是有光照的效果，右图是没有光照的效果
<div align="center">{% asset_img light.png 400 %} {% asset_img none.png 400 %}</div>

**MeshLambertMaterial** 具有很好的性能表现，但物理效果还有更好的 Material

### 2.7 MeshLambertMaterial

**MeshLambertMaterial 与 ** **MeshLambertMaterial** 表现效果类似，但视觉光反射上会更好。

可以通过 ```shininess``` 和 ```specular``` 调整光反射效果

```javascript
material.shininess = 100    //值越大光反射效果越明显
material.specular = new THREE.Color(0x1188ff)
```

<div align="center">{% asset_img specular.png 400 %} </div>

### 2.8 MeshToonMaterial

**MeshToonMaterial** 效果与 **MeshLambertMaterial** 类似，但会有一种漫画滤镜

<div align="center">{% asset_img Toon.png 400 %} </div>

要添加更多的色彩渐变，可以使用 gradientMap 赋予渐变材质。下图为渐变材质示例
<div align="center">{% asset_img gradient.png 400 %} </div>

```javascript
material.gradientMap = gradientTexture
```
添加渐变材质后色彩分割感就小时了，因为渐变单位变小了，并且 magFilter 会尝试使用 mipmapping 来优化视觉（模糊）效果
<div align="center">{% asset_img gMap.png 400 %} </div>
如果我们想改变这种情况，可以给 THREE.NearestFilter 设置 minFilter 和 magFilter

```javascript
grdientTexture.minFilter = THREE.NearestFilter
grdientTexture.magFilter = THREE.NearestFilter

grdientTexture.generateMipmaps = false //采用了最近点采样方法后建议将 mipmapping生成关掉，这样会优化性能
```

<div align="center">{% asset_img magFilter.png 400 %} </div>

### 2.9 MeshStandardMaterial

**MeshStandardMaterial** 采用了 PBR 标准，使得视觉上光照更接近物理效果。与 **MeshLambertMaterial** 和 **MeshPhongMaterial** 类似，它支持很多与光照有关的算法，且额外提供了类似 ```roughness``` 和 ```metalness``` 这样的参数

```javascript
const material = new MeshStandardMaterial

material.metalness = 0.45
material.roughness = 0.65
```

<div align="center">{% asset_img stand.png 400 %} </div>


你可以使用 **map** 属性添加普通材质，也可以用 aoMap(ambient occlusion map) 添加纹理中提供的阴影,当然仅设置 ```aoMap``` 还不够，我们还需要提供另一组 UV 数组坐标： ```uv2```

> uv坐标提供纹理如何应用于材质

<div align="center">{% asset_img ao.png 400 %} </div>

```javascript
plane.geometry.setAttribute(
    'uv2', 
    new THREE.BufferAttribute(plane.geometry.attributes.uv.arry, 2))
    //因为阴影材质uv与纹理材质一致，所以直接使用plane.geometry.attributes.uv.arry
```
左图为未设置 ```aoMap``` 右图未设置了 ```aoMap```
<div align="center"> {% asset_img noaomap.png 400 %} {% asset_img aomap.png 400 %}</div>

aoMap 添加了 doorAmbientOcclusionTexture 后就可以使用 aoMapIntensity 控制阴影深度

使用 ```displacementMap``` 添加了高度材质后就可以做出浮雕效果

```javascript
material.displacementMap = doorHeightTexture
```
<div align="center">{% asset_img dis.png 400 %} </div>

我们发现在使用了 displacement 后几何体就变得很奇怪，这是因为几何体提供的定点数太少了，而 displacement值又太大

所以我们需要用
 ```displacementScale``` 降低 ```displacement``` 效果, 并给几何体添加细分度。

```javascript
material.displacementScale = 0.05
//...
new THREE.PlaneBufferGeometry(1, 1, 100, 100)
//...
```
<div align="center">{% asset_img scale.png 400 %} </div>

除了通过 matalness 和 roughness 添加光照效果，我们也可以用 ```metalnessMap``` 和 ```roughnessMap``` 来添加光照效果

当同时使用 **matalness** 与 **metalnessMap**、 **roughness** 与 **roughnessMap** 后效果会很奇怪，因为他们都会起作用从而造成效果叠加

**normalMap**会改变法线朝向并给表面添加更多细节。左图是没有 **normalMap** 的预览，右图是添加 **normalMap** 后的预览
<div align="center">{% asset_img nonor.png 400 %} {% asset_img detail.png 400 %}</div>

我们也可以使用 ```normalScale: Vector2``` 改变法线性质

```javascript
material.normalScale.set(0.5, 0.5)
```

最后 ，我们可以使用 ```alphaMap``` 属性控制 **alpha**，同时不要忘记将 ```transparent``` 设置为 true

### 2.10 MeshPhysicalMaterial

**MeshPhysicalMaterial** 和 **MeshStandardMaterial** 一样，不过支持透明涂层效果

<div align="center"> {% asset_img phy.png 400 %}</div>

### 2.11 PointsMaterial

用于粒子的材质

## 3.ENVIRONMENT MAP

环境贴图是周围场景的图片，它不仅可以用来做反射和折射效果，也能做光照效果。通常环境贴图不止有一张，而是很多张的集合(左右上下前后)

要使用环境贴图需要用 CubeTextureLoader 在 material 实例化之前加载完成

<div align="center"> {% asset_img loader.png 400 %}</div>

获取好看的环境贴图，我们可以在 [HDRI HAVEN](https://polyhaven.com/) 下载，不过从这上面下载下来的图片是 HDRIs(High Dynamic Range Imaging) 图片, 并不是 cube maps。

所以我们可以使用 [在线工具](https://matheowis.github.io/HDRI-to-CubeMap/) 将 HDRIs 转换为 cube maps