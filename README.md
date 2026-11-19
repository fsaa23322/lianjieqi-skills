# 智能体连接器（lianjieqi.ai）跨厂牌连接器 Skills

本仓库是 [智能体连接器（lianjieqi.ai）](https://lianjieqi.ai) 自研的跨厂牌连接器（Skill）源码。跨厂牌连接器把不同公司的 MCP、Skill、CLI 串成一个完整流程：哪一家好用就用哪一家，不要求整套都来自同一家厂商。

每个文件夹就是一个可安装的 Skill（文件夹名与技能名相同，内含 `SKILL.md`），符合千问办公、豆包工作、WorkBuddy 的 ZIP 上传格式，也可以直接放到 Claude Code 的 `~/.claude/skills/` 目录。

- 网站：[https://lianjieqi.ai](https://lianjieqi.ai)
- 跨厂牌连接器专页：[https://lianjieqi.ai/cross-vendor/](https://lianjieqi.ai/cross-vendor/)
- 投稿：[https://lianjieqi.ai/submit/](https://lianjieqi.ai/submit/)

授权协议：MIT。版权归智能体连接器（lianjieqi.ai）。

## 技能一览

| 技能 | 做什么 | 依赖（国内网络能否直接用） |
| --- | --- | --- |
| [Blender 中文建模助手](https://lianjieqi.ai/connectors/blender-cn-starter/)（`blender-cn-starter`） | 一句中文指挥本机 Blender 建模、上材质、多角度自检并渲染出图 | Blender、MCP for Blender（可直接用）；可选腾讯混元3D（可直接用）、Poly Haven 等素材库（依赖海外服务） |
| [海报多语言版一键出图](https://lianjieqi.ai/connectors/poster-multilang/)（`poster-multilang`） | 把中文海报改成英文、日文、韩文版，文字放不下时自动缩小并重新居中 | GIMP、GIMP MCP（可直接用）；可选 DeepL MCP（依赖海外服务） |
| [3D 封面图一条龙](https://lianjieqi.ai/connectors/blender-gimp-cover/)（`blender-gimp-cover`） | 先用 Blender 按小红书、抖音、B 站尺寸渲染，再用 GIMP 加中文标题导出 | Blender、MCP for Blender、GIMP、GIMP MCP（均可直接用） |

## 安装

先按各技能页面装好依赖的连接器，并在本机打开对应软件、启动 MCP 服务。这些 Skill 都要操作你电脑上的软件，智能体必须能访问本机。

你可以从本仓库复制对应文件夹，也可以在技能页面下载现成的 ZIP。ZIP 内必须有与技能同名的顶层文件夹，并包含 `SKILL.md`。

### 千问办公

1. 在「扩展」→「连接器」→ 右上角「+ 添加」中，按技能页面给出的配置添加所需 MCP。
2. 「扩展」→「技能」→「+ 添加」→「上传技能」，选择该技能的 ZIP，等待解析安装。
3. 打开本机软件并启动 MCP 服务后，新建对话直接用中文描述需求，或输入 `/` 选择技能。

### 豆包工作

1. 在「插件·技能·伙伴」→「插件」→「+ 添加」→「新建自定义连接器」中添加所需 MCP（本机软件一般选 STDIO）。
2. 切换到「技能」→「+ 添加」→「上传技能」，上传 ZIP。
3. 启动本机 MCP 服务后，在任务里输入 `/` 搜索技能名，或直接描述需求。

### WorkBuddy

1. 「专家·技能·连接器」→「技能」→「添加技能」→「上传技能」，选择 ZIP，等待安全检测完成。
2. 在「连接器」页签用「自定义连接器」添加所需 MCP。
3. 输入 `@` 加技能名调用，或直接描述需求。

注意：本站此前测试中，WorkBuddy 的任务运行在云端容器里，访问不到你电脑上的 Blender、GIMP 等本机软件。如果连接失败，请改用千问办公或豆包工作的桌面端。

### Claude Code

1. 用 `claude mcp add ...` 添加所需 MCP（命令见各技能页面），并在对应软件里启动服务。
2. 把技能文件夹放到 `~/.claude/skills/` 下，例如 `~/.claude/skills/blender-cn-starter/SKILL.md`。
3. 重启 Claude Code，直接用中文描述需求即可。

## 实测与未测说明

本站在 Linux 上用真实软件跑通了这 3 个 Skill 的核心流程，细节写在各技能页面。以下环节尚未实测，使用时请留意：

- 在 WorkBuddy、千问办公、豆包工作里的实际触发效果（安装步骤已按各家官方文档整理）。
- 需要付费账号的步骤，例如腾讯混元3D、DeepL。
- macOS / Windows 上的安装过程和字体名称（可能与测试环境不同）。
- Poly Haven、Sketchfab 等海外素材库；海报技能的日文、韩文字体显示。

## 投稿与改进

欢迎提交好用的跨厂牌连接器。请走 [投稿页](https://lianjieqi.ai/submit/)：标准是真正能解决问题，依赖的连接器真实存在且可以安装。本仓库也欢迎 Issue 和 Pull Request。

使用前请先阅读对应的 `SKILL.md`。这些 Skill 会通过 MCP 在本机软件里执行与任务相关的脚本，请只从本仓库或 [智能体连接器（lianjieqi.ai）](https://lianjieqi.ai) 安装。
