# hermes-bot-video（视频剪辑与剪映草稿 Bot）

这是一个打包为 **Hermes Profile Distribution** 的 AI 视频剪辑与剪映草稿生成助手。它结合本机 `ffmpeg`（切片/转码）、`Whisper`（语音识别生成 SRT 字幕）以及 **VectCutAPI MCP**，能够根据你的素材和自然语言要求自动编排生成**剪映专业版（Jianying Pro）/ CapCut 本地草稿**，由你在剪映中微调并导出最终成片。

## 核心能力与工作流

1. **FFmpeg 切片与转码**：通过终端工具调用本机 `ffmpeg` / `ffprobe`，探测素材时长与分辨率，执行片段截取、格式转换、音轨提取与静音裁剪。
2. **Whisper 语音转字幕**：调用本机 Whisper 工具将人声转写为带时间轴的 `.srt` 字幕文件，并自动挂载到剪映字幕轨道。
3. **生成剪映草稿（人机协同）**：通过 `vectcut` MCP 服务创建草稿、铺设多轨视频/音频/图片、添加 SRT 字幕、设置转场与关键帧动画。**Bot 只负责生成草稿，不直接导出成片或发布**，为你保留 100% 的剪映时间线微调自由度。

## 剪映草稿 MCP 选型说明（为什么选 VectCutAPI 而不是 jianying-draft-mcp）

在构建本 Bot 时，我们对 GitHub 上的 `VectCutAPI` 与 `jianying-draft-mcp` 进行了对比调研，最终选择 **`sun-guannan/VectCutAPI`** 作为内置配置的 MCP 服务端，理由如下：

| 对比维度 | `sun-guannan/VectCutAPI`（已选用） | `jianying-draft-mcp`（未选用） |
|---|---|---|
| **社区成熟度与维护** | **2,350+ Stars / 490+ Forks**，社区极其活跃，持续更新维护 | GitHub 上同名仓库（如 `Angelo236/jianying-draft-mcp`、`sdjvhbuw/jianying-draft-mcp`）均为 **0 Star** 的早期个人仓库，且均未发布到 PyPI/npm |
| **剪映版本兼容性** | 支持通过 `draft_profile` 切换 `jianying_legacy`、`capcut_legacy` 及最新 **`jianying_pro_10`**（剪映专业版 10.x Windows `draft_content.json` 结构） | `Angelo236` 版仅验证 macOS 且 Windows 兼容性待验证；`sdjvhbuw` 版仅针对旧版剪映 4.3.1 实测 |
| **功能完整度（与本 Bot 契合度）** | 原生提供 `mcp_server.py`（stdio 协议），内置 11 个核心工具，完整覆盖 **创建草稿、视频轨、音频轨、图片、文本花字、SRT 字幕直接导入（`add_subtitle`）、特效、贴纸、关键帧动画与保存部署**，与 Whisper 产出的 `.srt` 完美衔接 | `Angelo236` 版（v0.1.1）仅 6 个工具，不支持音频/BGM、变速、转场、特效，也无直接导入 `.srt` 文件的工具；`sdjvhbuw` 版虽有 47 个底层字段工具，但缺乏成熟社区验证 |
| **底层依赖** | 基于成熟的 `pyJianYingDraft` 封装，带 stdout 隔离保护，MCP JSON-RPC 通信稳定 | 各自独立封装，生态割裂 |

## 本机前置依赖

在使用本 Bot 前，请确保本机已准备好以下环境：

1. **FFmpeg**（必须，用于视频切片、转码与时长探测）：
   ```bash
   ffmpeg -version
   ffprobe -version
   ```
2. **Python 3.10+**（必须，用于运行 VectCutAPI MCP 与 Whisper）：
   ```bash
   python --version
   ```
3. **VectCutAPI 仓库与依赖**（必须，提供剪映草稿 MCP 服务）：
   ```bash
   git clone https://github.com/sun-guannan/VectCutAPI.git
   cd VectCutAPI
   pip install -r requirements.txt -r requirements-mcp.txt
   ```
4. **Whisper 语音识别工具**（推荐，用于自动生成 `.srt` 字幕）：
   ```bash
   pip install -U openai-whisper
   # 或安装更快的 faster-whisper
   pip install -U faster-whisper
   ```
5. **剪映专业版（Jianying Pro）或 CapCut**（用于打开生成的草稿进行人工微调与导出）。

## 安装与配置命令

### 1. 安装 Bot

```bash
hermes profile install https://github.com/zzyxiangnian-star/hermes-bot-video --yes --alias
```

### 2. 配置 `VECTCUT_API_DIR` 环境变量

安装后，Hermes 会在 `~/.hermes/profiles/video/.env.EXAMPLE` 生成环境变量模板。将其复制为 `.env` 并填入你本机克隆 `VectCutAPI` 的绝对路径：

```bash
cp ~/.hermes/profiles/video/.env.EXAMPLE ~/.hermes/profiles/video/.env
```

编辑 `~/.hermes/profiles/video/.env`：

```ini
# Windows 示例（建议使用正斜杠避免转义问题）：
VECTCUT_API_DIR=F:/tools/VectCutAPI

# macOS / Linux 示例：
# VECTCUT_API_DIR=/Users/yourname/tools/VectCutAPI
```

### 3. 启动对话

```bash
hermes -p video chat
# 若使用了 --alias，也可直接运行：
video chat
```

后续更新本 Bot 配方：

```bash
hermes profile update video
```

## 推荐搭配的 Hermes 技能（Skills）

本仓库不内置自定义 Skill。你可以根据创作需求，配合使用 Hermes 内置技能或从技能市场（`hermes skills`）安装相关技能：

- `media/youtube-content`（内置）：提取视频字幕与章节摘要，辅助提炼短视频高光切片脚本。
- `creative/manim-video`（内置）：生成数学或算法演示动画片段，作为 B-roll 素材导入剪映草稿。
- `creative/ascii-video`（内置）：将视频/音频转换为彩色 ASCII 风格片段，丰富视觉特效素材。
