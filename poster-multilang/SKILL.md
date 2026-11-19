---
name: poster-multilang
description: 把中文海报、宣传图、商品主图、活动图一键做成英文、日文、韩文等多语言版本的技能。用户说「海报翻译」「图片翻译」「把图上的中文换成英文」「做英文版海报」「出海宣传图」「多语言版本」「跨境电商主图翻译」「改图片上的文字」时使用。用开源的 GIMP MCP 读出图里的文字图层、替换文字、自动缩放排版并导出，用 DeepL MCP 翻译；DeepL 用不了时改用智能体自身翻译。最适合带文字图层的 GIMP .xcf 设计稿；纯图片（PNG/JPG）也能处理但效果有限。由智能体连接器（lianjieqi.ai）编写并实测。
license: MIT
metadata:
  author: 智能体连接器（lianjieqi.ai）
  version: "1.0.0"
  homepage: https://lianjieqi.ai/connectors/poster-multilang/
---

# 海报多语言版一键出图

输入一张中文设计稿，输出每种目标语言各一张图（PNG）和一份可继续编辑的 .xcf，原文件不改动。

## 依赖（先检查，缺了就告诉用户怎么装）

| 依赖 | 作用 | 是否必需 | 国内网络能否直接用 |
| --- | --- | --- | --- |
| GIMP 3.2 及以上（本机，开源免费） | 打开设计稿、改字、导出 | 必需 | 可直接用（本机运行） |
| GIMP MCP（开源，`uvx gimp-mcp-plugin@latest`） | 让智能体操作 GIMP | 必需 | 可直接用（本机通信；首次安装需从 PyPI 下载） |
| DeepL MCP（官方托管 `https://mcp.deepl.com/v1/mcp`，DeepL 账号登录） | 高质量翻译，一次最多 10 种语言 | 可选（推荐） | 依赖海外服务，可能不稳定或无法访问 |
| 智能体自身的翻译能力 | DeepL 不可用时的替代 | 兜底 | 随智能体 |

安装说明：GIMP MCP 见 https://lianjieqi.ai/connectors/gimp-mcp/ ，DeepL MCP 见 https://lianjieqi.ai/connectors/deepl/ 。

注意：GIMP MCP 是**本机连接器**，需要用户先打开 GIMP 并在「工具 > MCP > Start MCP Server」启动服务（默认端口 9877）。如果智能体的任务跑在云端容器里，连不上用户本机的 GIMP，要直接告诉用户。

## 工作流程

### 第 1 步：确认连接并打开文件
1. 调用 `check_server`，`connected` 为 true 才继续；否则提示用户启动 GIMP 里的 MCP 服务。
2. 问清：源文件路径、目标语言（默认英文）、要不要保留中文版并排对照。
3. `open_image` 打开源文件，记下返回的 `image_id`；再用 `list_images` 确认它的 `index`（最新打开的通常是 0）。后面的工具调用都带上这个 `image_index`。

### 第 2 步：找出所有文字
用 `call_api` 执行下面这段（列出每个文字图层的名字、内容、位置、宽高）：

```json
{"api_path": "exec", "args": ["pyGObject-console", [
  "import json",
  "img = Gimp.Image.get_by_id(第1步的image_id)",
  "print(json.dumps([{'layer': l.get_name(), 'text': l.get_text(), 'x': l.get_offsets()[1], 'y': l.get_offsets()[2], 'w': l.get_width(), 'h': l.get_height(), 'canvas_w': img.get_width()} for l in img.get_layers() if isinstance(l, Gimp.TextLayer)], ensure_ascii=False))"
]]}
```

（把 `第1步的image_id` 换成真实数字，例如 `Gimp.Image.get_by_id(8)`。）

- **有文字图层**：进入第 3 步。
- **没有文字图层（纯图片、文字已栅格化，或 PSD 导入后文字变成了普通图层）**：用 `get_state_snapshot` 看图，自己识别文字和大致位置，列给用户确认。之后只能「盖住原字再写新字」：`get_pixel_color` 取原字周围的底色，`fill_rectangle` 盖住原字区域，再 `add_text` 写译文。底色是渐变或图案时效果会差，要提前告诉用户，建议改用带图层的源文件。

### 第 3 步：先给文字图层改固定名字
GIMP 的文字图层默认以文字内容命名，改字后名字会跟着变，后面就找不到了。所以先用 `rename_layer` 把它们依次改成 `text-1`、`text-2`……，并记下「编号 → 原文」对照表。

### 第 4 步：翻译
1. 把所有原文整理成一个列表，**一次性**翻译，保证术语前后一致：
   - DeepL 可用：调用 `translate-text`，源语言中文，目标语言可一次传多个（最多 10 个）。免费账号单次 5000 字符以内。
   - DeepL 不可用（连不上、没登录、额度用完）：由智能体自己翻译，并在交付时注明「本次未使用 DeepL」。
2. 广告语要意译、短而有力，不要逐字直译；价格、日期、单位换成目标市场习惯写法（例如「满300减50」→「¥50 off every ¥300」）。品牌名、商品型号、二维码旁的网址不翻译。
3. 把译文对照表给用户过目；用户没时间看时可以直接继续，但交付时附上对照表。

### 第 5 步：替换文字并自动排版（每种语言做一遍）
1. 对每个 `text-N` 调用 `edit_text` 换成译文（只改 `text`，字体字号先不动）。
2. 再跑一次第 2 步的代码，看每个图层的 `x + w` 是否超出 `canvas_w`。超出的，用 `edit_text` 把 `size` 按比例调小（新字号 ≈ 原字号 × 可用宽度 / 当前宽度，留 5% 余量），直到不超出。
3. 原本居中的标题要重新居中：
   ```python
   img = Gimp.Image.get_by_id(第1步的image_id)
   W = img.get_width()
   for l in img.get_layers():
       if isinstance(l, Gimp.TextLayer) and l.get_name() == 'text-1':
           l.set_offsets(int((W - l.get_width()) / 2), l.get_offsets()[2])
   Gimp.displays_flush()
   ```
4. 中文字体通常没有好看的西文字形：用 `list_fonts` 找一款无衬线西文字体（如 Sans-serif Bold）换上；日文、韩文要确认所选字体能显示对应文字，用 `get_state_snapshot` 放大检查有没有方块字。
5. `get_state_snapshot` 看整体效果，必要时用 `region` 放大局部检查。

### 第 6 步：导出
- `save_xcf` 另存为 `原文件名-en.xcf`（日文用 `-ja`，韩文用 `-ko`……），**不要覆盖原文件**。
- `export_image` 导出同名 PNG。需要多尺寸时可用 `export_social_media_kit` 或 `batch_resize`。
- 做下一种语言前，重新 `open_image` 打开**原文件**（会得到新的 `image_id`，记得替换），从第 3 步开始，不要在译文版上再翻译。

## 交付格式
- 每种语言的 PNG 和 XCF 路径；
- 原文 → 译文对照表；
- 用了 DeepL 还是智能体自身翻译；
- 需要人工复核的地方（字号被缩小较多、纯图片盖字、字体缺字等）。

## 边界
- 不处理用户没有使用权的图片（他人商标、版权图）的二次发布问题，只做文字替换；发布前的授权由用户自己确认。
- `call_api` 能执行任意 Python，只用于读取和调整当前图片的文字图层。
- 法律、医疗、金融类宣传语的译文要提醒用户请专业人士复核。
