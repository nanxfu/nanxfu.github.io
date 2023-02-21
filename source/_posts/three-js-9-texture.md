---
title: three.js_9-texture
date: 2023-02-19 16:35:10
tags: 
 - three.js
 - web
 - javascript
index_img: /img/threejs.png
---
## 1.什么是材质?
材质就是覆盖在几何体上的图片，并且也能有不同的视觉效果（例如反射），不仅仅是贴图

## 2.材质种类
### 2.1 alpha
- alpha 材质是一个灰度图，白色的地方代表可见，黑色则代表不可见，如下图
<div align="center"> {% asset_img door\alpha.jpg 400%} </div>

### 2.2 HEIGHT
- HEIGHT 材质也是一个灰度图，代表高度。
- 一个像素点的灰度会提高或降低平面的高度。 需要提供细分信息（subdivision）如下图
<div align="center"> {% asset_img door\height.jpg 400%} </div>

### 2.3 NORMAL
- NORMAL 材质需要我们添加一些表面细节，如光照。
- 无需细分信息相比HEIGHT材质有更高的性能
<div align="center"> {% asset_img door\normal.jpg 400%} </div>

### 2.4 AMBIENT OCCLUSION
- AMBIENT OCCLUSION 也是一个灰度图，作用是增加物体的阴影
- 但它并不会模拟真实的物理光照效果（假阴影）。通常用来添加反差效果和更多细节
<div align="center"> {% asset_img door\ambientOcclusion.jpg 400%} </div>

### 2.5 METALNESS
- METALNESS 是一个灰度图。白色地方代表金属光泽，黑色代表无金属光照
- 大多用来展示光线反射（如镜子）
<div align="center"> {% asset_img door\metalness.jpg 400%} </div>

### 2.6 ROUGHNESS
- ROUGHNESS 是一个灰度图，通常与 METALNESS 配合使用。
- 白色代表粗糙，黑色代表平滑。
- 通常用来展示光线漫反射
例如用它来展示一个地毯（粗糙）或者金属（光滑）
<div align="center"> {% asset_img door\roughness.jpg 400%} </div>

### 2.7 PBR principles
这些种类的材质都需要遵循 **[PBR](https://zhuanlan.zhihu.com/p/33464301)** 规范
> ### PBR 规范
> PBR是基于物理表现的渲染，例如光照、金属光泽、粗糙度（实际上大多数技术都倾向于遵循现实世界的物理效果）
> PBR定义了一系列的算法用来模拟现实世界物体的渲染效果

## 3.使用材质
了解了几种材质类型后就可以开始使用材质了。首先使用一个由Paulo做的[门材质](https://3dtextures.me/2019/04/16/door-wood-001/),这是一个很有名的库
<div align="center"> {% asset_img door-texture.png 400%}</div>

### 3.1加载材质静态资源

使用材质首先需要解决的问题是引入材质图片，我们有两种方法引入
1. 可以使用 import 导入
```javascript
import imageSource from './color.jpg'
console.log(imageSource)    //http://localhost/images/c43f...jpg
```
2. 静态资源服务器
   我们可以在项目根目录的static目录下放置图片
<div align="center"> {% asset_img project.png 400%} </div>
然后就能在浏览器中直接访问到材质图片了
<div align="center"> {% asset_img project2.png 400%} </div>
以上方式皆基于webpack

3. Native Javascript
```javascript
const image = new Image()

image.onload = () => {
    console.log("image loaded")
}

image.src = '/textures/door/color.jpg'
```
缺点：当图片大，宽带小时会花费很长时间加载

### 3.2初始化材质
材质被加载进来后就可以开始初始化材质,并将材质应用在物体上了
```javascript
const image = new Image()

image.onload = () => {
    const texture = new THREE.Texture(image)    //初始化材质
    console.log("image loaded")
}

image.src = '/textures/door/color.jpg'
```
实际上这段代码可以使用 **texture.needsUpdate** 属性实现材质的自动更新，这样就可以避免因作用域问题需要手动维护texture的状态。
```javascript
const image = new Image()
const texture = new THREE.Texture(image)
image.onload = () => {
    texture.needsUpdate = true
    console.log("image loaded")
}

image.src = '/textures/door/color.jpg'
```
### TextureLoader
除了使用img初始化材质以外，还可以使用 **TextureLoader** 直接加载材质
```javascript
const textureLoader = new THREE.TextureLoader()
const texture = textureLoader.load('/textures/door/color.jpg')
const geometry = new THREE.BoxBufferGeometry(1, 1, 1)
const material = new Three.MeshBasicMaterial({map:texture}) //使用材质
const mesh = new THREE.Mesh(geometry, material)
```
**TextureLoader** 可以在第一个参数后添加三个function用来处理加载材质的生命周期
- load -当材质成功加载后触发
- progress -材质加载中触发
- error -材质加载失败触发

### LoadingManager
当加载字体、模型、材质时就需要一个全局的加载进度来告知用户哪些被加载完成了，这个功能已在three.js中集成，既**LoadingManager**
```javascript
const loadingManager = new THREE.LoadingManager()
const textureLoader = new THREE.TextureLoader(loadingManager)
const texture = textureLoader.load('/textures/door/color.jpg')
```
**LoadingManager**提供了许多事件可供监听,例如 **onStart**、**onLoaded**、**onProgress**、**onError**

```javascript
const loadingManager = new THREE.LoadingManager()
loadingManager.onStart = ()=>{
   console.log("onStart")
}
loadingManager.onLoaded = ()=>{
   console.log("onLoaded")
}
loadingManager.onProgress = ()=>{
   console.log("onProgress")
}
const textureLoader = new THREE.TextureLoader(loadingManager)
const texture = textureLoader.load('/textures/door/color.jpg')
```
实际上，在写完这些代码后你会发现仅仅触发 "onStart"和"onProgress" ，Loaded事件并未被触发/
<div align="center"> {% asset_img log.png %} </div>

### UV UNWRAPPER
了解 **UV Unwrapping** 之前先来了解以下 **UV map（贴图)** 和 **UV mapping(映射)**

UV Map 是包裹在3D模型上的材质，创建UV 贴图的过程就叫做UV Unwrapping(拆图)

<div align="center"> {% asset_img uv.png %} </div>

> U和V指的是2D空间的水平和垂直轴，因为X、Y、Z已经在3D空间中使用

UV mapping即是将 3D网格转换为2D信息，用于指定材质如何"贴在几何体上"

类似Three.js 内置的 ```BoxBufferGeometry``` UV 坐标已经被指定过，可以用 ```geometry.attributes.uv``` 访问

<div align="center"> {% asset_img coodinate.png %} </div>

可以看到在这个 ```Float32Array``` 里 ```itemSize = 2```，因为 UV 坐标是 2D 的

如果你创建了自己的几何体，你就需要自己指定 UV coordinates(坐标),当你使用 3D 建模软件建模时，UV Unwrapping 也是必不可少的一步

## 4.材质变换

### 4.1 重复
我们可以用 ```repeat``` 属性让材质重复，它是一个  ```Vector2``` 的属性
<div align="center"> {% asset_img beRepet.png 400 %} </div>

```javascript
const colorTexture = textureLoader.load('...')

colorTexture.repeat.x = 2
colorTexture.repeat.y = 3
```
<div align="center"> {% asset_img afRepet.png 400 %} </div>

注意默认情况下，材质不会重复，采用最后一个材质像素拉伸适应几何体，这也是为什么上面设置 ```repet``` 属性后的怪异行为。我们可以给 ```wrapS``` 和 ```wrapT``` 属性设置 ```Three.RepeatWrapping```改变重复方式 

```javascript
colorTexture.wrapS = Three.RepeatWrapping
colorTexture.wrapT = Three.RepeatWrapping
```

<div align="center"> {% asset_img wrap.png 400 %} </div>

我们也可以使用 ```THREE.MirroredRepeatWrapping``` 达到重复的目的
```javascript
colorTexture.wrapS = Three.MirroredRepeatWrapping
colorTexture.wrapT = Three.MirroredRepeatWrapping
```

### 4.2 偏移
要偏移一个材质使用 ```offset``` 属性设置，这个属性也是一个 ```Vector2``` 的类型

```javascript
colorTexture.offset.x = 0.5
colorTexture.offset.y = 0.5
```
<div align="center"> {% asset_img offsetxy.png 400 %} </div>

### 4.3 旋转
使用 ```rotation``` 属性旋转材质。注意单位是 rad(弧度)

<div align="center"> {% asset_img rotate.png 400 %} </div>

```javascript
colorTexture.rotation = 1
```

旋转时是按照 pivot point (轴心) 旋转的。可以使用 ```center``` 属性修改轴心，这个属性是 ```Vector2``` 类型的

```javascript
colorTexture.center.x = 0.5
colorTexture.center.y = 0.5
```

### 4.4 filtering and mipmapping

当你把 camera 移动至立方体的顶端至几乎看不见的时候就会发现材质变得模糊，这是因为 **filtering** 和 **mipmapping(多级渐远纹理)** 特性

**mipmapping** 是把一个材质宽高不断减半细分的技术，直到得到 1x1 像素的材质分量。这些材质分量会被送到 GPU 内部，并由 GPU 决定最适合显示的材质分量

这些过程会被 Three.js 和 GPU 自动接管，不过我们任然能改变两种算法来达到合适的显示效果

1. minification filtering
  当camera与纹理表面距离变的更远时就会发生这种情况，我们可以使用材质的 ```minFilter``` 属性来改变 filter. 它有下面六个取值
   - THREE.NearestFilter
   - THREE.LinearFilter
   - THREE.NearestMipmapNearestFilter
   - THREE.NearestMipmapLinearFilter
   - THREE.LinearMipmapNearestFilter
   - THREE.LinearMipmapLinearFilter(default)

2. Magnification filtering
 当camera与纹理表面距离变的更远时就会发生这种情况，我们可以使用材质的 ```magFilter``` 属性改变 filter. 它有两种取值
   - THREE.NearestFilter
   - THREE.LinearFilter (default)

当我们在 **minFilter** 使用 **THREE.NearestFilter** 时就不需要  mipmaps 了， 使用 ```colorTexture.generateMipmaps = false``` 来禁止生成 mipmaps

## 5.材质优化

材质图片的大小可能很大，所以我们可以使用 TinyPNG 做一个压缩