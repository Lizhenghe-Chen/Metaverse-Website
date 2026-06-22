# 军事 VR 培训 — 手势捕捉交互系统

基于 Unity3D 的 VR 手势捕捉的军事培训软件是元宇宙联合创新实验室开发的一套在 Unity 3D 及多平台 VR 设备（Pico 为主，Oculus、ARCore 等可拓展）运行的基于 OpenXR 标准的虚拟现实 VR 观光及培训软件。

---

## 软件概述

本项目旨在构建一套在 Unity 3D 以及多平台 VR 设备运行的基于 OpenXR 标准的虚拟现实 VR 观光及培训软件。系统通过 Pico 系列 VR 设备，利用其摄像感知系统对用户手势进行捕捉进而实现交互，抛弃了相对传统笨重的手柄控制体系，给用户更加沉浸、易于使用和上手的轻量化交互体验。

本项目为一款基于 Unity XR 框架开发的虚拟现实军事教育互动体验应用，面向 PICO 4 等 OpenXR 兼容 VR 头显设备。系统通过沉浸式虚拟现实技术，构建了包含海上军事讲解、体系化空战模拟、武器装备三维展示等模块的军事教学闭环体验；针对中国部分新型军事装备以及军事战略进行趣味性的交互设计和开发，可以流畅进行端侧渲染和计算，不需要借助额外硬件资源。

### 软件用途

- **军事教育科普**：面向国内高校学生，通过沉浸式 VR 交互学习中国新型军事装备、军事战略与国防知识，将传统枯燥的军事理论转化为可交互、可体验的数字化内容。
- **VR 观光体验**：构建虚拟军事场景（海上军事、空战体系、装备图鉴等），用户可在 360° 全景环境中自由探索。
- **军训培训模拟**：模拟体系化空战流程（预警扫描 → 加速 → 攻击），包含导弹物理追踪、爆炸特效等战斗环节；作为传统军训的补充，提供安全、低成本、可重复的虚拟演练环境。

---

## 创新点

### 无手柄手势交互

利用 PICO 摄像头直接捕捉手部姿态，通过竖拇指等静态手势触发事件，完全抛弃传统 VR 手柄控制。

**优势说明**：

- **降低学习门槛**：用户无需记忆复杂的手柄按键映射，自然手势即可操作，对军事培训场景中的非游戏用户群体（如高校学生、幼童、老人）更友好。
- **提升沉浸感**：双手解放，避免了手柄这一"物理中介"以及额外按键逻辑对虚拟环境沉浸感的割裂。
- **轻量化体验**：省去手柄佩戴、握持、电量管理等环节，设备准备时间大幅缩短，适合大批量轮换使用。

### 多模态军事内容融合

将 3D 装备展示、360° 全景视频、物理导弹模拟、AI 对话、手势交互整合为统一叙事体验。

**优势说明**：

- **多维知识传递**：同一军事主题可通过"看（3D 模型）、听（讲解）、动（手势交互）、战（模拟演练）"多通道强化记忆。
- **趣味性与专业性兼顾**：DOTween 动画、粒子特效、物理导弹追踪等游戏化设计提升吸引力，同时保持军事内容的严谨性。
- **场景灵活组合**：模块化场景设计（海上军事、空战体系、装备图鉴等）可根据教学需求自由搭配，适应不同课时安排。

### OpenXR 跨平台架构

基于 OpenXR 标准开发，以 PICO 为主力平台，架构预留 Oculus、ARCore 等扩展能力。

**优势说明**：

- **避免平台锁定**：不依赖单一厂商 SDK，未来可平滑迁移至其他 VR/AR 设备，保护长期研发投入。
- **降低拓展成本**：新增设备支持时无需重构核心交互逻辑，适配工作量显著减少。
- **符合行业趋势**：OpenXR 为业界统一标准，确保技术方案的先进性和可持续性。

### 端侧全栈渲染与计算

所有图形渲染、语音处理均在 VR 头显端侧完成，无需外接 PC 或云端算力。

**优势说明**：

- **部署成本极低**：无需采购高性能电脑、服务器集群，单台头显即可完整运行。
- **运维简单**：无复杂网络拓扑、服务器维护需求。
- **使用场景灵活**：不受场地电源、网络条件限制，可在普通教室、操场、展厅等多种环境随时开展体验。

### 版本快速更新检测机制

在应用启动时通过 HTTP 请求远程服务器获取最新版本号，与本地版本对比，发现新版本时提供下载按钮（一键跳转下载页）。无需走应用商店审核流程，适合本地教育场景的快速迭代。

---

## 运行环境要求

- **客户端**：PICO 4 等 OpenXR 兼容 VR 头显设备
- **开发端**：Unity 引擎，PICO XR SDK 3.4.0

---

## 技术架构

### 总体架构设计

本应用采用模块化分层架构，从顶层到底层依次为：应用表现层、业务逻辑层、交互控制层、SDK 服务层。

#### 应用表现层

- **UI 系统**：基于 uGUI Canvas 构建，文本渲染使用 TextMeshPro，动画由 DOTween 驱动。
- **视频系统**：支持标准视频（开场背景、教学与谢幕）、360° 全景视频（海上军事讲解）、动态取色功能 VideoColorSkybox 实时采样画面驱动天空盒变色。
- **视觉特效**：VR 屏幕渐隐、开场烟花与导弹爆炸粒子特效、视频驱动的动态天空盒着色。

#### 业务逻辑层

应用包含 6 个注册场景，索引 0–5。支持两种加载方式：

- **标准加载** `SceneManager.LoadScene()`：全量切换
- **叠加加载** `SceneManager.LoadSceneAsync(name, LoadSceneMode.Additive)`：保留当前场景

各场景交互配置：

| 场景 | 交互方式 | 控制器 |
| :--- | :--- | :--- |
| A. 开始 | Poke + 静态手势（点赞） | IntroManager / StartTrigger |
| B. 海上军事讲解 | 静态手势（双手点赞）+ 空间传送 | SeaSceneManager |
| C. 体系化空战 | Poke + 空间传送 | AirBattleStepManager |
| D. 装备一览 | Poke + Pinch（选取/抓取） | GalleryManager3D |
| E. 谢幕视频 | 自动播放 | EndManager |

#### 交互控制层

应用层支持四种 VR 交互模态，由 XR Interaction Toolkit 与 XR Hands 驱动：

| 交互模态 | 说明 | 应用场景 |
| :--- | :--- | :--- |
| **Poke（手指戳按）** | 食指伸展戳向 UI，其余手指蜷曲 | 菜单按钮按压与任务推进 |
| **Pinch（双指捏合）** | 拇指与食指指尖捏合，捏合值 ≥ 0.8 触发 | 远程射线选取菜单与物理抓取装备 |
| **Static Gesture（静态手势）** | 保持特定手型超过 0.2 秒 | 点赞手势触发场景加载 |
| **Teleport（空间传送）** | 传送请求至目标位置 | 空战视角切换与海上场景导航 |

---

## 场景系统设计

系统共包含 6 个核心场景，按业务阶段划分为入场（Entry）、核心体验（Core Experience）、闭幕（Closing）三大组。

### 场景详细规格说明

#### A. 开始（S00）

- **流转起点**：应用冷启动首个挂载点（Build Index 0）。
- **核心组件**：IntroManager、VersionCheck（HTTP 请求热更比对）。
- **流转出口**：用户戳按（Poke）startButton 后，执行全量切换进入 S01。

#### B. 海上军事讲解（S01）

- **核心组件**：SeaSceneManager。
- **默认路径**：通过 VideoPlayer.loopPointReached 事件等待 360° 全景讲解视频播毕，全量切换进入 S02。
- **手势衍生路径**：双手并发 ThumbsUp（点赞）维持 0.2 秒后，以 Additive 形式拉起 S03 在本场景内嵌入展示。

#### C. 体系化空战（S02）

- **核心组件**：AirBattleStepManager 驱动的战局核心控制组件。
- **流转出口**：经过导弹车打击最终目标后，确认主线流程终结并标准切换至 S04。

#### C. 体系化空战 mini（S03）

- **特性配置**：作为加载在 S01 体系之上的 Mini 子模式场景，依赖底图呈现体系化微动。
- **流转出口**：展示流转完结后卸载当前 Additive 层资产，使控制权交还回底层 S01。

#### D. 装备一览（S04）

- **核心组件**：GalleryManager3D。
- **军事装备**：涵盖空中、海上、地面三大领域，共导入 16 种以上高精度 3D 模型。

| 领域 | 装备 |
| :--- | :--- |
| 空中装备 | 歼-20、歼-36、歼-50、歼-16、轰-6K、空警-2000、运-20、F-22、Su-57 |
| 海上装备 | CV-18 福建舰、055 型万吨驱逐舰、002 型航母改进型 |
| 地面装备 | ZTZ-25 主战坦克、东风系列导弹发射车 |

- **流转出口**：带有时间沙漏机制（keepSeconds = 180）进行游玩隔离，超时或全物品巡礼检阅完毕后进入谢幕 S05。

#### E. 谢幕视频（S05）

- **核心组件**：EndManager 以及渲染辅助级 VideoColorSkybox。
- **流转出口**：谢幕视频结束后强制触发闭合回路，调用 FadeManager 遮罩后重定位于场景 S00。

---

## 手势识别配置

### 点赞手势 (StaticHandGesture)

- 通过 XRHandShape ScriptableObject 定义手型（拇指竖起，其余四指握拳）
- 手势保持 0.2 秒后触发 gesturePerformed 事件
- 海上军事讲解场景中组合左右手点赞检测 → 加载 Mini 空战

### Poke 戳按手势 (PokeGestureDetector)

- 食指伸展、其余四指蜷曲时触发
- 指尖与 UI 按钮碰撞后触发 XRSimpleInteractable.selectEntered 事件
- 应用于所有菜单按钮和任务触发按钮

### Pinch 捏合手势

- 拇指与食指捏合值 ≥ 0.8 时触发交互
- 释放阈值 0.25，防止手部抖动误触
- 用于远程射线选取和 XRGrabInteractable 物体抓取

---

## SDK 服务层

应用依赖以下外部 SDK 与服务：

| SDK / 服务 | 版本 | 用途 |
| :--- | :--- | :--- |
| PICO XR SDK | 3.4.0 | PICO 4 设备适配与 PXR_ScreenFade 屏幕渐隐 |
| XR Interaction Toolkit | 3.1.2 | Poke / Grab / Teleport / Socket 等交互框架 |
| XR Hands | 1.5.1 | 手部追踪与手势识别（StaticHandGesture） |
| Input System | 1.14.0 | 统一管理手柄、手部、键盘等多源输入 |
| DOTween (DG.Tweening) | — | 补间动画引擎 |

---

## 场景主控组件技术说明

### S00 IntroManager（开场控制器）

- 输入通道双路触发：同一入口动作可由 startButton.selectEntered（Poke 按压）或"双手点赞"手势共同触发。
- 开场流程状态化：StartIntroInit() 完成时间线内容实例化与初始隐藏；StartShow() 触发倒计时并关闭入口按钮；StartIntro() 接管音视频、粒子与 UI 动画。
- 倒计时技术细节：采用 DOTween.To 驱动数值递减，结合粒子 Burst 动态改写与文本缩放动画，构成"数字 + 视效 + 音效"同步反馈。

### S01 SeaSceneManager（讲解控制器）

- S03 内嵌机制：双手点赞满足后调用 `SceneManager.LoadScene("C. 体系化空战 mini", LoadSceneMode.Additive)`。
- 手势检测防重入：触发后关闭 gestureDetectorObject，降低重复触发概率。
- 视频结束回调：videoPlayer.loopPointReached += OnVideoEnd，播毕后统一走 FadeManager.VRFadeOut() 再切换到 S02。
- 视角迁移：使用 TeleportationProvider.QueueTeleportRequest 将用户传送至 miniAirBattleView。

### S02 AirBattleStepManager（空战流程控制器）

- 任务状态机：通过 SetNewMission(Action, string) 动态替换按钮事件和任务文案，形成"提示 → 触发 → 清理监听 → 下发下一任务"的阶段推进链。
- 容错推进机制：并行运行 CheckTargetViewConstraint() 与 CheckFinalTargetViewConstraint() 轮询 ParentConstraint 源；当源丢失（目标失效）自动推进阶段。
- 多输入兼容：除 XR 按钮外，Keyboard.current.digit1Key 可触发相同 selectEntered 回调，用于调试与演示容错。

### S04 GalleryManager3D（装备展陈控制器）

- 菜单自动生成：SpawnGalleryItems() 运行时根据 galleryItems 生成按钮网格并绑定回调。
- 展示切换原子操作：按钮点击后依次执行 ShowOnlyOneObject.ShowOnly()、介绍文本更新、XRGrabInteractable 复位、语音切换播放。
- 导弹演示事件链：监听 groundToAirMissile.onHit，命中后切断路径动画、切换刚体物理状态并触发坠毁表现。
- 时长收束机制：StartCountDown() 使用 DOTween 驱动 180 秒倒计时，结束后调用 FadeManager.VRFadeOut() 跳转 S05。

### S05 EndManager（谢幕控制器）

- 视频时长绑定倒计时：读取 videoPlayer.clip.length 作为倒计时总时长。
- 回环收束：倒计时完成后统一经 FadeManager.VRFadeOut(() => SceneManager.LoadScene(0)) 返回 S00，保证闭环体验一致。
