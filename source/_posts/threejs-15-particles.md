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

设置 alphatest 后看起来效果好多了，但某些情况下任然会很怪。比如我们会在一个正方体中看见正方体后面的粒子

<div align="center"> {% asset_img alphatest.png 400 %} {% asset_img cube.png 400 %}</div>

2.depth test

在three.js中，depth test是一个用于控制深度测试的属性。深度测试是一种用于检查像素深度信息的技术，以确定在渲染过程中哪些像素应该被渲染。当depth test属性被启用时，它会启用深度测试功能，这意味着每个像素的深度值将与深度缓冲区中的深度值进行比较。

深度测试通常使用一个叫做深度函数（depth function）的函数来控制。深度函数是一个用于比较像素深度和深度缓冲区中深度值的函数，通常有以下几种选项：

LESS：像素的深度值小于深度缓冲区中的深度值时通过深度测试。
LEQUAL：像素的深度值小于或等于深度缓冲区中的深度值时通过深度测试。
EQUAL：像素的深度值等于深度缓冲区中的深度值时通过深度测试。
GREATER：像素的深度值大于深度缓冲区中的深度值时通过深度测试。
GEQUAL：像素的深度值大于或等于深度缓冲区中的深度值时通过深度测试。
NOTEQUAL：像素的深度值不等于深度缓冲区中的深度值时通过深度测试。
ALWAYS：始终通过深度测试。
NEVER：始终不通过深度测试。
使用depth test属性，你可以控制在渲染过程中使用哪种深度函数进行深度测试。以下是一个设置深度测试属性的示例：

```javascript
const material = new THREE.MeshBasicMaterial({
   color: 0xff0000,
   depthTest: true,
   depthFunc: THREE.LessEqualDepth,
});
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

在这个例子中，depth test属性被设置为true，意味着将启用深度测试。depthFunc属性被设置为THREE.LessEqualDepth，意味着将使用小于等于深度缓冲区深度值的像素通过深度测试。
3.depth write
在three.js中，depth write是一个用于控制深度缓冲区写入的属性。深度缓冲区是一个特殊的缓冲区，用于存储每个像素的深度信息，它通常用于实现深度测试，以确定在渲染过程中哪些像素应该被渲染。

当depth write属性设置为true时，深度信息将被写入深度缓冲区，这意味着每个像素的深度值将被记录下来。当它设置为false时，深度信息将不会被写入深度缓冲区，而只会执行深度测试，以便根据需要渲染像素。这意味着在这种情况下，即使像素的深度测试通过，它们的深度信息也不会被写入深度缓冲区。

使用depth write属性可以在一定程度上控制three.js场景的深度行为，以实现更好的可视化效果。例如，在渲染半透明对象时，你可能想要禁用depth write属性，以便后面的对象能够正确地显示在前面的对象之上，从而实现透明效果。

——made by chatgpt

4.blending
Blending（混合）是three.js中一个用于控制物体混合透明度的属性。当两个或多个物体重叠时，混合可以使它们看起来更自然和逼真。

在three.js中，混合可以通过设置材质的blending属性来控制。blending属性是一个包含源混合因子和目标混合因子的数组，它们用于计算片段的颜色和alpha值，以便将其混合到场景中。

源混合因子（source blending factor）指定了如何混合当前片段的颜色和alpha值，而目标混合因子（destination blending factor）指定了如何混合已经在场景中的颜色和alpha值。

在three.js中，可以使用以下混合模式：

THREE.NoBlending: 不混合，即完全覆盖。
THREE.NormalBlending: 普通混合模式，即使用默认的混合因子。
THREE.AdditiveBlending: 加法混合模式，即将源和目标颜色相加。
THREE.SubtractiveBlending: 减法混合模式，即将源和目标颜色相减。
THREE.MultiplyBlending: 乘法混合模式，即将源和目标颜色相乘。
THREE.CustomBlending: 自定义混合模式，可以自己指定混合因子。
以下是一个使用混合的示例：

```javascript
const material = new THREE.MeshBasicMaterial({
   color: 0xff0000,
   opacity: 0.5,
   transparent: true,
   blending: THREE.AdditiveBlending,
});
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

```

在这个例子中，材质的opacity属性被设置为0.5，使得物体半透明。transparent属性被设置为true，以启用透明度。blending属性被设置为THREE.AdditiveBlending，以使用加法混合模式。这将使场景中的物体看起来更加逼真和自然。

注意 使用blending会影响性能

<div align="center"> {% asset_img blending.png 400 %}</div>

## 4.随机颜色
我们可以给每个粒子不同的颜色。通过添加 **color** 属性即可控制颜色。color 是一个 vector3 (r,g,b)

```javascript
const position = new Float32Array(count * 3)
const colors = new Float32Array(count * 3)

for(let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5 ) * 10
    colors[i] = Math.random()
}
particlesMaterial.vertexColors = true   //注意，需在此为粒子设置这个属性
particlesGeometry.setAttribute(
    'position',
    new THREE.BufferAttribute(positions, 3)
)
particlesGeometry.setAttribute(
    'color',
    new THREE.BufferAttribute(colors, 3)
)
```

生成的vertex颜色会被 main material 的颜色影响，为了去除影响，我们可以注释掉这行
```javascript
//particlesMaterial.color = new THREE.Color('#ff88cc')
```

<div align="center"> {% asset_img colors.png 400 %} {% asset_img color2.png 400 %}</div>

## 5.制作动画

为粒子制作动画的方式用很多种
1.将粒子作为 object 使用

**Points** 类继承于 **Object3D** 类，所以我们可以移动，旋转，缩放粒子
例如，我们在 **tick** 函数里旋转粒子

```javascript
const tick = () => {
    const elapsedTime = clock.getElpasedTime()
    
    particles.rotation.y = elapsedTime * 0.2
}
```

2.改变 **attributes**

我们可以改变 ```particlesGeomerty.attributes.position.array``` 每个粒子坐标的值操控粒子（每三个值代表一个）
接下来看一个
```javascript
const tick = () => {
    //...
    for(let i = 0; i < count; i++){
        const i3 = i * 3
        const x = particlesGeometry.attributes.position.array[i3]
        particlesGeometry.attributes.position.array[i3 + 1] = Math.sin(elapsedTime + x) //改变y值
    }
    particlesGeomerty.attributes.position.needsUpdate = true    //需要设置这个属性
}
```

<div align="center"> {% asset_img i3.png 400 %}</div>
这样一来就可以看起sin波的粒子动画了。但是你应该避免这样实现动画，因为我们更新了上千个粒子，会造成电脑风扇狂转。这不是一个最佳的实现方案（使用shader）
<div align="center"> {% asset_img wave.png 400 %}</div>
