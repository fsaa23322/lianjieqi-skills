---
name: blender-cn-starter
description: 用中文指挥 Blender 做 3D 建模的入门技能。用户说「做个3D模型」「Blender建模」「建一张桌子/房间/场景」「做3D场景」「改材质」「换颜色」「渲染出图」「3D出图」「生成3D模型」「混元3D」时使用。通过开源的 MCP for Blender 连接本机 Blender，按「先确认连接→分步搭建→多角度截图自检→保存工程→渲染出图」的流程完成；需要现成模型时可选用腾讯混元3D（用户自己的腾讯云密钥）或 Poly Haven 等素材库。由智能体连接器（lianjieqi.ai）编写并实测。
license: MIT
metadata:
  author: 智能体连接器（lianjieqi.ai）
  version: "1.0.0"
  homepage: https://lianjieqi.ai/connectors/blender-cn-starter/
---

# Blender 中文建模助手

让完全没用过 Blender 的人，用一句中文就能搭出 3D 物体或场景，并拿到一张渲染图和可继续编辑的 .blend 工程文件。

## 依赖（先检查，缺了就告诉用户怎么装）

| 依赖 | 作用 | 是否必需 | 国内网络能否直接用 |
| --- | --- | --- | --- |
| Blender 3.0 及以上（本机） | 真正干活的 3D 软件 | 必需 | 可直接用（本机运行；安装包从 blender.org 下载） |
| MCP for Blender（开源，`uvx mcp-for-blender`） | 让智能体操作 Blender | 必需 | 可直接用（本机 socket 通信；首次安装需从 PyPI 下载） |
| 腾讯混元3D（腾讯云 AI3D 接口，用户自己的 SecretId/SecretKey） | 文字/图片生成现成 3D 模型 | 可选 | 可直接用（大陆账号走 ap-guangzhou） |
| Poly Haven / Sketchfab / Poly Pizza 素材库 | 下载现成模型、贴图、HDRI | 可选 | 依赖海外服务，可能较慢或无法访问 |

安装说明见 https://lianjieqi.ai/connectors/blender-mcp/ 。MCP 配置建议加上关闭遥测的环境变量：

```json
{ "mcpServers": { "blender": { "command": "uvx", "args": ["mcp-for-blender"], "env": { "DISABLE_TELEMETRY": "true" } } } }
```

注意：这是**本机 STDIO 连接器**，智能体必须能访问用户这台电脑（千问办公、豆包工作桌面端的自定义 STDIO 连接器、Claude Code、Cursor 等可以）。如果智能体的任务跑在云端容器里，连不上用户本机的 Blender，要直接告诉用户，不要假装完成。

## 工作流程

### 第 1 步：确认连接
1. 调用 `get_addon_status`。能返回 Blender 版本就说明连上了。
2. 连不上时，按顺序提示用户：打开 Blender → 在 3D 视图按 N 打开侧栏「MCP for Blender」→ 点击连接/启动服务；仍不行就执行 `uvx mcp-for-blender install-addon` 后重启 Blender。同一时间只能有一个客户端连接。
3. 从返回结果里记下哪些素材库和生成器是开着的（`libraries`、`generators`），后面只用开着的。

### 第 2 步：把用户的话变成清单
先用 3～6 行中文复述要做什么，再动手：物体有哪些、大概尺寸（米）、颜色/材质、摆放关系、要不要出图、图片比例。用户说得含糊时按常见尺寸取值（桌高约 0.75 米、椅座高约 0.45 米、门高约 2 米），并在复述里写明。

### 第 3 步：分步搭建（每步一小段代码）
用 `execute_blender_code` 执行 Blender Python（bpy），**每次只做一件事**（例如先做桌面，再做四条腿，再上材质），不要一次塞几百行。要点：
- 给每个物体起中文名，例如 `obj.name = '桌面'`，方便用户在大纲里找到。
- 尺寸用米，物体放在地面（z=0）以上。
- 材质用 Principled BSDF，例如：
  ```python
  mat = bpy.data.materials.new('木纹色'); mat.use_nodes = True
  mat.node_tree.nodes['Principled BSDF'].inputs['Base Color'].default_value = (0.45, 0.28, 0.15, 1)
  obj.data.materials.append(mat)
  ```
- 场景里没有灯光和相机时补上（太阳光 + 一台对准主体的相机，并设为 `scene.camera`）。
- 删除或清空场景前必须先问用户，并建议先另存一份。

### 第 4 步：自己检查，再给用户看
1. 调用 `get_scene_info` 核对物体数量、位置、尺寸是否和清单一致。
2. 调用 `look`，`mode` 用 `angles`、`shading` 用 `solid`（快），从正面、侧面、顶面、斜 45° 看有没有穿模、悬空、比例不对。
3. 有问题就回到第 3 步改，最多自查 3 轮；把最后一张多角度截图给用户看并说明改了什么。

### 第 5 步：需要现成模型时（可选）
- 先看第 1 步记下的生成器/素材库是否开启；没开就说明需要用户在 Blender 侧栏里开启并填自己的密钥，**不要替用户申请或填写密钥**。
- 生成模型：`generate_3d`，`prompt` 用简短英文描述**一个**物体；国内用户优先 `provider` 选 `hunyuan3d`（腾讯混元3D，用户自己的腾讯云账号，按量计费）。每次调用都可能花钱，调用前先告诉用户；同样的东西生成一次后复制即可。
- 素材库：`search_assets` 再 `import_asset`。Poly Haven 资源是 CC0；Sketchfab 和 Poly Pizza 的授权各不相同，导入后要把授权和作者告诉用户。海外素材库下载慢或失败时，改用基础几何体搭建，不要反复重试。

### 第 6 步：保存和出图
1. 保存工程：`bpy.ops.wm.save_as_mainfile(filepath='用户指定或桌面上的路径.blend')`。
2. 渲染一张图（兼容 Blender 4.x 和 5.x 的引擎名）：
   ```python
   import bpy
   s = bpy.context.scene
   ids = [e.identifier for e in bpy.types.RenderSettings.bl_rna.properties['engine'].enum_items]
   s.render.engine = 'BLENDER_EEVEE' if 'BLENDER_EEVEE' in ids else 'BLENDER_EEVEE_NEXT'
   s.render.resolution_x, s.render.resolution_y = 1080, 1440   # 竖版 3:4；横版可用 1920x1080
   s.render.filepath = '输出路径.png'
   bpy.ops.render.render(write_still=True)
   ```
3. 用 `look`（`mode` 为 `camera`）确认构图，再把 .blend 和 .png 的完整路径告诉用户。

## 交付格式
- 一句话说明做了什么；
- 物体清单（中文名 + 尺寸）；
- 最后一张检查截图；
- 文件路径（.blend、.png）；
- 如果用了生成模型或素材：来源、授权、可能产生的费用。

## 边界
- `execute_blender_code` 可以执行任意 Python，只写和建模、渲染有关的代码，不读写与任务无关的文件，不联网下载脚本。
- 不保证复杂角色、写实人物一次做好；遇到这类需求，建议用户用生成模型（第 5 步）或现成素材。
- 遥测：MCP for Blender 的内容采集默认关闭，若用户问起隐私，告诉用户可设 `DISABLE_TELEMETRY=true` 彻底关闭。
