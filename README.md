# Magic Christmas Tree 🎄

基于 Three.js 与 MediaPipe 的交互式 3D 圣诞树：手势聚合/散开粒子星云、程序化飘雪、上传照片环绕成背景。所有前端依赖与视觉模型都已本地化，不访问 jsDelivr 或 Google 存储，国内网络环境可直接运行。

## 特性

- **粒子圣诞树**：金球、银块、红蓝宝石与树顶星，配合泛光（Bloom）与色调映射。
- **本地手势识别**：MediaPipe HandLandmarker 的 wasm 与 `.task` 模型放在 `vendor/`，点击「开启手势」时才按需加载。
- **双控制方式**：手势识别可用时支持握拳、张手、挥手、捏合；摄像头不可用或模型加载失败时自动降级为鼠标/触控拖动。
- **飘雪**：雪花贴图由 Canvas 程序化生成，无外部图片资源；每片雪花独立速度与横向摆动。
- **照片环绕背景**：上传的本地照片分布在一个缓慢旋转的照片环上，环绕整个场景；点击或捏合可把照片放大到镜头前，并自动切换到高清纹理。
- **照片不被泛光糊掉**：照片单独放在一个图层，Bloom 通过遮罩把照片排除在外，白底或高亮照片也能保持清晰。
- **沉浸模式**：右下角按钮隐藏界面，只留画面。

## 操作

| 操作 | 效果 |
| --- | --- |
| 握拳 | 粒子聚合成圣诞树 |
| 张手 | 粒子散开为星云 |
| 挥手（散开时） | 带惯性旋转视图 |
| 捏合 / 点击照片 | 聚焦照片，点击空白处返回 |
| 鼠标 / 触控拖动 | 任意状态下旋转视图 |
| 上传纪念照片 | 照片进入环绕背景的光环，自动均匀分布 |

## 运行

页面使用 ES Module、WASM 和本地模型文件，**必须通过 HTTP 服务访问**，直接双击 `index.html` 无法加载。

```bash
# 任选其一
npx serve .
python -m http.server 4173
```

然后打开 `http://localhost:4173/`。部署到 GitHub Pages、Nginx 等静态托管同样不需要额外配置。

摄像头功能需要 `localhost` 或 HTTPS 环境，并允许浏览器请求的摄像头权限。

## 目录结构

```
index.html                          页面、样式与全部运行逻辑
vendor/three/build/three.module.js  three r160
vendor/three/addons/                用到的后处理与环境模块（共 9 个文件）
vendor/mediapipe/vision_bundle.js   @mediapipe/tasks-vision 0.10.0（ES Module）
vendor/mediapipe/wasm/              视觉任务 WASM（SIMD 与非 SIMD 两套）
vendor/models/hand_landmarker.task  手部关键点模型（约 7.5 MB）
```

`vendor/` 合计约 27 MB。手势相关文件只在点击「开启手势」后请求，普通浏览和拖拽交互不会加载它们。

## 配置

`index.html` 顶部的 `CONFIG` 可调整：

| 字段 | 默认值 | 说明 |
| --- | --- | --- |
| `goldCount` / `silverCount` | 500 / 500 | 金球、银块数量 |
| `gemCount` / `emeraldCount` | 300 / 300 | 红蓝宝石与绿宝石数量 |
| `dustCount` | 1000 | 树体光尘数量 |
| `treeHeight` / `maxRadius` | 75 / 30 | 树高与最大半径 |
| `camDistance` | 110 | 相机距场景中心距离 |
| `ringRadius` | 150 | 照片环半径（相机位于环内） |
| `ringPhotoSize` | 42 | 环上照片的最大边长 |
| `ringSpeed` | 0.08 | 照片环角速度（弧度/秒） |
| `maxPhotos` | 24 | 环绕背景照片上限 |
| `snowCount` | 900 | 雪花数量 |
| `snowArea` / `snowSpeed` | 340 / 8 | 飘雪空间范围与下落速度 |
| `snowSize` | 3.0 | 雪花尺寸 |

## 照片清晰度

- 环上照片使用 640px 缩略纹理，保证环绕旋转和多张照片时的显存占用可控。
- 点击/捏合聚焦时，按需重新读取原始文件，生成最长边不超过 2048px 的高清纹理；同一时间只保留一张高清纹理，切换焦点会释放上一张。
- 纹理各向异性过滤开到 8 倍，照片倾斜或斜视时不会发糊。
- 渲染像素比上限为 2，在高分屏上照片边缘更清晰。
- 照片环的雾效已关闭，远处照片不会被场景雾压灰。

## 本地化与降级说明

- 视觉模型的 wasm、模型权重、three.js 全部从 `vendor/` 读取，`index.html` 中不再出现任何 CDN 或 Google 存储地址。
- 点击「开启手势」后才动态 `import` MediaPipe 并创建识别器，首屏不受其体积影响。
- 识别器优先使用 GPU delegate，创建失败时自动回退到 CPU。
- 摄像头被拒绝、无摄像头或模型异常时，状态栏提示原因，鼠标/触控拖动与照片环绕功能不受影响。

## 常见问题

- **页面空白或模块加载失败**：确认通过 `http://` 访问，而不是 `file://`。
- **点击开启手势后等待数秒**：需要加载约 18 MB 的 wasm 与 7.5 MB 模型，首次加载后由浏览器缓存。
- **状态提示「未获得摄像头权限」**：允许摄像头权限后点击「重试摄像头」，或直接使用拖动旋转。
- **性能偏低**：可减少 `snowCount`、`goldCount` 等数量，或降低 `renderer.setPixelRatio` 的上限（默认限制 2）。

## 来源与许可

- three.js：MIT，见 `vendor/three/LICENSE`。
- MediaPipe Tasks Vision：Apache License 2.0，说明见 `vendor/mediapipe/LICENSE-README.md`。
- 本项目在 [etocs/ChristmasTree](https://github.com/etocs/ChristmasTree) 基础上做了依赖本地化、视觉模型按需加载与降级、雪花和照片环绕背景等改造。

Merry Christmas!
