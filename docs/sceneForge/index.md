# SceneForge 场景生成系统

面向 Unity 的 3D 室内场景生成平台，通过前端 Unity 插件与后端深度生成服务（LLM + 空间布局算法）的集成，实现从自然语言描述到 3D 场景的一键式转化。主要面向具身智能训练、VR/AR 原型设计、医疗/工业仿真及科研数据集生成。

<div class="download-buttons">
  <a class="md-button md-button--primary" href="https://gofile.me/7U4bu/bbxMTlVwm" target="_blank" rel="noopener noreferrer">⬇️ 下载 Unity 插件 (.unitypackage)</a>
  <a class="md-button md-button--primary" href="https://gofile.me/7U4bu/RLbhv081O" target="_blank" rel="noopener noreferrer">🐳 下载 Docker 镜像</a>
</div>

| 环境             | 要求                                   |
| :--------------- | :------------------------------------- |
| **客户端** | Unity 2022.3.20f1 LTS 以上 URP 管线    |
| **服务端** | Python 3.10+, FastAPI / Docker / Conda |

## 技术架构

前后端分离异步通信架构。前端 Unity 插件负责交互与场景实例化；后端屏蔽 AI 推理与资产转换，输出标准 JSON 场景描述及 3D 资产包。

- **表现层**：Unity 编辑器扩展（SceneBuilderRequestEditor / SceneBuilderEditor），提供 API 配置、提示词输入、日志面板及三阶段操作按钮。
- **业务逻辑层**：SceneBuilder 驱动，按 GroundBuilder → WallBuilder → CeilingBuilder → WindowBuilder → DoorBuilder → ObjectBuilder → LightBuilder 管线化顺序执行，Builder 均为无状态静态类。
- **基础服务层**：异步 HTTP 通信、PrefabLookupDatabaseBuilder 索引构建、批量模型物理组件自动化配置。

## 关键技术接口

为了实现完整的场景生成管线，客户端主要与服务端的以下关键接口进行对接：

| 接口                   | 路径                    | 说明                                                                     |
| :--------------------- | :---------------------- | :----------------------------------------------------------------------- |
| Generate Endpoint      | `/generate`           | 异步提交生成任务，传递场景提示词及约束参数，返回任务唯一标识符 (Task ID) |
| Status Endpoint        | `/status/{task_id}`   | 轮询查询任务进度，若任务完成则返回场景 JSON 的下载路径                   |
| Download Endpoint      | `/download`           | 获取并下载结构化的场景描述 JSON 文件至本地 jsonDataSaveFolder            |
| Asset Process Endpoint | `/process_scene_json` | 提交 JSON 数据请求处理并下载场景中包含的美术资源模型压缩包               |

---

## 数据库设计

采用"本地预置+动态下发"双轨制。

### Unity 本地静态素材库 (AI2-THOR 兼容库)

- **存储**：Prefab 形式存储在 `Assets/HoloSceneAssets`，遵循 AI2-THOR 分类规范。
- **索引**：ScriptableObject 实现的 PrefabLookupDatabase，维护 key 与 GameObject 的映射。
- **自动化构建**：PrefabLookupDatabaseBuilder 遍历资产目录，NormalizeKey 算法消除大小写差异，生成唯一索引。

### 服务端动态资产库 (Objathor 预处理库)

- **资产收集**：后端解析场景 JSON，`collect_asset_ids` 递归提取动态模型 ID。
- **匹配校验**：`match_asset_id` 对比本地存储目录，确保资产完整。
- **按需打包**：`export_matched_assets` 转换 `.pkl` 为 `.obj`，封装为 `scene_assets.zip` 供客户端下载。

---

## 服务端部署说明

### 数据下载

```bash
python -m objathor.dataset.download_holodeck_base_data --version 2023_09_23
python -m objathor.dataset.download_assets --version 2023_09_23
python -m objathor.dataset.download_annotations --version 2023_09_23
python -m objathor.dataset.download_features --version 2023_09_23
```

默认情况下，数据保存至 `~/.objathor-assets/`。如需更改路径，通过 `--path` 参数指定。若更改了路径，使用 holoscene 时需设置环境变量 `OBJAVERSE_ASSETS_DIR` 指向数据存储路径。

### Conda 安装

```bash
conda create --name holoscene python=3.10
conda activate holoscene
pip install -r requirements.txt
```

### Docker 安装

#### 方式一：使用导出的镜像文件

```bash
# 导出镜像（开发者）
docker save holoscene-holoscene:latest | gzip > holoscene.tar.gz

# 导入镜像（用户）
gunzip -c holoscene.tar.gz | docker load

# 运行容器（替换为你自己的 API Key 及数据路径）
docker run -d --name holoscene -p 8000:8000 --shm-size=2g \
  -e OPENAI_API_BASE=http://10.120.47.138:8000/v1/ \
  -e OPENAI_API_KEY='sk-' \
  -e LLM_MODEL_NAME=qwen3.5-27B \
  -e OBJATHOR_ASSETS_BASE_DIR=/data/objathor-assets \
  -v /Volumes/DoggyChen/objathor-assets:/data/objathor-assets:rw \
  holoscene-holoscene:latest
```

#### 方式二：源码构建镜像

1. （中国大陆用户）配置 Docker 镜像加速器：

   Docker Desktop: Preferences → Docker Engine，修改 `registry-mirrors`。

   Linux: 编辑 `/etc/docker/daemon.json`：

   ```json
   {
     "registry-mirrors": [
       "https://docker.1ms.run",
       "https://dockerproxy.net",
       "https://docker.m.daocloud.io"
     ]
   }
   ```
2. 修改 `docker-compose.yml` 中的 `volumes` 字段，指向数据路径：

   ```yaml
   volumes:
     - /Volumes/DoggyChen/objathor-assets:/Volumes/DoggyChen/objathor-assets:rw
   ```
3. 构建并启动容器：

   ```bash
   docker compose up -d --build
   ```
4. 首次运行（中国大陆用户需开启 VPN），执行 `bash init.sh`。若看到 `Generation complete for xxx. Scene saved and any other data saved to /app/data/scenes/...` 则表示生成成功。
5. 服务默认监听 `8000` 端口。

---

## 客户端使用说明

### 插件导入

1. 将 `Unity HoloScene场景生成.unitypackage` 拖入 Unity 文件目录窗口，等待解压后全部导入。
2. 打开 `Assets/GenScene/Scenes/GenScene.unity` 场景，选中 Gen 即可在 Inspector 中看到插件界面。

### 生成前配置

1. **queryInput**：填写场景生成的提示词（如"一个带有厨房和卧室的两居室公寓"）。
2. **baseUrl**：配置服务端 API 接口地址。
3. 确保 `jsonDataSaveFolder`、`objectsFolder` 等文件夹指定不为空，其余变量保持默认。
4. 如需更新素材索引，点击菜单 **Tools/GenScene/Rebuild Prefab Lookup Database**，将新放入 `HoloSceneAssets` 文件夹的模型登记到本地数据库。

### 三步生成流程

按插件界面按钮依次操作：

1. **Generate Scene** — 向服务端提交生成任务。请求成功后，输出窗口（Log Output）会持续返回日志信息。生成过程约 10 分钟，与服务端大模型性能相关。
2. **Request Processed Assets** — 向服务端请求美术资源（需服务端正确配置资产文件路径）。导入成功后，素材保存在 `Assets/Resources/ExportModels` 目录下。
3. **生成场景** — 将 JSON 场景数据实例化为完整的 3D 场景，包含地面、墙体、天花板、门窗及各类内饰物件。

### 生成后操作

- **赋予碰撞体**：为场景中所有物件自动添加物理碰撞组件。
- **清除场景**：清空当前生成的场景内容。
- **物理组件自动化**：使用工具遍历 `Assets/Resources/ExportModels` 下的 `.obj` 文件，自动勾选 `addCollider` 并触发重新导入，使模型支持物理碰撞模拟、NavMesh 烘焙及光线遮挡剔除。

!!! warning "注意"
    生成过程不要操作 Unity 和编辑 Unity 脚本，否则会打断生成通讯！整个生成过程一般在 10 分钟左右，与后端大模型性能高度相关。由于大模型生成的随机性，生成结果不一定每次都能满足输出格式，存在小概率失败的可能。

---

## 模块列表

| 模块名称        | 描述                      | 技术栈    |
| :-------------- | :------------------------ | :-------- |
| Holodeck Plugin | 基于 Unity 的场景生成插件 | Unity, C# |

## 了解更多

请查看各模块的详细技术文档。
