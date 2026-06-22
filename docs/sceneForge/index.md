# SceneForge 场景生成系统

SceneForge（面向 Unity 的 3D 室内场景生成软件）是元宇宙联合创新实验室开发的标准化、工业级 3D 室内场景生成平台。系统通过前端 Unity 插件与后端深度生成服务（基于大语言模型与空间布局算法）的深度集成，实现了从自然语言描述到具备物理属性、光影效果的复杂 3D 场景的一键式转化。

---

## 软件概述

本项目旨在构建一套在 Unity 环境中运行的标准化、工业级 3D 室内场景生成平台。系统通过前端 Unity 插件与后端深度生成服务（基于大语言模型与空间布局算法）的深度集成，实现了从自然语言描述到具备物理属性、光影效果的复杂 3D 场景的一键式转化。

### 软件用途

本插件主要面向具身智能（Embodied AI）研究、虚拟现实（VR）开发、数字化转型及数字孪生领域，具体用途包括：

- **具身智能训练环境构建**：为机器人导航、物体抓取等算法提供海量、多样且符合物理规律的 3D 训练场景，降低数据获取成本。
- **医疗/工业仿真模拟**：快速搭建特定的模拟环境（室内多种布局和内饰），结合 Unity 的物理引擎进行软体动力学或交互训练模拟。
- **VR/AR 内容原型设计**：允许开发者通过提示词（Prompt）快速生成室内设计原型，极大地缩短了从创意到 3D 实体的开发周期。
- **科研数据集标准化生成**：支持生成符合 AI2-THOR 规范的标准化场景数据，便于在学术研究中进行结果的可复现与跨平台迁移。

### 创新点

相比于传统的 3D 建模工具或单一的生成模型，本项目在工程化与服务化方面具有以下显著创新：

#### "双轨制"动态资产管线 (Hybrid Asset Pipeline)

系统打破了单一素材库的限制。首创性地结合了 AI2-THOR 本地预置库（保证高频交互物体的性能与稳定性）与 Objathor 远程海量资产库（通过后端实时转换 `.pkl` 模型为 Unity 可读格式）。既保留了本地素材的快速加载能力，又通过云端动态补齐了长尾物件的多样性，解决了 3D 生成中"资产荒"的痛点。

#### 非阻塞式异步生成机制 (Asynchronous Task Orchestration)

针对 LLM 生成耗时长（分钟级）的问题，在 Unity 内部实现了一套完整的任务轮询与状态同步模块。生成过程不会导致 Unity 编辑器假死或主线程卡顿，开发者可以实时监控任务进度，并支持在生成完成后进行二次资产的"按需拉取"。

#### 语义驱动与物理落地的深度融合 (Physically-Grounded Generation)

不仅是生成视觉上的 3D 布局，更在 Unity 插件端集成了自动化物理组件赋予与射线吸附辅助（Raycast Placement）技术。生成的物件自动具备 Collider 和合理的空间层级关系，确保生成的场景"即开即用"，直接支持物理交互与导航烘焙，而非仅供观察的 3D 模型堆砌。

#### 工程友好的自动化索引体系 (Automated Indexing)

内置了 PrefabLookupDatabase 自动构建工具，能通过名称规范化（NormalizeKey）算法自动匹配后端语义标签。极大简化了开发者扩充素材库的难度，只需将新 Prefab 放入指定目录，即可实现后端算法对新资产的自动感知与调用。

---

## 运行环境要求

### 硬件环境

- **客户端**：支持 Unity 2022.3.20f1 LTS 以上 URP 管线的设备
- **服务端**：支持 Docker 或 Conda 环境的服务器

### 软件环境

- **客户端**：Unity 2022.3.20f1 LTS 以上 URP 管线
- **服务端**：Python 3.10+, FastAPI

---

## 技术架构

### 总体架构设计

本项目整体采用前后端分离的异步通信架构。前端作为 Unity 插件，是用户交互的核心入口，负责提示词输入、API 参数配置、生成状态的异步轮询以及最终 3D 场景的实例化与渲染。后端服务则屏蔽了复杂的 AI 推理和资产转换过程，将各种生成模型的输出统一规范为标准的 JSON 场景描述及配套的 3D 资产包。

### 分层架构详细说明

#### 表现层（Presentation Layer）

通过 Unity 自定义编辑器扩展（SceneBuilderRequestEditor 与 SceneBuilderEditor）实现 Inspector 面板交互。提供直观的 API 端点配置、提示词（queryInput）输入框、日志输出面板，以及标准化的三阶段操作流按钮（1. Generate Scene, 2. Request Processed Assets, 3. 生成场景/清除场景/赋予碰撞体）。

#### 业务逻辑层（Business Logic Layer）

由核心脚本 SceneBuilder 驱动。负责解析从服务端下载的 JSON 场景数据，将其映射为 Unity 内部的数据结构（如 RoomData, WallData, ObjectData 等）。根据层级关系依次生成地面、墙体、天花板、门窗及各类物件。支持通过高度适配与射线检测算法，确保物体物理位置的合理性。

以 SceneBuilder 为总协调器，采用 Builder 设计模式和管线化架构，按照固定顺序依次调用各生成器模块，Builder 模块均为无状态静态类，通过统一的 SceneData 数据模型进行数据交换，松耦合设计使新增或替换生成器无需修改其他模块。

#### 基础服务层（Data Service Layer）

负责处理非渲染类的底层任务，包括：异步 HTTP 网络通信（请求发起与文件下载）、本地 Prefab 索引库的自动化构建与匹配（通过 PrefabLookupDatabaseBuilder），以及对批量导入的模型资产进行自动化物理组件配置。

---

## 关键技术接口

为了实现完整的场景生成管线，客户端主要与服务端的以下关键接口进行对接：

| 接口 | 路径 | 说明 |
| :--- | :--- | :--- |
| Generate Endpoint | `/generate` | 异步提交生成任务，传递场景提示词及约束参数，返回任务唯一标识符 (Task ID) |
| Status Endpoint | `/status/{task_id}` | 轮询查询任务进度，若任务完成则返回场景 JSON 的下载路径 |
| Download Endpoint | `/download` | 获取并下载结构化的场景描述 JSON 文件至本地 jsonDataSaveFolder |
| Asset Process Endpoint | `/process_scene_json` | 提交 JSON 数据请求处理并下载场景中包含的美术资源模型压缩包 |

---

## 核心功能模块

### 网络连接与服务模块

网络连接模块负责 Unity 客户端与 FastAPI 后端服务之间的数据交换。由于 3D 场景生成涉及大语言模型（LLM）推理与空间几何优化，过程具有高延迟特征，因此该模块采用了基于任务 ID 的异步解耦通信架构。

**通信协议与数据交互规范**：

- 通信协议：采用标准 RESTful API 架构，基于 HTTP/1.1 协议进行交互。
- 数据格式：控制信令采用 `application/json` 格式；资源数据通过 `application/zip` 流式传输。
- 异步机制：前端发送 POST 请求后立即获取 `task_id`，随后进入非阻塞式的 GET 轮询状态，确保 Unity 主线程在生成期间不会卡死。

**状态机驱动的轮询逻辑**：

- **Idle（空闲）**：等待用户输入提示词。
- **Requesting（请求中）**：向服务端提交 SceneRequest 结构体。
- **Polling（轮询中）**：根据预设的时间间隔（默认 1-2s）请求服务器，解析返回的 status 字段（pending / running / completed / failed）。
- **Success/Error（终态）**：任务成功后触发 DownloadJson 回调；若失败则捕获 HTTP Exception 并通过日志系统抛出。

### 场景解析与构建逻辑模块

该模块是插件的核心，负责将 JSON 文本转化为可见的 Unity 场景：

- **层级化实例化**：脚本严格按照 rooms 数据分组，将场景内的资产分类为 `floor_objects`（地面物件）、`wall_objects`（墙面物件）、`small_objects`（桌面/小型物件）和 `ceiling_objects`（天花板物件），并建立清晰的父子级 Transform 关系。
- **空间高度自适应**：系统通过 WallBuilder 动态提取房间的墙体高度（Wall Height），并利用该高度信息对壁画、吊灯等特定分类的物体进行 Y 轴坐标的自动适配修正。
- **射线吸附技术（Raycast Placement）**：提供精准的物理落点对齐功能。当开启 `useRaycastPlacement` 时，在生成小型悬浮物件（如切菜板、水杯）时会向下发射检测射线，自动将物体底部贴合至正下方最近的模型表面（如橱柜顶面），解决生成坐标精度不足导致的穿模或悬空问题。

```
positionToBoundsBottom = transform.position.y - bounds.min.y
newY = bestHitY + positionToBoundsBottom + SurfaceSnapGap
```

其中 `SurfaceSnapGap = 0.01m` 为保留的微小空隙，防止物体与贴合表面因浮点精度产生 Z-Fighting 闪烁问题。

- **墙体开口**：每遇到一个开口，墙体被分为三段（开口前 / 开口下 / 开口上），currentPos 指针循环推进确保无重叠。
- **物理解析**：地面物体（floor_objects）非 Kinematic 受重力影响；墙面物体（wall_objects）Kinematic 不受重力；小型物体（small_objects）由 `useRaycastPlacement` 参数控制。自动扫描已有 MeshCollider，非凸包时自动设置为 Convex；无 MeshCollider 时自动遍历 MeshFilter 添加新碰撞体。
- **光源转换规则**：`"directional"` → Directional（方向光）、`"spot"` → Spot（聚光灯）、`"area"` → Rectangle（区域光）、其他 → Point（点光源）。

### 资产处理与持久化模块

- **资产请求与解包**：客户端根据 JSON 中的 assetId 向服务端请求具体模型文件，下载并解压存入 `Assets/Resources/ExportModels` 目录。
- **物理组件自动化**：提供工具遍历该目录下的所有 `.obj` 模型文件，通过反射 Unity 的 ModelImporter API，自动勾选 `addCollider` 属性并强制触发重新导入（Reimport）。

---

## 数据库设计

本项目的资产管理系统采用"本地预置+动态下发"的双轨制设计，旨在平衡生成场景的即时性与素材的多样性。

### Unity 本地静态素材库 (AI2-THOR 兼容库)

- **存储架构**：实体层以 Unity Prefab 形式存储在 `Assets/HoloSceneAssets` 目录下，遵循 AI2-THOR 的分类命名规范。
- **索引层（PrefabLookupDatabase）**：采用 ScriptableObject 实现轻量级数据库。通过 PrefabLookupEntry 结构体维护 key（资产标识符）与 GameObject（预制体引用）的映射关系。
- **自动化构建机制**：通过 PrefabLookupDatabaseBuilder 编辑器工具，系统可自动遍历资产目录，利用 NormalizeKey 算法消除大小写和特殊字符差异，生成全局唯一的资产索引表。

### 服务端动态资产库 (Objathor 预处理库)

针对 LLM 生成的高多样性物件需求，系统通过服务端对接 Objathor 资产库。该库包含数万计经过预处理的 3D 模型，支持复杂的语义检索。

- **资产收集**：后端解析生成后的场景 JSON，通过 `collect_asset_ids` 递归提取所有引用的动态模型 ID。
- **匹配与校验**：`match_asset_id` 逻辑会对比本地存储目录，确保所需资产存在且完整。
- **按需打包**：系统将选中的资产通过 `export_matched_assets` 转化为可供 Unity 直接读取的格式（如转换 `.pkl` 为 `.obj`），并实时封装为 `scene_assets.zip` 供客户端下载。

---

## 服务端部署说明

### 数据下载

```bash
python -m objathor.dataset.download_holodeck_base_data --version 2023_09_23
python -m objathor.dataset.download_assets --version 2023_09_23
python -m objathor.dataset.download_annotations --version 2023_09_23
python -m objathor.dataset.download_features --version 2023_09_23
```

### Conda 安装

```bash
conda create --name holoscene python=3.10
conda activate holoscene
pip install -r requirements.txt
```

### Docker 安装

```bash
docker run -d --name holoscene -p 8000:8000 --shm-size=2g \
  -e OPENAI_API_BASE=http://10.120.47.138:8000/v1/ \
  -e OPENAI_API_KEY='sk-' \
  -e LLM_MODEL_NAME=qwen3.5-27B \
  -e OBJATHOR_ASSETS_BASE_DIR=/data/objathor-assets \
  -v /Volumes/DoggyChen/objathor-assets:/data/objathor-assets:rw \
  holoscene-holoscene:latest
```

---

## 客户端使用说明

1. 将 `Unity HoloScene场景生成.unitypackage` 拖入 Unity 的文件目录窗口，等待解压后导入。
2. 打开 `Assets/GenScene/Scenes/GenScene.unity` 场景，选中 Gen 即可在 Inspector 中看到插件界面。
3. 配置 `queryInput`（场景生成的提示词）以及服务端配置好的 `baseUrl` API 接口地址。
4. 确保 `jsonDataSaveFolder`、`objectsFolder` 等文件夹指定不为空。
5. 如需更新素材索引，点击 `Tools/GenScene/Rebuild Prefab Lookup Database` 菜单。
6. 按插件界面的三步提示按钮依次点击：**Generate Scene** → **Request Processed Assets** → **生成场景**。

!!! warning "注意"
    生成过程不要操作 Unity 和编辑 Unity 脚本，否则会打断生成通讯！整个生成过程一般在 10 分钟左右，与后端大模型性能高度相关。

---

## 模块列表

| 模块名称 | 描述 | 技术栈 |
| :--- | :--- | :--- |
| Holodeck Plugin | 基于 Unity 的场景生成插件 | Unity, C# |

## 了解更多

请查看各模块的详细技术文档。
