---
title: 'three.js-13:Shadow'
index_img: /img/threejs.png
date: 2023-03-02 20:54:41
tags:
- three.js
- web
- javascript
---
在添加光源后我们就可以看到物体上的明暗变化了，这个阴影实际叫做 **core shadows**，但是我们还没有投射到其他物体上的 **drop shadow**，本章将讲解如何获得 **drop shadows**

光线追踪实际是个很耗费资源的计算，开发者需要用一些小技巧来增强视觉体验

Three.js 中实现阴影是在每一次渲染的时候会假设每一个光源上有个“相机”来计算哪个地方应该会有阴影，最后作为阴影texture贴在物体上。
[原理示例](https://threejs.org/examples/?q=shadow#webgl_shadowmap_viewer)
<div align="center"> {% asset_img example.png 600 %} </div>

Three.js实际上采用的就是 shadow map 技术实现阴影，它基于渲染场景时在某个点处拍摄的**深度信息**来确定光线的可见性，从而确定哪些物体会被遮挡，哪些物体应该被照亮。

Shadow map的大小决定了阴影的质量和分辨率。较大的阴影贴图能够提供更高分辨率的阴影，但也需要更多的GPU资源来渲染。过小的阴影贴图可能会导致阴影出现锯齿状，或者出现无法预期的阴影效果。

一般来说，选择阴影贴图的大小取决于场景的大小和需要达到的渲染质量。通常可以通过试验和测试来找到适合特定场景的最佳大小


## 1.如何激活阴影

1. 在renderer上激活 shadow maps
```javascript
renderer.shadowMap.enabled = true
```
2.在Mesh上启用阴影。如果能投射阴影则启用```castShadow```,如果能接收阴影影响则启用```receiveShadow```
```javascript
sphere.castShadow = true

plane.receiveShadow = true
```

3.启动光源上的castShadow属性

注意，仅以下光源支持阴影
- PointLight
- DirectionalLight
- SpotLight

```javascript
directionalLight.castShadow = true
```

## 2.优化阴影
下图就是默认的阴影效果了，可以看到非常不好看，所以我们需要优化阴影
<div align="center"> {% asset_img example.png 400 %} </div>

我们可以通过光源的 ```shadow``` 属性来访问 shadow map

```javascript
console.log(directionalLight.shadow)
```
<div align="center"> {% asset_img shadow.png 400 %} </div>

默认情况下 shadow map大小是512x512的，我们可以将它增大。不过增大后当然会增加渲染压力，所以需要做好性能与视觉效果的平衡

```javascript
directionalLight.shadow.mapSize.width = 1024
directionalLight.shadow.mapSize.height = 1024
```
可以观察一下增大map分辨率后的效果：左图为512像素，右图为1024像素
<div align="center"> {% asset_img before.png 400 %} {% asset_img after.png 400 %}</div>

在Three.js中，光源对象是可以具有camera属性的，这是因为一些光源需要像摄像机一样具有位置和方向，例如投影灯（SpotLight）和逐像素光（PointLight）。这些光源需要发射光线，以便根据场景中的物体计算阴影。

与摄像机类似，光源的位置和方向是在场景中进行计算的。如果光源的位置或方向发生变化，则必须更新光源的camera属性。这样，当场景被渲染时，Three.js可以在正确的位置和方向发射光线，并生成正确的阴影效果。

因此，虽然光源不是一个摄像机对象，但为了光源能够在正确的位置和方向发射光线，它需要具有camera属性。这个属性可以使光源与摄像机类似，具有位置和方向等属性，从而在场景中正确地渲染阴影效果。

我们可以通过 camera 属性访问光源的 camera 来调整shadow map，为了辅助我们调试，我们可以用 CameraHelper

```javascript
const directionalLightCamerHelper = new THREE.CameraHelper(directionalLight.shadow.camera)
scene.add(directionalLightCamerHelper)
```

<div align="center"> {% asset_img helper.png 400 %}</div>

调整一下近端(far)和远端(near)。
同样可以通过 top、right、bottom、left调整各个面的距离

```javascript
directionalLight.shadow.camera.near = 1
directionalLight.shadow.camera.far = 6

directionalLight.shadow.camera.top = 2
directionalLight.shadow.camera.left = 2
directionalLight.shadow.camera.right = 2
directionalLight.shadow.camera.bottom = 2
```
我们可以用visible隐藏helper

```javascript
directionalLightCameraHelper.visible = false
```

调整阴影模糊通过改变 radius 的值来改变模糊半径

```javascript
directionalLight.shadow.radius = 10
```
这种模糊只是在阴影上面模糊，并没有根据物体与物体、光线之间的距离确定模糊效果

## 3.阴影贴图算法

- THREE.BasicShadowMap: 效率高，但是质量低。会出现锯齿状边缘
- THREE.PCFShadowMap：效率低，阴影质量和平滑度提高。（默认）
- THREE.PCFSoftShadowMap
- THREE.VSMShadowMap: 该算法使用两个深度图像素，一个用于存储平均深度，另一个用于存储深度方差。通过使用深度方差来计算阴影强度，可以提高阴影质量和平滑度。

应用阴影贴图属性通过设置renderer.shaodwMap.type属性改变。使用PCFSoftShadowMap算法时，radius会失效
```javascript
renderer.shaodwMap.type = THREE.PCFSoftShadowMap
```

## 4.SPOTLIGHT
讲解完以DirectionalLight为例的阴影。接下来以SPOTLIGHT为例解说阴影

先创建一个SPOTLIGHT

```javascript
const spotLight = new THREE.SpotLight(0xffffff, 0.4, 10, Math.PI * 0.3)

spotLight.castShadow = true

spotLight.position.set(0, 2, 2)

scene.add(spotLight)
scene.add(spotLight.target)
//添加helper
const spotLightCameraHelper = new THREE.CameraHelper(spotLight.shadow.camera)
scene.add(spotLightCameraHelper)

//降低其他灯光的亮度

const ambientLight = new THREE.AmbitenLight(0xffffff, 0.4)

const directionalLight = new THREE.directionalLight(0xffffff, 0.4)
```

<div align="center"> {% asset_img spotlight.png 400 %}</div>

可以看出几个灯光混合出来的阴影和现实世界的并不相符，但起码有阴影了（

SpotLight会使用透视相机生成阴影，所以我们可以改变相机的fov来影响阴影。

同样也可以调整相机的 near 和 far

隐藏helper
```javascript
spotLight.shadow.camera.fov = 30
```

## 5.PointLight

第三个支持阴影的就是 pointlight, 我们再添加一个PointLight

```javascript
const pointLight = new THREE.pointLight(0xffffff, 0.3)

pointLight.castShadow = true

spotLight.position.set(-1, 1, 0)

scene.add(spotLight)

```

能设置的东西都差不多，懒得写了

如果你给PointLight的camera添加了helper会看见一个朝向一个方向的透视相机，看起来很不合理。实际上PointLight的camera已经完成了六个方向的阴影生成，我们看见的camera朝向仅是最后一步

注意，点光源需要渲染生成周围所有的阴影。所以它的camera不能设置fov

## 6.Baking shadow
如果你有很多光源，需要生成一个好看的阴影，GPU是个挑战，所以我们需要烘焙阴影来减少GPU的负担。我们将来看一个使用烘焙阴影的例子。

首先关闭渲染器生成阴影贴图

```javascript
renderer.shadowMap.enabled = false
```

然后使用这张在blender生成的材质，要使用这张材质，我们要先加载

<div align="center"> {% asset_img bakedShadow.jpg 600 %}</div>

```javascript
//加载shadow
const textureLoader = new THREE.TextureLoader()
const bakedShadow = textureLoader.load('/textures/bakedShadow.jpg')
```

然后用 MeshBasicMaterial 代替平面上的 MeshStandarMaterial。并加载材质
```javascript
const plane = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(5, 5),
    new THREE.MeshBasicMaterial({
        map: bakedShadow
    })
)
```
然后你就能看见比较好看的阴影效果了，但注意因为是贴图所以移动球体后阴影并不会产生变化
<div align="center"> {% asset_img baked.png 400 %} {% asset_img posi.png 400 %}</div>

为了解决动态问题，我们可以在平面上创建一个小的平面，然后跟着球体运动

我们用这张阴影，来实现当球体移动的时候，阴影也随之移动。当球体上升时将阴影透明度降低
<div align="center"> {% asset_img simpleShadow.jpg 400 %}</div>
首先使用 MeshStandarMaterial 放回去，我们将simpleShadow应用在alpha贴图上

```javascript

///...load simpleShadow
const sphereShadow = new THREE.Mesh(
    new THREE.PlaneBufferGeometry(5, 5),
    new THREE.MeshStandarMaterial({
        color: 0xff0000,
        //transparent: true,
        alphaMap: simpleShadow
    })
)

sphereShadow.rotation.x = - Math.PI * 0.5

scene.add(sphereShadow,)
```
<div align="center"> {% asset_img sphere.png 400 %}</div>

然后就可以看到这个鬼样子，并且会产生plane与sphere层叠打架的问题,所以我们需要移动一下sphereShadow

```javascript
sphereShadow.position.y = plane.position.y + 0.01
```

然后将transparent的注释取消掉，并将color变成黑色，就可以得到一个比较好看的阴影了

<div align="center"> {% asset_img perfect.png 400 %}</div>

然后我们让球体动起来,转着圈跳动。并让阴影跟随

```javascript
const clock = new THREE.Clock() 
const tick = () => {
    const elapsedTime = clock.getElapsedTime()
    
    //update sphere
    sphere.position.x = Math.cos(elapsedTime) * 1.5
    sphere.position.z = Math.sin(elapsedTime) * 1.5
    sphere.position.y = Math.abs(Math.sin(elapsedTime * 3))
    
    //update shadow
    sphereShadow.position.x = sphere.position.x
    sphereShadow.position.z = shpere.position.z
    sphereShadow.material.opacity = (1 - Math.abs(sphere.position.y)) * 0.3
}
```

<div align="center"> {% asset_img follow2.png 400 %} {% asset_img fllow.png 400 %}</div>