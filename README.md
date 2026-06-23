# 猫猫的记忆管理工具 · Maomaomz Memory Management Tool

![version](https://img.shields.io/badge/version-2.0.228-blue)
![platform](https://img.shields.io/badge/platform-SillyTavern-purple)
![license](https://img.shields.io/badge/license-AFPL%20v9-lightgrey)
![i18n](https://img.shields.io/badge/i18n-中文%20%7C%20English-green)

面向 [SillyTavern](https://github.com/SillyTavern/SillyTavern)（酒馆）的第三方扩展，把冗长的聊天记录**自动/手动总结**为结构化记忆并写入**世界书（World Book）**，在长程对话中保持上下文连贯、降低 Token 消耗。围绕这一核心，还内置反八股、NPC 生成、关系分析、开场白生成、状态栏生成等一整套创作工具。

A third-party [SillyTavern](https://github.com/SillyTavern/SillyTavern) extension that summarizes long chat histories into structured memory stored in the **World Book**, keeping context coherent and cutting token cost on long sessions — plus a full toolbox of authoring helpers.

> [!IMPORTANT]
> 禁止商业化 · 禁止倒卖 — Non-commercial · No resale.

## 功能 · Features

### 记忆管理 · Memory management
- **自动总结**：按楼层间隔自动触发，可视化显示已总结楼层与下次起始楼层，失败自动重试。
- **历史总结管理**：结构化编辑、Markdown 复制、导出全部、清空、一键补世界书候选。
- **世界书集成**：总结结果可一键创建/绑定世界书条目，并自动隐藏已总结楼层以节省上下文。
- **可视化总结编辑器**：结构化填写剧情、角色、关系、伏笔与世界书候选。

### 创作工具箱 · Authoring toolbox
- **反八股工具** — 清理角色卡 / 世界书 / 开场白中的套话八股。
- **NPC 快速生成** — 快速产出配角设定。
- **世界书条目生成** — AI 生成世界书条目。
- **人物关系分析 / 关系图谱** — 分析并可视化角色关系。
- **开场白生成器** — AI 生成角色开场白，支持 Token 控制。
- **Token 压缩 / 统计** — 压缩文本降低消耗，统计用量。
- **状态栏生成器** — 生成翻页状态栏，支持预览、配置持久化与一键插入世界书。
- **前端项目管理器** — 管理前端界面项目，支持导出/导入与存储空间监控。

### 体验 · Experience
- iOS 风格界面，统一系统蓝与分组列表样式（含按钮 / 输入 / 卡片 / 滑动开关组件）。
- 中英文双语界面一键切换。
- 主题自定义：主题色、渐变、背景图片、透明度。
- 任务中心、数据备份（一键导出/导入全部设置、总结历史与偏好 JSON）。

## 快速开始 · Getting Started

### 前置条件 · Prerequisites
- 已安装并运行 [SillyTavern](https://github.com/SillyTavern/SillyTavern)。
- 已安装 **TavernHelper**（酒馆助手）扩展 — 本工具依赖其 API。
- 一个可用的 AI API（可单独配置，或直接复用酒馆已配置的 API）。

### 安装 · Installation

**通过扩展安装器（推荐）** — 打开 SillyTavern → `Extensions` → `Install extension`，填入仓库地址：

```
https://github.com/mzrodyu/maomaomz-release
```

安装后刷新页面，面板会自动出现（可在偏好设置中关闭自动弹出）。

**手动安装** — 将本仓库放入第三方扩展目录后重启 / 刷新 SillyTavern：

```
SillyTavern/data/<user>/extensions/maomaomz-release
```

> [!TIP]
> `manifest.json` 中 `auto_update: true`，扩展会从发布仓库自动检查更新。

### 配置 · Configuration

进入面板 **设置（Settings）** 标签页：

- **使用酒馆 API** — 勾选后直接复用酒馆主界面配置的 API，完美绕过 CORS（推荐）。
- **独立 API** — 填写提供商、端点、API Key、模型名称，可调 `max_tokens` / `temperature` / `top_p` / 惩罚项，并支持「测试连接」「拉取模型列表」。
- **自动总结** — 设置楼层间隔、总结后自动绑定世界书、自动隐藏已总结楼层、保留最近 N 条消息、总结风格（简洁 / 详细 / 叙事）。

## 架构 · Architecture

这是一个**纯前端、可热插拔**的 SillyTavern 扩展。SillyTavern 在加载时读取 `manifest.json`，按 `loading_order` 注入入口脚本 `dist/index.js`，随后由 Vite 的动态导入按需加载各功能 chunk。

```
SillyTavern (宿主)
   │  读取 manifest.json (loading_order: 150)
   ▼
dist/index.js  ──┬─► vue / pinia / zod / utils vendor   (第三方库分包)
 (扩展入口)       ├─► 添加导航按钮.chunk.js               (在酒馆 UI 注入入口按钮)
                 ├─► 浮动面板.chunk.js                    (主面板 UI：标签页 / 工具 / 设置)
                 └─► zh / en chunk                       (按需加载的语言包)
   │
   ▼
TavernHelper API  ──►  角色卡 · 世界书 · 聊天楼层 · 酒馆内置 API
```

| 类别 · Category | 技术 · Tech |
| --- | --- |
| 前端框架 | Vue 3 |
| 状态管理 | Pinia |
| 数据校验 | Zod |
| 构建工具 | Vite（ES Module + 代码分包 chunk）|
| 宿主平台 | SillyTavern 扩展 + TavernHelper API |
| 国际化 | `i18n/`（`zh.json` / `en.json`）+ 运行时语言包 chunk |
| 样式 | 原生 CSS + iOS 设计系统组件 |

> [!NOTE]
> 本仓库为**发布产物仓库**，源码工程不在此处。技术栈由 `dist/` 中的 vendor 分包推断；版本与变更记录见 [`manifest.json`](manifest.json)。

## 许可证 · License

本项目采用 **Aladdin Free Public License (AFPL) Version 9** — 允许免费使用与分发，禁止任何商业用途与倒卖。完整条款见 [LICENSE](LICENSE)。

---

<sub>作者 · Author：猫猫 (mzrodyu) · 仓库 · Repo：<https://github.com/mzrodyu/maomaomz-release></sub>
