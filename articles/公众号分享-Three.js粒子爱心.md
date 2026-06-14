# <span style="color: #C445B8;">抖音上这么火的 3D 爱心浪漫动画，原来是这么做的</span>

## <span style="color: #7B5CD6;">一、从一个热门粒子爱心说起</span>

最近在短视频平台上，经常能看到这样的动画。

粒子从地面升起。

形成粉紫色龙卷风。

随后聚合成一颗旋转的 3D 爱心。

爱心转动时，满屏光子也被带动，形成浪漫的粒子环流。

<div style="text-align: center; margin: 16px 0 24px;">
  <video controls preload="metadata" src="屏幕录制 2026-06-14 233446.mp4" title="3D 粒子爱心效果演示" style="display: block; width: 100%; max-width: 720px; height: auto; margin: 0 auto; border-radius: 8px;"></video>
</div>

现在把效果描述给 AI。

它很快就能搭建 Three.js 场景、创建粒子系统，甚至编写 Shader。

我也和一些做过类似作品的作者交流过。

他们并不会传统编程。

但通过 Vibe Coding，同样做出了不错的效果。

<div style="margin: 18px 0; padding: 14px 16px; border-left: 4px solid #C445B8; background: #FAF2F9; color: #5A3155; line-height: 1.8;">
AI 确实降低了创意编程的门槛。<br />
但这并不等于“一句话生成成品”。
</div>
并不像营销号说的那样什么什么模型就无敌了，什么什么行业就要完蛋了。。。。 

真实过程，往往是反复描述、生成、观察和纠偏。

这个过程不仅耗时，也会消耗不少 Token。

项目代码越长，每轮读取上下文和重写代码的成本就越高。

如果只会说“效果不对，再优化一下”，AI 很可能连续几轮都没有改到关键位置。

因此，了解基本原理仍然很有价值。

它能帮助我们把“感觉不对”，转换成更准确的技术描述。

也能让每次沟通都更接近目标。

<p style="color: #C445B8; font-weight: 700; margin: 20px 0 8px;">下面，我们从形状、运动、旋转和背景力场四个方面，拆解这个 3D 粒子爱心。</p>

## <span style="color: #7B5CD6;">二、实现原理</span>

### <span style="color: #C445B8;">1. 画面由三层粒子组成</span>

整个场景由三套 `THREE.Points` 构成：

- **主体层**：7600 颗粒子组成爱心，1100 颗粒子保留在地面。
- **星尘层**：3600 颗小粒子放在远景，增加空间深度。
- **光子层**：2600 颗不同尺寸的柔光粒子覆盖全屏，并响应爱心旋转。

最底层再使用一个 Shader 渐变平面，提供暗紫色雾光。将主体、星尘和光子分开，可以独立控制密度、亮度和运动速度，画面也更有层次。

### <span style="color: #C445B8;">2. 用隐式方程生成真正的 3D 爱心</span>

常见做法，是在二维平面生成心形，再随机增加 Z 坐标。

它正面看似正常。

转到侧面，却会像两张分离的薄片。

这里使用隐式三维心形方程：

```js
function heartVolumeField(x, y, z) {
  const base = x * x + 2.25 * z * z + y * y - 1;
  return base * base * base
    - x * x * y * y * y
    - 0.1125 * z * z * y * y * y;
}
```

函数值小于或等于 0，表示点位于爱心内部。通过随机生成空间点并保留内部点，就能得到连续的三维爱心体积。

为了兼顾清晰轮廓和内部厚度，目标点分为三组：

- 40% 位于主要轮廓和侧壁。
- 30% 位于完整三维表面。
- 30% 位于内部体积。

表面点通过随机方向和二分查找获得，内部点通过拒绝采样获得。最后分别调节 X、Y、Z 缩放，控制爱心的宽度、高度和厚度。

### <span style="color: #C445B8;">3. 每颗粒子拥有独立的运动数据</span>

主体粒子使用 `BufferGeometry` 保存起点、目标点、延迟、速度、尺寸和路径参数：

```js
geometry.setAttribute("aStart", startAttribute);
geometry.setAttribute("aTarget", targetAttribute);
geometry.setAttribute("aDelay", delayAttribute);
geometry.setAttribute("aAngle", angleAttribute);
geometry.setAttribute("aSpeed", speedAttribute);
geometry.setAttribute("aPath", pathAttribute);
```

这些数据初始化后，直接交给 GPU。

动画过程中，JavaScript 只更新时间和旋转角等少量 uniform。

不需要每帧遍历数千颗粒子。

### <span style="color: #C445B8;">4. 粒子运动分为四个阶段</span>

每颗粒子都有独立延迟，因此不会同时起飞。顶点着色器根据局部时间切换四个阶段。

<span style="color: #7B5CD6; font-weight: 700;">地面等待</span>

粒子停留在地面，只做轻微漂移。靠近中心的粒子延迟更短，会优先升起。

<span style="color: #7B5CD6; font-weight: 700;">螺旋上升</span>

粒子一边提高 Y 坐标，一边围绕垂直轴旋转。旋转半径从底部逐渐收窄，接近爱心时再略微散开：

```glsl
float bottomRadius = mix(0.92, 0.27, smoothstep(0.0, 0.48, progress));
float topSpread = 0.38 * smoothstep(0.70, 1.0, progress);
float radius = (bottomRadius + topSpread) * aPath.x;
float theta = aAngle + progress * TAU * (5.0 + aSpeed * 2.15);
```

同时改变高度、角度和半径，才能形成龙卷风，而不是普通喷泉。

<span style="color: #7B5CD6; font-weight: 700;">弧线吸附</span>

粒子离开风柱后，通过三次贝塞尔曲线飞向目标点：

```glsl
transformed = bezier(p0, p1, p2, rotatedTarget, arcProgress);
```

两个控制点负责侧向甩出和接近目标时的弧度，使运动更像被爱心吸引，而不是沿直线移动。

<span style="color: #7B5CD6; font-weight: 700;">固定成形</span>

粒子到达目标后立即固定，成为爱心的一部分，只保留轻微闪烁。

### <span style="color: #C445B8;">5. 吸附目标必须跟随爱心旋转</span>

爱心在聚合过程中开始旋转，并在约 9 秒内完成三圈：

```js
const rotationProgress = THREE.MathUtils.smoothstep(elapsed, 4.2, 13.2);
const heartRotation = rotationProgress * Math.PI * 2 * 3;
```

关键在于，旋转不能只作用于已经固定的粒子。

正在飞行的粒子，也必须追踪旋转后的目标位置：

```glsl
vec3 rotatedTarget = rotateHeart(aTarget);
```

贝塞尔曲线的终点和固定位置都使用 `rotatedTarget`，这样粒子轨迹与爱心旋转始终保持连续。

鼠标和触摸拖动也会叠加到旋转角上。松手后速度逐渐衰减，形成轻微惯性。

### <span style="color: #C445B8;">6. 背景光子通过距离力场响应旋转</span>

背景光子持续漂浮，并在屏幕边界循环。爱心旋转时，Shader 会根据光子到爱心中心的距离计算影响强度：

```glsl
float nearInfluence = 1.0 - smoothstep(4.6, 13.2, distanceToHeart);
float influence = nearInfluence * uAttraction;
```

距离越近，光子受到的旋转和收拢效果越强。

远处光子，只产生轻微偏转。

这样，背景就不再是独立装饰。

它与主体共享同一个视觉力场。

粒子本身通过 `gl_PointCoord` 绘制圆形核心和窄柔边。主体使用 `NormalBlending` 保持细节，背景柔光使用低透明度 `AdditiveBlending`，避免爱心叠加成白色实体。

所有运动都在 Shader 中计算，每帧只更新少量 uniform；同时将 `devicePixelRatio` 限制在 2 以内，从而兼顾粒子密度与性能。

## <span style="color: #7B5CD6;">三、提示词分享</span>

当然我感觉大家可能更关心怎么快速复现，下面就分享下提示词。
实际使用时，可以先生成完整版本，再根据画面继续细化参数。

```text
请使用 Three.js 实现一个粒子从地面聚合成三维爱心的浪漫动画，输出完整可运行的单文件 HTML、CSS 和 JavaScript。

画面要求：
1. 使用 16:9 深黑色背景和固定正面透视镜头。
2. 爱心位于画面上方中央，地面粒子铺满下方约 25% 区域。
3. 爱心与地面保持明显距离，中间只有一股粒子旋风连接。

爱心要求：
1. 爱心完全由数千颗独立粒子组成，禁止使用实体心形 Mesh 或填充平面。
2. 使用隐式三维心形方程生成连续封闭的 3D 心形体，不能使用两张二维薄片拼接。
3. 70% 的目标点位于轮廓和表面，30% 位于内部。
4. 轮廓稍密、内部稍疏，粒子之间保留黑色缝隙。
5. 颜色以亮粉、紫红和紫色为主，混合少量白色粒子。
6. 粒子主要为屏幕 2-4 像素，中心清晰，只有轻微柔光。

运动要求：
1. 粒子初始散落在地面，每颗粒子具有独立延迟。
2. 中心粒子先升起，围绕垂直轴旋转，形成底部较宽、中段较细、顶部略散开的龙卷风。
3. 粒子离开风柱后，通过三次贝塞尔曲线飞向各自目标点，禁止线性飞向中心。
4. 到达目标后立即固定。爱心先形成轮廓，再逐步填充内部。
5. 聚合过程持续约 8-12 秒，最后保留少量地面粒子。

旋转与交互：
1. 爱心在吸附阶段开始绕 Y 轴旋转，并自动旋转三圈。
2. 飞行中的粒子必须实时追踪旋转后的目标点。
3. 支持鼠标和触摸拖动，水平控制旋转，垂直控制倾斜，松开后带轻微惯性。

背景要求：
1. 加入暗紫粉渐变雾光、远景星尘和满屏漂浮光子。
2. 光子分为远、中、近景三种尺寸，并缓慢循环漂浮。
3. 爱心旋转时，附近光子受距离力场牵引，围绕爱心旋转、适度收拢并略微增亮。

技术要求：
1. 使用 THREE.Points、BufferGeometry 和 ShaderMaterial。
2. 每颗主体粒子保存起点、目标点、延迟、角度、速度、尺寸和路径参数。
3. 地面等待、螺旋上升、贝塞尔吸附和固定四个阶段均在顶点着色器中计算。
4. 使用 gl_PointCoord 绘制圆形粒子。
5. 主体优先使用 NormalBlending，背景柔光可低透明度使用 AdditiveBlending。
6. toneMappingExposure 不超过 0.9，devicePixelRatio 不超过 2。
7. 禁止强 Bloom、大尺寸 PointSprite、白色实心爱心、喷泉和爆炸光团。
8. 动画中必须始终能够分辨单独粒子，并在页面关闭时释放相关资源。
```

## <span style="color: #7B5CD6;">四、总结</span>

AI 可以快速完成 Three.js 场景、粒子属性和 Shader 框架，但最终效果仍取决于几个核心原理：

1. 用隐式三维方程生成连续的爱心体积。
2. 用四阶段运动组织粒子的聚合过程。
3. 用实时旋转目标保证吸附轨迹连续。
4. 用距离力场连接主体与背景光子。

Vibe Coding 降低了编程门槛。

但它没有完全消除时间和 Token 成本。

理解这些原理，可以让需求描述更准确。

也能减少无效修改，更好地判断 AI 生成的结果。

<div style="margin: 20px 0; padding: 15px 17px; border-radius: 8px; background: linear-gradient(135deg, #FAF2F9, #F3F0FF); color: #5A3155; line-height: 1.8;">
<strong style="color: #C445B8;">掌握这套结构后，</strong><br />
目标形状还可以替换成文字、Logo、人物轮廓或产品模型，继续制作粒子聚合、解体和交互动画。
</div>

<div style="text-align: center; margin: 24px 0 8px;">
  <img src="扫码_搜索联合传播样式-白色版-1.png" alt="关注与搜索入口" style="display: block; width: 100%; max-width: 680px; height: auto; margin: 0 auto;" />
</div>
