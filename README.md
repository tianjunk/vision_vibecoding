# 🎨 Vision Vibecoding

> 基于 Web 摄像头 + AI 手势识别的交互式视觉特效项目集合。  
> 使用 **MediaPipe Hands/SelfieSegmentation** + **Three.js** + **WebGL** 实现实时手势驱动的视觉体验。

---

## 📁 项目结构

```
vision_vibecoding/
│
├── gesture-water-person/        ← 🫧 透明水滴代码人
│   ├── index.html               # 主页面
│   └── explosion-test.png       # 效果截图
│
├── gesture-football/            ← ⚽ 手势足球
│   ├── index.html               # 主页面
│   ├── football-ready.png       # 就绪界面
│   ├── football-flight.png      # 射门飞行
│   ├── football-result.png      # 进球结果
│   ├── football-hand-overlay.png # 手势叠加层
│   ├── football-controls-ready.png  # 双指操控
│   └── football-two-finger-controls.png
│
├── gesture-particle-orb/        ← 🔮 手势粒子球
│   ├── index.html               # 主页面
│   ├── particle-orb-normal.png  # 正常状态
│   ├── particle-orb-compressed.png # 压缩状态
│   ├── particle-orb-spread.png  # 扩散状态
│   └── new-glass-test.png       # 玻璃效果测试
│
├── particle-heart.html          ← 💖 粒子爱心（主页面）
│
├── articles/                    ← 📝 公众号文章
│   ├── Three.js手势粒子球.md
│   └── Three.js粒子爱心.md
│
└── assets/                      ← 🖼️ 共享资源
    ├── 屏幕录制*.mp4             # 屏幕录制视频
    └── 扫码_*.png               # 公众号二维码
```

---

## 🫧 gesture-water-person — 透明水滴代码人

**核心技术：** WebGL Shader + MediaPipe Selfie Segmentation + MediaPipe Hands

### 功能说明
打开摄像头后，你的身体会被渲染成**透明水滴/玻璃质感**的人形，身体轮廓内部流动着绿色的 **「黑客帝国」风格代码雨**。

- 🧊 **玻璃折射**：身体呈现透明水滴的折射、焦散光效
- 💚 **代码雨**：绿色字符在身体轮廓内部持续下落，密度可覆盖全身
- 🌀 **螺旋丸（Rasengan）**：手部快速挥动可投掷蓝色能量球，碰撞屏幕中心产生爆炸粒子效果
- 💥 **玻璃碎裂**：爆炸时产生裂纹和飞溅碎片特效

### 使用方式
1. 浏览器打开 `index.html`
2. 允许摄像头权限
3. 站在镜头前，身体轮廓自动识别
4. **快速挥手**投掷螺旋丸

### 技术亮点
- 自研 WebGL 片元着色器，实时计算折射/焦散/边缘光
- 人体分割蒙版（Selfie Segmentation）与手势追踪（Hands）并行运行
- 5000+ 粒子爆炸系统 + 蛛网裂纹特效

---

## ⚽ gesture-football — 手势足球

**核心技术：** Three.js 3D 渲染 + Cannon-es 物理引擎 + MediaPipe Hands

### 功能说明
用手势控制踢足球！一个完整的 3D 足球射门游戏。

- 🏟️ **3D 场景**：完整的足球场、观众席、球门、门将
- 👆 **双指操控**：
  - **食指** = 左脚（蓝色）→ 射门带左弧线
  - **中指** = 右脚（橙色）→ 射门带右弧线
- 🥅 **瞄准系统**：手部位置控制瞄准方向，HUD 显示力度条和准星
- 🤖 **AI 门将**：自动扑救，对射门方向做出反应
- 📊 **计分系统**：常规进球 1 分，死角进球 2-3 分，连击加成
- ⏱️ **60 秒回合制**：时间到显示总分，自动重新开始

### 使用方式
1. 浏览器打开 `index.html`
2. 允许摄像头权限
3. 手掌对准摄像头，移动**瞄准球门**
4. **快速弯曲食指或中指**完成射门
5. 无摄像头时可用**鼠标点击**或**空格键**射门

### 技术亮点
- 物理引擎驱动的足球轨迹（重力、摩擦力、碰撞检测）
- Bezier 曲线指导弹道预测线
- 自定义足球纹理 + 草地程序化纹理
- 3200 个观众粒子点云

---

## 🔮 gesture-particle-orb — 手势粒子球

**核心技术：** Three.js ShaderMaterial + MediaPipe Hands

### 功能说明
一个由 **30,000 个粒子**组成的能量球，实时响应手势变化。

- 🖐️ **手势控制**：
  - **捏合手指** → 球体收缩
  - **张开手掌** → 粒子扩散爆发
  - **握拳** → 球体压缩变红
  - **移动手** → 球体跟随位移/旋转
- ✨ **星空背景**：9000 颗星星 + 1800 个尘埃粒子
- 🌈 **颜色变化**：从蓝青色（冷）渐变到橙红色（热），反映压缩程度

### 使用方式
1. 浏览器打开 `index.html`
2. 允许摄像头权限
3. 手掌在镜头前做各种手势，观察粒子球实时变化

### 技术亮点
- 30,000 粒子自定义着色器，GPU 实时计算位移/颜色/大小
- 粒子方向向量 + 切向量波动的体感运动
- 菲涅尔核心光晕效果

---

## 💖 particle-heart — 粒子爱心

**核心技术：** Three.js ShaderMaterial（纯前端，无需摄像头）

### 功能说明
**纯视觉体验** — 约 17,600 个粒子从地面旋转升空，汇聚成一个立体的粉紫色爱心。

- 💫 **聚拢动画**：粒子从地面螺旋上升→弧线飞行→精准落位形成爱心
- 🖱️ **交互旋转**：鼠标/手指拖拽可旋转爱心视角（带惯性）
- 🌌 **浪漫背景**：程序化渐变天幕 + 3600 颗闪烁星星 + 2600 个漂浮光点
- 💡 **光点被爱心吸引**：背景光点会围绕爱心缓慢旋转

### 使用方式
1. 浏览器直接打开 `index.html`（无需摄像头）
2. **鼠标拖拽**旋转视角
3. 调用 `window.restartParticleHeart()` 重新播放聚拢动画

### 技术亮点
- 隐式曲面（心形方程）采样，粒子精准分布在爱心表面/内部
- 分层延迟策略：外壳→表面→内部，形成立体填充感
- 自定义贝塞尔曲线飞行路径
- 背景光点着色器实现吸引力轨道效果

---

## 🛠️ 技术栈总览

| 技术 | 用途 |
|------|------|
| **MediaPipe Hands** | 手部 21 个关键点实时追踪 |
| **MediaPipe Selfie Segmentation** | 人体/背景分割蒙版 |
| **Three.js v0.160** | 3D 场景渲染 |
| **Cannon-es** | 物理引擎（足球碰撞/重力） |
| **WebGL Shader (GLSL)** | 自定义着色器（折射、粒子、代码雨） |
| **Canvas 2D** | 手势叠加层绘制 / 程序化纹理 |

## 🚀 快速启动

所有项目均为**纯静态 HTML**，无需构建工具：

```bash
# 方式一：直接用浏览器打开任意 index.html
# 方式二：启动本地服务器（推荐，摄像头 API 需要）
npx serve .
# 然后访问 http://localhost:3000/gesture-football/
```

> ⚠️ **注意：** 摄像头相关项目需要 **HTTPS 或 localhost** 环境。直接用 `file://` 协议打开可能无法调用摄像头。

---

*Made with ❤️ by Vibecoding, powered by Claude Code & Three.js*
