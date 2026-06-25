# DeepOwl AI 动作生成系统

DeepOwl AI 动作生成系统（亦称 AIMotion）是一套从前端到后端的完整文本驱动 3D 人体动作生成平台。前端为 Blender 4.2+ 插件，后端基于 Ray Serve 分布式部署框架，集成 MoMask（单人文本驱动）与 InterGen（双人交互生成）两大模型，实现从自然语言描述到 3D 角色动画的一键式转化。主要面向 3D 动画制作、游戏开发、虚拟现实、影视预演及教育科研等场景。

<div class="download-buttons">
  <a class="md-button md-button--disabled">⬇️ 下载 Blender 插件 (.zip)</a>
  <a class="md-button md-button--disabled">🐳 下载后端 Docker 镜像</a>
</div>

| 环境 | 要求 |
| :--- | :--- |
| **客户端** | Blender 4.2 LTS 及以上，支持 Windows / macOS / Linux |
| **服务端** | Python 3.10+，NVIDIA GPU（显存 ≥ 24 GB，推荐 A100/H100），Docker / Ray Serve |

## 技术架构

系统采用前后端分离的四层架构设计，各层职责清晰、接口标准化，支持独立升级与横向扩展。

- **用户交互层（User Interaction Layer）**：面向终端用户的操作入口，包括 Blender 4.2+ 插件（主要交互方式）、MLOps Web 控制面板（运维管理入口）以及第三方客户端（REST API 调用）。负责用户输入接收、参数校验、结果展示和状态反馈。
- **API 网关层（API Gateway Layer）**：系统的统一入口和流量调度中心，基于 FastAPI 实现。提供负载均衡、请求路由（`/forward`、`/redict`、`/serve` 等端点）、API Key 认证鉴权和速率限制（Rate Limiting）功能。
- **模型服务层（Model Serving Layer）**：核心业务逻辑处理层，基于 Ray Serve 分布式部署框架构建。包含 MoMask Deployment（单人文本驱动动作生成）、InterGen Deployment（双人交互动作生成）和 MotionAPI Deployment（通用动作服务接口）。
- **基础设施层（Infrastructure Layer）**：底层资源支撑，包括 GPU 计算集群（NVIDIA A100/H100，CUDA 11.8+）、Ray Cluster（Head Node + Worker Nodes，支持自动扩缩容）、Docker 容器运行时以及模型持久化存储（checkpoint 目录存储 `*.pt` 权重文件，同时支持 ONNX/NCNN 格式导出）。

## 核心功能

| 功能模块 | 描述 | 技术栈 |
| :--- | :--- | :--- |
| **文本驱动动作生成** | 输入自然语言描述，MoMask 模型生成对应 3D 单人骨骼动画 | VQ-VAE + Transformer, MoMask |
| **双人交互动作生成** | 输入两个角色的动作描述，InterGen 模型生成体现交互关系的协同动画 | Cross-Attention, InterGen |
| **骨骼自动绑定** | 三级匹配算法（直接匹配 → 字典映射 → 模糊匹配）将标准骨骼动作映射到目标角色骨骼 | Blender Python API (bpy) |
| **动画关键帧烘焙** | 将 BVH 动作数据烘焙为 Blender NLA 动作条带和视觉关键帧，支持直接编辑与导出 | BVH Parser, NLA Bake |
| **动画切片与复制** | 基于 Blender NLA 编辑器，对动画片段进行循环、偏移、拼接与混合 | NLA Action Strip |
| **MLOps 运维面板** | 模型部署管理、任务调度监控、系统资源监控、告警通知 | FastAPI, Ray Serve, Web UI |

## 前后端通信流程

以单人文本驱动动作生成为例，标准通信时序如下：

1. **输入动作描述**：用户在 Blender 插件文本框中输入自然语言描述（如 "A person walks forward and waves right hand"），选择目标骨骼模型，点击 **生成动作** 按钮。
2. **前端预处理**：插件对文本进行清洗、长度截断，并提取当前选中骨骼的名称列表与层级关系。
3. **发送请求**：插件通过 HTTP POST 将 JSON 请求体发送至后端 `/forward` 端点，包含 `text`、`skeleton` 和 `timestamp` 字段。
4. **网关路由**：FastAPI 网关完成 API Key 认证与格式校验，将请求路由至 MoMask Deployment。
5. **模型推理**：Ray Serve 将请求分配给可用的 MoMask 副本，在 GPU 上执行前向推理（VQ-VAE 编码 + Transformer 解码）。
6. **返回 BVH 数据**：推理完成后，动作数据被编码为 BVH 格式（包含骨骼旋转/位移关键帧序列），通过 HTTP Response 返回给 Blender 插件。
7. **骨骼映射与烘焙**：插件执行骨骼映射、动画烘焙，生成 NLA 关键帧和视觉关键帧，最终在 Blender 视口中展示动画效果。

## 部署方式

### Blender 插件安装

1. 下载插件安装包 `ai_motion_generator.zip`。
2. 打开 Blender，点击菜单栏 **编辑 → 偏好设置 → 插件**。
3. 点击 **安装...** 按钮，选择下载的 `.zip` 文件。
4. 在搜索框中输入 "AI Motion"，勾选启用该插件。
5. 在插件设置面板中配置后端服务器地址（默认 `http://localhost:8000`）和 API Key。
6. 点击 **保存用户设置** 完成安装。

### 后端服务部署（Docker Compose）

```bash
# 1. 克隆部署仓库
git clone https://github.com/aimotion/deploy.git
cd deploy

# 2. 配置环境变量
cp .env.example .env
# 填写 RAY_ADDRESS、MODEL_PATH、API_KEY 等参数

# 3. 启动服务
docker-compose up -d

# 4. 验证服务状态
curl http://localhost:8000/health
# 应返回 {"status": "ok"}
```

!!! warning "注意"
    后端服务对 GPU 显存要求较高，MoMask 与 InterGen 每个副本建议配置 1 块显存不低于 24 GB 的 NVIDIA GPU。首次部署前请确保已安装 Docker 24.0+ 与 NVIDIA Container Toolkit。

## 应用场景

- **3D 动画制作**：为角色动画提供快速原型生成能力，通过文本描述直接生成基础动作，大幅缩短制作周期。
- **游戏开发**：为游戏角色生成行走、奔跑、跳跃、攻击等标准化动作，支持导入 Unity、Unreal 等主流游戏引擎。
- **虚拟现实与教育科研**：为虚拟课堂、VR 体验、人机交互研究提供批量动作数据生产能力。
- **影视预演**：为导演和动作指导提供快速的可视化预演，支持迭代调整。

## 模块列表

| 模块名称 | 描述 | 技术栈 |
| :--- | :--- | :--- |
| MoMask 核心逻辑模块 | 文本驱动动作生成的核心推理逻辑 | Python, PyTorch, MoMask |
| 操作符模块 | Blender 插件中的核心操作符实现 | Blender Python API (bpy) |
| UI 面板模块 | Blender 插件的图形化界面布局 | Blender Python API (bpy) |
| BVH 转换工具模块 | BVH 格式解析与 Blender 关键帧转换 | Python, BVH |
| 热加载工具模块 | 插件运行时模块热更新机制 | Blender Python API (bpy) |

## 了解更多

请查看各模块的详细技术文档，深入了解系统实现细节。
