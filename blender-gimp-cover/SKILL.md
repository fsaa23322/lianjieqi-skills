---
name: blender-gimp-cover
description: 用 3D 渲染图做封面、海报和商品展示图的技能，全程开源免费软件。用户说「做封面」「做小红书封面」「做B站封面」「做抖音封面」「3D封面」「3D海报」「产品展示图」「商品主图」「渲染图加字」「3D出图加标题」时使用。先用 MCP for Blender 在本机 Blender 里搭好或打开 3D 场景，按目标平台尺寸直接渲染，再用 GIMP MCP 在 GIMP 里加标题文字、阴影并导出 PNG，一次可出多个平台尺寸。由智能体连接器（lianjieqi.ai）编写并实测。
license: MIT
metadata:
  author: 智能体连接器（lianjieqi.ai）
  version: "1.0.0"
  homepage: https://lianjieqi.ai/connectors/blender-gimp-cover/
---

# 3D 封面图一条龙（Blender + GIMP）

一句话需求 → 3D 渲染 → 加中文标题 → 按平台尺寸导出。两个软件都是开源免费的，不需要任何付费账号。

## 依赖（先检查，缺了就告诉用户怎么装）

| 依赖 | 作用 | 是否必需 | 国内网络能否直接用 |
| --- | --- | --- | --- |
| Blender 3.0 及以上 + MCP for Blender（`uvx mcp-for-blender`） | 搭场景、渲染 | 必需 | 可直接用（本机运行；首次安装需从 PyPI 下载） |
| GIMP 3.2 及以上 + GIMP MCP（`uvx gimp-mcp-plugin@latest`） | 加字、阴影、导出 | 必需 | 可直接用（本机运行；首次安装需从 PyPI 下载） |
| 本站技能 blender-cn-starter | 不会建模时按它的步骤搭场景 | 可选 | 同上 |

安装说明：https://lianjieqi.ai/connectors/blender-mcp/ 和 https://lianjieqi.ai/connectors/gimp-mcp/ 。两个 MCP 同时连接时互不冲突（Blender 默认端口 9876，GIMP 默认 9877）。

注意：两者都是**本机连接器**。智能体必须能访问用户这台电脑；任务跑在云端容器里时连不上，要直接告诉用户。

## 常用尺寸（像素）

| 用途 | 宽 × 高 |
| --- | --- |
| 小红书笔记封面（3:4 竖版） | 1080 × 1440 |
| 抖音 / 视频号竖版封面（9:16） | 1080 × 1920 |
| B 站 / 横版视频封面（16:9） | 1920 × 1080 |
| 电商商品主图（1:1） | 800 × 800 或 1080 × 1080 |

平台规格会调整，用户给了具体尺寸就以用户为准。

## 工作流程

### 第 1 步：问清三件事
封面上放什么（主体）、标题文字（几个字）、要哪几个平台尺寸。没说就默认小红书 3:4。

### 第 2 步：准备 3D 场景（Blender）
1. `get_addon_status` 确认连上 Blender。
2. 用户有现成 .blend：用 `execute_blender_code` 执行 `bpy.ops.wm.open_mainfile(filepath='...')`。没有：按 blender-cn-starter 的第 2～4 步搭一个简单场景（主体 + 地面 + 灯光 + 相机）。
3. **给标题留白**：调整相机，让主体偏下或偏一侧，画面上方或一侧留出约 1/4 的空间放字。用 `look`（`mode` 为 `camera`、`shading` 为 `material`）检查构图。

### 第 3 步：按尺寸渲染（每个尺寸一张）
```python
import bpy
s = bpy.context.scene
ids = [e.identifier for e in bpy.types.RenderSettings.bl_rna.properties['engine'].enum_items]
s.render.engine = 'BLENDER_EEVEE' if 'BLENDER_EEVEE' in ids else 'BLENDER_EEVEE_NEXT'
s.render.resolution_x, s.render.resolution_y, s.render.resolution_percentage = 1080, 1440, 100
s.render.filepath = '输出目录/cover-3d-1080x1440.png'
bpy.ops.render.render(write_still=True)
print(s.render.filepath)
```
换尺寸后构图会变：每换一次比例都用 `look`（`mode` 为 `camera`）看一下，必要时挪相机再渲染。直接按目标尺寸渲染，比渲一张大图再裁切更清晰。

### 第 4 步：加标题（GIMP）
1. `check_server` 确认连上 GIMP。
2. `open_image` 打开第 3 步的 PNG，`list_images` 确认它的 `index`（最新打开的通常是 0），后面都带上 `image_index`。
3. `add_text` 写标题：中文用 `Sans-serif Bold` 一类粗体（先 `list_fonts` 看本机有哪些字体），字号约为画面宽度的 8%～10%（1080 宽约 90～110 像素），白字或亮色字。返回的 `text_width` 超过画面宽度的 90% 时，用 `edit_text` 调小 `size`。
4. 字和背景对比不够时，`apply_drop_shadow` 加阴影；或先用 `fill_rectangle` 在字后面画一块纯色色块再放字（色块颜色可用 `get_pixel_color` 取画面里的颜色）。
5. `get_state_snapshot` 检查：字有没有压住主体、有没有出界、有没有方块字（缺字体）。

### 第 5 步：导出
- `export_image` 导出 `cover-平台-宽x高.png`；要保留图层继续改，就再 `save_xcf`。
- 多个平台尺寸：对每张渲染图重复第 4 步（不要把竖版图硬拉成横版）。

## 交付格式
- 每个尺寸的 PNG 路径（以及 .blend / .xcf 路径）；
- 最终效果截图；
- 用到的字体名称（商用发布前提醒用户确认字体授权）。

## 边界
- 只操作用户指定的文件和输出目录；`execute_blender_code` 和 GIMP 的 `call_api` 都能执行任意 Python，只写与本任务相关的代码。
- 不替用户上传或发布到任何平台。
