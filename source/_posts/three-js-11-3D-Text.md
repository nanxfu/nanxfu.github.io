---
title: 'three.js-11:3D-Text'
index_img: /img/threejs.png
date: 2023-02-24 21:11:33
tags:
- three.js
- web
- javascript
---
在这本节中，我们将实现 ilithya的[3D文本](https://www.ilithya.rocks/)特效。如下图所示
<div align="center"> {% asset_img 3Dtext.png 400%} </div>

## 1.字体加载
我们首先使用 ```TextBufferGeometry``` 在场景中创建一个3D文本。要设定字体需要使用[typeface](https://gero3.github.io/facetype.js/)将字体转换为指定格式。或者使用 Three.js 提供的字体。
```javascript
import typefaceFont from 'three/example/fonts/helvetiker_regular.typeface.json'
```

> 要练习通过静态资源加载字体的方法可以采用以下步骤：
> - 打开 /node_modules/three/example/fonts/
> - 将 helvetiker_regular.typeface.json 文件和 LICENSE 文件粘贴到 /static/fonts 文件夹下
> - 然后就可以通过访问 /font/helvetiker_regular.typeface.json URL 加载字体资源了

加载字体需要使用 ```FontLoader```
```javascript
/**
 * Fonts
 */
const fontLoader = new THREE.FontLoader()
fontLoader.load(
    '/font/helvetiker_regular.typeface.json',
    (font)=>{
        console.log('font loaded')
    }
)

//注意，此处与材质加载器不同，材质加载器加载后会得到材质，而字体加载器加载后需要在回调函数里获取字体
const texture = textureLoader.load()    //得到材质
```

## 2.文本创建

```javascript
fontLoader.load(    //加载字体
    '/font/helvetiker_regular.typeface.json',
    (font)=>{
        //创建文本顶点数据
        const textGeometry = new ThREE.TextBufferGeometry(
            "Hello Three.js",
            {
                //font config
                font,
                size: 0.5,
                height: 0.2,
                curveSegments: 12,
                bevelEnable: true,
                bevelThickness: 0.03,
                bevelSize: 0.02,
                bevelOffset: 0,
                bevelSegments: 5
            }
        )
        //创建材质
        const textMaterial = new THREE.MeshBasicMaterial()
        //创建网格
        const text = new THREE.Mesh(textGeometry, textMaterial)
        //加入场景
        scene.add(text)
    }
)
```
注意创建文字几何体是非常耗费资源的，因为一段文字会包含许多的几何体（三角形、圆形），所以我们尽量通过减少 ```curveSegments``` 和 ```bevelSegments```的值来保持文本几何体是低模的

### 2.1 文本居中
居中文本的方式有很多种。
1. BOUNDING
**BOUNDING** 就是几何体的边界信息，意味着几何体在空间中占用的大小，他可能是一个立方体，也可能是一个球体
<div align="center"> {% asset_img BOUNDING.png 400%} </div>。它可以让 **THree.js** 计算哪些物体应该在视锥范围内显示。所以我们可以用 **bounding** 让文本居中

默认情况下， **THree.js** 使用球面 **BOUNDING**，我们可以用 ```computeBoundingBox()``` 计算 **Bounding**

```javascript
textGeometry.computeBoundingBox()
console.log(textGeometry.boundingBox)   //注意，在使用textGeometry.boundingBox之前需要先使用computeBoundingBox
```
**textGeometry.boundingBox** 的结果是 ```Box3``` 实例，具有 **min** 和 **max属性**。如果你观察上面打印出来的结果会发现即使文本是从原点开始创建的但 **min** property并不是0，这是因为 **bevelThickness** 和 **bevelSize**的影响。

我们接下来使用 ```translate``` 方法来移动整个几何体的顶点，而不是移动 mesh。

```javascript
textGeometry.translate(
    - textGeometry.boundingBox.max.x * 0.5,
    - textGeometry.boundingBox.max.y * 0.5,
    - textGeometry.boundingBox.max.z * 0.5
)
```
<div align="center"> {% asset_img translate.png 400%} </div>

表面看起来移动在中心了，实际上并没有。因为bevelThickness和bevelSize的影响。所以还需微调以下
```javascript
textGeometry.translate(
    - ( textGeometry.boundingBox.max.x - 0.02) * 0.5,
    - ( textGeometry.boundingBox.max.y - 0.02 )* 0.5, // -0.02是因为 bevelsize（从字体内部到外部的距离）
    - ( textGeometry.boundingBox.max.z - 0.03 ) * 0.5 // -0.03是因为bevelThickness
)
```

这样才能是真正的居中

### 2.2 使用 center() 居中
实际上three.js内置了一个 center() 函数来使文字居中
```javascript
textGeometry.center()
```
center函数也是基于BOUNDING实现的
## 3.添加MATCAT
我们使用 **MeshMatcapMaterial** 应用 matcaps(模拟光照阴影)，你可以在这里[下载matcaps](https://github.com/nidorx/matcaps)

```javascript
const textureLoader = new THREE.TextureLoader()
const matcapTexture = textureLoader.load('/textures/matcaps/1.png')
```
然后使用 **MeshMatcapMaterial** 替换 **MeshBasicMaterial**，并使用 matcap

```javascript
const textMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})

```
<div align="center"> {% asset_img matcap.png 400%} </div>

## 4.添加装饰几何体
在场景中创建完3D文本后我们可以添加100个几何体用来装饰周围环境，首先我们用一个低效的方法实现创建100个几何体
```javascript
for(let i = 0; i<100; i++){
    const donutGeometry = new THREE.TorusBufferGeometry(0.3, 0.2, 20, 45)
    const donutMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})
    const donut = new THREE.Mesh(donutGeometry, donutMaterial)
    
    donut.position.x = (Math.random() - 0.5) * 10
    donut.position.y = (Math.random() - 0.5) * 10
    donut.position.z = (Math.random() - 0.5) * 10
    
    scene.add(donut)
}
```
效果如下
<div align="center"> {% asset_img donut.png 400%} </div>
所有的甜甜圈都朝向一个地方,且大小相同，所以我们最好给甜甜圈旋转以下，并缩放甜甜圈

```javascript
//...
donut.rotation.x = Math.random() * Math.PI
donut.rotation.y = Math.random() * Math.PI

const scale = Math.random()
donut.scale.x = scale
donut.scale.y = scale
donut.scale.z = scale

//或者使用 donut.scale.set(scale, scale, scale)
scene.add(donut)
//...
```

## 5.优化
实际上以上的性能非常低效，我们可以用 ```console.time(donuts)``` 看一下性能消耗

```javascript
console.time("donuts")
for (let i= 0; i < 100 ; i++)
{
    //...
}
console.time("donuts")  //44...ms
```
可以看出是非常消耗性能的，所以我们有一个优化思路：我们可以用同样的 **geometry** 和 **material** 在多个 **Meshes** 上
```javascript
console.time("donuts")
const donutGeometry = new THREE.TorusBufferGeometry(0.3, 0.2, 20, 45)
const donutMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})

for(let i = 0; i < 100; i++){
    //...
}
console.time("donuts") //1.... ms
```
可以看到优化非常的明显

此外，我们还可以让 **donut** 和 **text** 使用同一个 **material**，而不必再为 **donut** 实例化一个 **material**