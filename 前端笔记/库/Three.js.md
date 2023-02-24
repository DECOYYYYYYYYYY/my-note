# Three.js

## 基本使用

### 场景创建与渲染

```javascript
// 创建场景
const scene = new THREE.Scene(); 
// 创建相机
const camera = new THREE.PerspectiveCamera(75, canvas元素宽度 / canvas元素高度, 0.1, 1000);
scene.add(camera);
// 创建渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
// 渲染函数
function animate() {
	requestAnimationFrame(animate); // 自动循环调用animate方法（每次屏幕刷新时）
	renderer.render(scene, camera); // 渲染一帧
}
animate();
```

### 内存释放

```javascript
try {
    scene.clear();
    renderer.dispose();
    renderer.forceContextLoss();
    renderer.content = null;
    cancelAnimationFrame(animationID);
    let gl = renderer.domElement.getContext("webgl");
    gl && gl.getExtension("WEBGL_lose_context").loseContext();
    console.log(this.renderer.info); //查看memery字段确认内存是否释放完毕
}catch (e) {
    console.log(e)
}
```

### 窗口Resize处理

```javascript
window.addEventListener("resize", () => {
    const width = canvas元素宽度;
    const height = canvas元素高度;

    // 更新相机
    camera.aspect = width / height;
    camera.updateProjectionMatrix();

    // 更新渲染器
    renderer.setSize(width, height);
    // this.renderer.setPixelRatio(window.devicePixelRatio);
    
   	// 若存在后处理，更新后处理相关对象
    postProcessing.composer.setSize(width, height);
    postProcessing.outlinePass.setSize(width, height);
	...
});
```

## 渲染器

创建：`new WebGLRenderer(参数对象)`

- `antialias: false` 抗锯齿
- `canvas` 传入canvas DOM元素，用于绘制输出。若不传会创建新的canvas

实例属性：

- `domElement` 
- `physicallyCorrectLights: false` 设置是否开启物理上的正确光照模式
- `outputEncoding: THREE.LinearEncoding` 设置纹理编码

实例方法：

- `render(scene: Object3D,camera: Camera): und`
  - 使用相机进行渲染
- `setClearColor(color: Color, alpha: flt): und` 
  - 指定无颜色像素的默认颜色与不透明度
- `setPixelRatio(value: num): und` 
  - 设置设备像素比，通常传入`window.devicePixelRatio`
- `setSize(width: int, height: int[, updateStyle: bool]): und` 
  -  设置输出canvas的大小，设置updateStyle为false以阻止对canvas的样式做任何改变

## 视角

### 透视相机

创建：`new THREE.PerspectiveCamera(fov: num, aspect: num, near: num, far : num )`

- fov：摄像机视锥体垂直视野角度。`75`
- aspect：摄像机视锥体长宽比。`window.innerWidth / window.innerHeight`
- near/far：摄像机视锥体近截面/远截面 `0.1 100`

设置位置：`camera.position.set(x: num, y: num, z: num): und`

### 控制器

```javascript
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
const controls = new OrbitControls( camera, renderer.domElement );
```

实例属性：

- `enabled: bool` 是否生效
- `enableDamping: bool` 是否启用阻尼（惯性）
  - 若启用阻尼，需要在渲染函数中调用`controls.update()`
- `dampingFactor: flt` 阻尼系数
- `enablePan: bool = true` 是否允许摄像机平移
- `enableRotate: bool = true` 是否允许摄像机旋转

## 物体

### Object3D

三维物体，是Three.js中大部分对象的基类

创建：`new THREE.Object3D();`

实例属性：

- `name: String`、`visible: bool`、`id: int`、`animations: arr[AnimationClip]`
- `parent: Object3D`、`children: arr[Object3D] `
- `position: Vector3`、`scale: Vector3`、`rotation: Euler`、`quaternion: Quaternion`
  - 上述四个属性均为局部变换属性，其坐标基于父元素而非`scene`。
  - `rotation`基于欧拉角表示旋转（单位为弧度），`quaternion`基于四元数表示旋转。两者都表示旋转相关信息，都会被转换为旋转变换矩阵。
- `userData: Object` 存储自定义数据的对象，不应当包含对函数的引用

实例方法：

- `add(object: Object3D, ...): this` 添加任意数量对象为该对象的子级。传入对象的父级将在这里被移除，因为一个对象仅能有一个父级。
- `copy(object: Object3D[, recursive: Boolean = true]):this`
  - 复制传入对象到该对象中（事件监听与用户定义的回调函数不会被复制）
  - 若recursive为true，则同时复制后代
- `clone([recursive: Boolean = true]): Object3D`
  - 返回该对象的克隆
- `attach(object: Object3D): this`  添加对象该对象的子级，同时保持被添加对象的世界变换。
- `getObjectById(id: int): Object3D` 搜索该对象及其子级，查找并返回id匹配的对象
- `getObjectByName(name: String): Object3D` 搜索该对象及其子级，查找并返回name匹配的对象
- `getObjectByProperty(name: String, value: Any): Object3D` 搜索该对象及其子级，查找并返回属性与值匹配的对象
- `getWorldPosition(target: Vector3): Vector3` 复制该物体在世界空间的位置矢量到target中
- `getWorldQuaternion(target: Quaternion): Quaternion` 、`getWorldScale(target: Vector3): Vector3` 同上
  - 获取世界坐标前，应先更新该物体的全局变换（`updataMatrixWorld`）
- `updateMatrixWorld(force: Boolean ): und` 更新该物体及其后代的全局变换

### 网格模型

```javascript
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({color: 0x00ff00});
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);
```

构建网格模型`Mesh`需要立方体几何体`geometry`和材质`material`。

创建：`new THREE.Mesh(geometry: BufferGeometry, material: Material)`

### 几何体

立方体几何体`geometry`本质上就是一系列的顶点构成

- `new THREE.BoxGeometry(width: flt, height: flt, depth: flt)` 立方体
- `new THREE.PlaneGeometry(width: flt, height: flt)` 平面

### 材质

- `new THREE.MeshBasicMaterial(parameters: obj)`  
  - 简单着色材质，不受光照影响
- `new THREE.MeshStandardMaterial(parameters: obj)`  
  - 基于物理渲染（PBR）的标准材质

## 纹理

纹理贴图可应用于一个表面，或作为反射/折射贴图。

**UVMap**：`geometry.attributes`中有`position`与`uv`属性，保存了xyz与uv的映射关系

**加载图片获得Texture纹理对象：**见[贴图加载器](###贴图加载器)

**设置物体贴图：**

- 颜色贴图：`obj.material.map: Texture`

- 发光（放射）贴图：
  - `obj.material.emissive: Color` 放射光颜色，默认为黑色，若设置了贴图，需设置为黑色外其他颜色
  - `obj.material.emissiveMap: Texture` 设置放射贴图
  
- 阴影（光照）贴图：
  - 光照贴图需要第二组UV：`obj.geometry.attributes.uv2 = obj.geometry.attributes.uv`
  - `obj.material.lightMap: Texture`  设置光照贴图
  
- 环境贴图：

  用于设置环境的光照信息，起到照亮物体，绘制物体表面反射图像的作用

  ```javascript
  import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js';
  new RGBELoader().load('quarry.hdr', (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping;
      scene.background = texture;
      scene.environment = texture;
  } );
  ```

- 法线贴图：
  - `obj.material.normalMap = Texture` 设置法线贴图
  - `obj.material.normalScale = new THREE.Vector2(3, 3)` 设置凹凸程度

## 光影

### 灯光

环境光：`new THREE.AmbientLight(color: int = 0xffffff[, intensity: flt = 1])`

点光源：`new THREE.PointLight(同上);`

```javascript
// 添加环境光
const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
scene.add(ambientLight);
// 添加点光源
const pointLight = new THREE.PointLight(0xffffff, 0.8);
camera.add(pointLight); // 点光源随摄像机移动
```

### 投影

```javascript
renderer.shadowMap.enabled = true; // 渲染器启用阴影
light.castShadow = true; // 光源产生阴影
cube.castShadow = true; // 物体产生阴影
plane.receiveShadow = true; // 其他物体接收投影
```

### 反射

#### 反射

Three.js中无光线追踪，为了实时反射周围动态物体，可用以下假反射：

**方法一：环境贴图反射（CubeCamera）**

- CubeCamera为一组六个透视相机，从其所处点向外拍摄全景照片，将拍摄所得制作为贴图，将其贴图应用于材质的环境贴图，可以实现实时反射
- 优点：应用于外部模型时，开启反射较方便
- 局限：该方法损失了外部物体与CubeCamera之间的距离信息，调整视角远近后反射图形不会发生变化。面越平，效果越差。

```javascript
const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(128); // 贴图尺寸（像素）
const cubeCamera = new THREE.CubeCamera(1, 100000, cubeRenderTarget); // 近截点，远截点
this.scene.add(cubeCamera);
obj.material.envMap = cubeRenderTarget.texture;

// 渲染函数内
obj.visible = false;
cubeCamera.position.copy(obj.position);
cubeCamera.update(renderer, scene);
obj.visible = true;
```

**方法二：平面反射（Reflector）**

- 一个平面，纯镜面
- 局限：只能通过代码创建镜面，无法用于使物体表面开启反射。

```javascript
import { Reflector } from 'three/examples/jsm/objects/Reflector.js';
const mirror = new Reflector(geometry实例, {
    clipBias: 0.02,
    textureWidth: window.innerWidth * window.devicePixelRatio,
    textureHeight: window.innerHeight * window.devicePixelRatio,
    color: new THREE.Color(0xffffff)
});
scene.add(mirror)
```

**方法三：屏幕空间反射（SSR）**

- 任何面都能实时反射，无畸变
- 性能较差
- 官方示例：https://github.com/mrdoob/three.js/blob/master/examples/webgl_postprocessing_ssr.html

## 动画

```javascript
const planeScaleTrack = new THREE.KeyframeTrack('plane.scale', [0, 0.8, 1.6], [1, 1, 1, 1.2, 1.2, 1.2, 1, 1, 1]);
const planeClip = new THREE.AnimationClip('planeClip', 2, [planeScaleTrack]);
const mixer = new THREE.AnimationMixer(scene);
const animationAction = mixer.clipAction(planeClip);
animationAction.play();

// 渲染函数中调用
mixer.update(clock.getDelta());
```

### KeyframeTrack

关键帧轨道是关键帧的定时序列，定义了对象的指定属性随时间如何变化

创建：`new THREE.KeyframeTrack(name: str, times: arr, values: arr)`

- name：属性标识符，格式为 `对象的name + '.' + 属性名`。例：`'cube.position'`
  - 若要修改position、rotation的单个轴：`cube.position[x]`
- times：时间数组，由各关键帧所在秒数组成。例`[0, 1]`
- values：值数组，由各关键帧对应属性值组成。例`[0, 0, 0, 2, 5, 0]`（三个一组，分别为position的xyz值）

### AnimationClip

动画剪辑是一个可重用的关键帧轨道集，它代表动画。

创建：`new THREE.AnimationClip(name: str, duration: num, tracks: arr)`

- name：此剪辑的名称
- duration：动画持续秒数
- tracks：关键帧轨道数组

### AnimationMixer

动画混合器是用于场景中特定对象的动画的播放器。

创建：`new THREE.AnimationMixer(rootObject: Object3D)`

- rootObject：混合器可以操作传入对象及其后代

实例方法：

- `update(deltaTimeInSeconds: num): this` 
  - 推进混合器时间并更新动画，一般传入`clock.getDelta()`
- `existingAction(clip: AnimationClip[, root: Object3D]): AnimationAction`
  - 返回传入剪辑的已有AnimationAction, 根对象参数默认值为混合器的默认根对象。
  - 第一个参数可以是剪辑对象或者动画剪辑的名称。 
- `uncacheClip(clip: AnimationClip): und`
  - 释放剪辑的所有内存资源
- `uncacheRoot(root: Object3D): und`
  - 释放根对象的所有内存资源
- `uncacheAction(clip: AnimationClip[, root: Object3D]): und`
  - 释放剪辑所在动作的所有内存资源

### AnimationAction

AnimationAction用于控制动画剪辑

创建：`mixer.clipAction(clip: AnimationClip, optionalRoot: Object3D)`

实例属性：

- `enabled: bool` 动画是否生效
- `loop: num` 循环模式（循环结束后自动设置`enabled`属性为false），其值为以下值之一：
  - `THREE.LoopOnce` 只执行一次
  - `THREE.LoopRepeat` 重复`repetitions值`次, 每次循环结束时回到起始点
  - `THREE.LoopPingPong` 重复`repetitions值`次, 在起始点与结束点之间来回循环
- `repetitions: num = Infinity` 循环次数
- `timeScale: num = 1` 时间比例因子。为负数时动画反向执行。
- `time: num` 动作开始的时间点（单位为秒），从0开始。
- `weight: num` 动作的影响程度（取值为[0, 1]，0为无影响，1为完全影响）
  - 当多个动作同时对一个属性进行操作时，根据weight属性来确定属性的实际变化
- `paused: bool` 当值为true时，会将timeScale设置为0以暂停动作

实例方法：

- `play(): this` 激活动画加载器

## 加载器

加载器实例可以使用`setPath('url')`方法设置基础路径，使用`load()`加载时会将基础路径与load接收路径进行拼接

注：加载器的url不会被Vue自动转换，因此在打包后仍和打包前一致，可以使用`process.env.NODE_ENV`判断是否处于生产环境，并生成对应的url

### 格式介绍

- GLTF（推荐）：一个glTF组件可传输一个或多个场景， 包括网格、材质、贴图、蒙皮、骨架、变形目标、动画、灯光以及摄像机。
- OBJ：是一种3D模型文件格式。不包含动画、材质、贴图等信息。
- MTL：OBJ的配套文件格式。用于描述一个或多个OBJ 文件中物体表面材质属性
- STL：用三角形表示实体的文件格式。只有点，三角形，体等几何信息。

### GLTF加载器

```javascript
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
const loader = new GLTFLoader();
renderer.outputEncoding = THREE.sRGBEncoding; // gltf格式默认使用sRGB色彩空间
// loader.setDRACOLoader(dracoLoader); // 若gltf使用了Draco进行压缩，需设置dracoLoader
loader.load('url', gltf => {}, xhr => {}, error => {}); 
```

可用于加载`.glb`、`.gltf`文件

创建：`new GLTFLoader()`

方法：

- `load(url: str, onLoad: gltf =>{}, onProgress: xhr =>{}, onError: error =>{}): und` 
  - 加载文件。三个函数分别在加载完成后调用、加载过程中调用、报错后调用

### Draco加载器

使用Draco库压缩的几何图形加载器。

```javascript
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('three/examples/js/libs/draco/');
```

若报错，将draco文件夹复制到static目录中，更改decoderPath为`static/draco/`

### OBJ与MTL加载器

```javascript
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader';
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader';
new MTLLoader().load('url', materials => {
    materials.preload();
    new OBJLoader().setMaterials(materials).load('url', obj => {
        scene.add(obj);
    }, xhr => {}, error => {}); 
});
```

- mtl文件可能需要修改贴图路径
- 环境贴图无法应用于该方法加载的模型，手动设置物体的material.envmap使其生效

### STL加载器

```javascript
import { STLLoader } from './jsm/loaders/STLLoader.js';
new STLLoader().load('url', geometry => {}, xhr => {}, error => {});
```

### 贴图加载器

```javascript
// 方法一：同步加载
const texture = new THREE.TextureLoader().load("url");
// 方法二：异步加载
new THREE.TextureLoader().load('url', texture => {}, undefined, err => {});
```

- `load`方法返回一个`Texture`实例
- `TextureLoader`不支持`onProgress`回调

## GUI

### 性能监控

```javascript
import Stats from 'three/examples/jsm/libs/stats.module.js';
const stats = new Stats();
document.body.appendChild(stats.dom);

// 渲染函数内
stats.update();
```

### GUI

文档：https://lil-gui.georgealways.com/

```javascript
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';
const gui = new GUI();
gui.open();
```

- 添加菜单项：`const folderLocal = gui.addFolder("菜单名");`

- 添加单选框：`folderLocal.add(对象, "属性名");` 对象需要有指定属性

- 添加拉动条：`folderLocal.add(对象, "属性名", min: num, max: num, step: num);` 要求同上

- 添加按钮：`folderLocal.add(对象, "属性名");` 对象中的指定属性值应该是一个方法

- 添加下拉表：

    ```javascript
    const dropdown= {state: 'Walking'}; // 默认值
    const states = ['Idle', 'Walking', 'Running', 'Dance', 'Death', 'Sitting', 'Standing'];
    const clipCtrl = folderLocal.add(dropdown, 'state').options(states);
    clipCtrl.onChange((newVal) => {}); // 设置点击事件
    ```

设置属性名：`xxx.add().name('abcd');`

## 辅助

### 模型边框

```javascript
const edges = new THREE.EdgesGeometry(geometry: BufferGeometry);
const line = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({color: 0xffffff}));
scene.add(line);
```

### 光线投射

实现鼠标选取物体：

```javascript
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
function onPointerMove(event) {
	// 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1, +1)
	pointer.x = (event.offsetX / canvas元素宽度) * 2 - 1;
	pointer.y = -(event.offsetY / canvas元素高度) * 2 + 1;
}
window.addEventListener('pointermove', onPointerMove);

// 渲染函数中
raycaster.setFromCamera(pointer, camera); // 通过摄像机和鼠标位置更新射线
const intersects = raycaster.intersectObjects(scene.children); // 计算物体和射线的焦点
for (let i = 0; i < intersects.length; i++) {
    intersects[i].object.material.color.set( 0xff0000 ); // 设置选中物体的样式
}
```

`raycaster.intersectObject(object: Object3D, recursive: bool[, optionalTarget: arr]): arr`

- `object` 检查与射线相交的物体。
- `recursive` 若为true，则同时也会检查所有的后代。否则将只会检查对象本身。默认值为true。
- `optionalTarget` 设置结果的目标数组。如果不设置这个值，则一个新的Array会被实例化；如果设置了这个值，则在每次调用之前必须清空这个数组（例如：array.length = 0;）。

`raycaster.intersectObjects(objects: arr, recursive: bool, optionalTarget: arr): arr`

## 后处理

```javascript
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
const composer = new EffectComposer(renderer);
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);
... // 添加其他pass

// 渲染函数中：不再调用renderer.render(...)，改为composer.render()
composer.render(); // 渲染一帧
```

### EffectComposer

效果合成器，用于渲染。

```javascript
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
const composer = new EffectComposer(renderer);
```

方法：

- `addPass(pass)`  添加效果通道
- `render()`  渲染一帧

### RenderPass

渲染通道，是最基础的`pass`，必须是第一个添加给`effectComposer`的`pass`。

```javascript
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
const renderPass = new RenderPass(scene, camera)
```

### OutlinePass

物体轮廓线渲染通道

```javascript
import { OutlinePass } from "three/examples/jsm/postprocessing/OutlinePass.js";
const outlinePass = new OutlinePass(
    new THREE.Vector2(canvas元素宽度, canvas元素高度),
    scene,
    camera
);
```

实例属性与方法：

- `enabled:bool`
- `selectedObjects: arr[Object3D]`  要显示轮廓线的物体
- `edgeThickness: num`  轮廓线光晕粗细
- `edgeStrength: num`  轮廓线粗细（单位为像素）
- `edgeGlow: num`  轮廓线发光强度
- `pulsePeriod: num = 0`  轮廓线闪烁。若值为0，不闪烁，否则闪烁周期为所设置的值（单位为秒）
- `usePatternTexture: bool`  被选中的物体上显示网格状纹理
- `visibleEdgeColor.set(color: String)`  轮廓线颜色（使用`"#777"`的颜色字符串）
- `hiddenEdgeColor.set(color: String)`  轮廓线被其他物体挡住时的颜色

### ShaderPass

着色器渲染通道，用于实现抗锯齿等效果。

FXAA抗锯齿：

```javascript
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass.js";
import { FXAAShader } from "three/examples/jsm/shaders/FXAAShader.js";
const effectFXAA = new ShaderPass(FXAAShader);
effectFXAA.uniforms["resolution"].value.set(1 / canvas元素宽度, 1 / canvas元素高度);
composer.addPass(effectFXAA);
```

## 进阶

### 更新机制

在大多数场景下，一些数据不需要每次渲染都要更新，为了提高性能，对于不经常更新的对象，默认不更新，如果有相关的更新发生，需要手动更新

- 纹理对象更新：`texture.needsUpdate = true;`
  - `needsUpdate`属性：渲染器在执行`render`方法时，若对象的`needsUpdate`属性为`true`，则更新对象，然后将该属性置回`false`

- 相机对象更新：`camera.updateProjectionMatrix();`
- 材质对象更新：`material.needsUpdate = true;`

## 常见BUG

### 深度冲突

描述：渲染器无法分清重叠面的先后顺序，导致闪烁。

解决方法（https://zhuanlan.zhihu.com/p/151649142）：

- 方法一：使两个面错开一小段距离

- 方法二：给其中一个面（物体）设置多边形偏移，在深度冲突时，它会被拉近/推离摄像机一段距离

```javascript
obj.material.polygonOffset = true;
// 以下两个参数设为负表示拉近
obj.material.polygonOffsetFactor = -1;
obj.material.polygonOffsetUnits = -1;
```

### 阴影条纹

描述：物体同时打开接受阴影与产生阴影开关时，其表面可能出现条纹

解决方法：`obj.material.shadowSide = THREE.BackSide;`

### 与前端框架搭配时性能差

Three.js的对象更新频率高，若对其进行数据劫持，会导致性能下降。