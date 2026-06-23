# 猫猫的记忆管理工具 (Maomaomz Memory Management Tool)

![version](https://img.shields.io/badge/version-2.0.225-blue)
![platform](https://img.shields.io/badge/platform-SillyTavern-purple)
![license](https://img.shields.io/badge/license-AFPL%20v9-lightgrey)
![i18n](https://img.shields.io/badge/i18n-中文%20%7C%20English-green)

> 面向 [SillyTavern](https://github.com/SillyTavern/SillyTavern)（酒馆）的第三方扩展，提供 AI 聊天**记忆管理**与一整套创作辅助工具箱。
>
> A third-party [SillyTavern](https://github.com/SillyTavern/SillyTavern) extension that delivers AI chat **memory management** plus a full toolbox of authoring helpers.

> ⚠️ **使用须知 / Notice：** 禁止商业化 · 禁止倒卖（Non-commercial · No resale）

---

## 项目简介 / Description

「猫猫的记忆管理工具」是一个安装在 SillyTavern 浮动面板中的扩展。它的核心是把冗长的聊天记录**自动/手动总结**成结构化记忆，并写入**世界书（World Book）**，从而在长程对话中保持上下文连贯、降低 Token 消耗。围绕这一核心，扩展还内置了反八股、NPC 生成、关系分析、开场白生成、状态栏生成、前端项目管理等多个创作工具。

界面采用 iOS 风格设计语言，支持**中英文双语**，可自定义主题色、渐变、背景图与透明度。

---

## 技术栈 / Technology Stack

| 类别 | 技术 |
| --- | --- |
| 前端框架 | Vue 3 |
| 状态管理 | Pinia |
| 数据校验 | Zod |
| 构建工具 | Vite（产物为 ES Module + 代码分包 chunk）|
| 宿主平台 | SillyTavern 扩展 + TavernHelper API |
| 国际化 | 自带 `i18n/`（`zh.json` / `en.json`）+ 运行时语言包 chunk |
| 样式 | 原生 CSS（`dist/style.css`），iOS 设计系统组件 |

> 版本信息见 [`manifest.json`](manifest.json)（当前 `2.0.225`）。技术栈由发布产物 `dist/` 中的 vendor 分包推断：`vue-vendor`、`pinia-vendor`、`zod-vendor`、`utils-vendor`。

---

## 项目架构 / Architecture

这是一个**纯前端、可热插拔**的 SillyTavern 扩展。SillyTavern 在加载时读取 `manifest.json`，按 `loading_order` 注入入口脚本 `dist/index.js`，随后由 Vite 的动态导入按需加载各功能 chunk。

```
SillyTavern (宿主)
   │  读取 manifest.json (loading_order: 150)
   ▼
dist/index.js  ──┬─► vue-vendor / pinia-vendor / zod-vendor / utils-vendor  (第三方库分包)
 (扩展入口)       │
                 ├─► 添加导航按钮.chunk.js     (在酒馆 UI 注入入口按钮)
                 ├─► 浮动面板.chunk.js          (主面板 UI：标签页 / 工具 / 设置)
                 └─► zh / en chunk             (按需加载的语言包)
   │
   ▼
TavernHelper API  ──►  角色卡 · 世界书 · 聊天楼层 · 酒馆内置 API
```

- **入口注入**：`添加导航按钮` 在酒馆界面挂载触发按钮与右上角猫猫头快捷图标。
- **主面板**：`浮动面板` 承载所有标签页，是体量最大的 chunk。
- **宿主交互**：通过 `TavernHelper` / `SillyTavern.getContext()` 读写角色卡、世界书、聊天楼层，并可直接复用酒馆主界面配置的 API（绕过 CORS）。

---

## 快速开始 / Getting Started

### 前置条件 / Prerequisites

- 已安装并运行 [SillyTavern](https://github.com/SillyTavern/SillyTavern)
- 已安装 **TavernHelper**（酒馆助手）扩展——本工具依赖其 API
- 一个可用的 AI API（可单独配置，或直接复用酒馆已配置的 API）

### 安装 / Installation

**方式一：通过扩展安装器（推荐）**

1. 打开 SillyTavern → `Extensions（扩展）` → `Install extension`
2. 填入仓库地址：
   ```
   https://github.com/mzrodyu/maomaomz-release
   ```
3. 安装后刷新页面，面板会自动出现（可在偏好设置中关闭自动弹出）。

**方式二：手动安装**

1. 将本仓库放入 SillyTavern 的第三方扩展目录：
   ```
   SillyTavern/data/<user>/extensions/maomaomz-release
   ```
2. 重启 / 刷新 SillyTavern。

> `manifest.json` 中 `auto_update: true`，扩展会从发布仓库自动检查更新。

### 配置 / Configuration

进入面板 **设置（Settings）** 标签页：

- **使用酒馆 API**：勾选后直接复用酒馆主界面配置的 API，完美绕过 CORS（推荐）。
- **独立 API**：填写 API 提供商、端点、API Key、模型名称，可调 `max_tokens` / `temperature` / `top_p` / 惩罚项，并「测试连接」「拉取模型列表」。
- **自动总结**：开启后按楼层间隔自动总结，可设置总结后自动绑定世界书、自动隐藏已总结楼层、保留最近 N 条消息、总结风格（简洁 / 详细 / 叙事）。

---

## 项目结构 / Project Structure

```
maomaomz-release/
├── manifest.json          # 扩展清单：名称、版本、入口、changelog、i18n、自动更新
├── dist/                  # 构建产物（运行时实际加载）
│   ├── index.js           #   扩展入口
│   ├── style.css          #   样式
│   ├── *-vendor.*.chunk.js#   第三方库分包（vue / pinia / zod / utils）
│   ├── 浮动面板.*.chunk.js #   主面板 UI
│   ├── 添加导航按钮.*.chunk.js # 入口按钮注入
│   └── zh / en .chunk.js  #   运行时语言包
├── i18n/                  # 国际化文案
│   ├── zh.json            #   中文
│   └── en.json            #   English
├── LICENSE                # Aladdin Free Public License (AFPL) v9
└── README.md
```

> 本仓库为**发布（release）仓库**，包含的是已构建产物 `dist/`，而非源码工程。

---

## 核心功能 / Key Features

### 记忆与总结
- **手动总结**：指定起止楼层生成总结，支持「大总结」聚合。
- **自动总结**：按楼层间隔自动触发，可视化显示已总结楼层与下次起始楼层，含失败自动重试。
- **历史总结管理**：结构化编辑、Markdown 复制、导出全部、清空、一键补世界书候选。
- **世界书集成**：总结结果可一键创建/绑定世界书条目，并自动隐藏已总结楼层以节省上下文。
- **可视化总结编辑器**：结构化填写剧情、角色、关系、伏笔与世界书候选。

### 创作工具箱
- **反八股工具**：清理角色卡 / 世界书 / 开场白中的套话八股。
- **NPC 快速生成**：快速产出配角设定。
- **世界书条目生成**：AI 生成世界书条目。
- **人物关系分析 / 关系图谱**：分析并可视化角色关系。
- **开场白生成器**：AI 生成角色开场白，支持 Token 控制。
- **Token 压缩 / 统计**：压缩文本降低消耗，统计 Token 用量。
- **状态栏生成器**：生成翻页状态栏，支持预览、配置持久化（保存/恢复、导入导出）、一键插入世界书。
- **前端项目管理器**：管理前端界面项目，支持导出/导入与存储空间监控。

### 体验与个性化
- **iOS 风格界面**：统一系统蓝与分组列表样式，含 iOS 设计系统组件（按钮 / 输入 / 卡片 / 滑动开关等）。
- **中英文双语**：界面一键切换。
- **主题自定义**：主题色、渐变模式与角度、背景图片、透明度。
- **任务中心**：右下角显示任务进度与状态。
- **数据备份**：一键导出/导入全部设置、总结历史与偏好（JSON）。

---

## 开发工作流 / Development Workflow

> 本仓库为发布产物仓库，源码工程不在此处。以下为基于产物与提交历史推断的发布约定。

- **版本发布**：采用语义化版本，提交信息格式为 `release: vX.Y.Z`（见 `git log`），并同步更新 `manifest.json` 的 `version` 与 `changelog`。
- **构建**：源码经 Vite 构建为 `dist/` 下的 ES Module 与按需 chunk。
- **自动更新**：发布仓库即更新源，客户端依据 `manifest.json` 自动检查更新。
- **变更记录**：每个版本的简述维护在 `manifest.json` 的 `changelog` 字段中。

---

## 编码规范 / Coding Standards

> 产物中已观测到的约定（供贡献参考）：

- **框架约定**：Vue 3 组件化 + Pinia 集中状态管理。
- **数据校验**：使用 Zod 对配置/数据结构做运行时校验。
- **国际化**：所有面向用户的文案统一走 `i18n/`（`zh.json` / `en.json`），键名按区域前缀分组（如 `pref_*`、`settings_*`、`tools_*`、`summary_*`）。新增文案需中英双语同步。
- **图标**：使用 FontAwesome 图标，避免 emoji。
- **UI 一致性**：复用 iOS 设计系统组件（IosButton / Input / Textarea / Select / Card、ToggleSwitch）。

---

## 测试 / Testing

仓库未包含独立的测试套件。当前质量保障以**手动验证**为主，结合：

- 构建期的 TypeScript / ESLint 检查（见 changelog `2.0.206` 修复记录）。
- 在 SillyTavern 实环境中验证 API 连接、自动总结、世界书绑定等关键链路。

如需贡献自动化测试，建议针对 Zod schema 与纯函数工具（Token 压缩、文本处理）补充单元测试。

---

## 贡献 / Contributing

欢迎反馈与改进建议：

1. 在 [GitHub Issues](https://github.com/mzrodyu/maomaomz-release/issues) 提交问题或建议。
2. 提交改动前，请保持与现有约定一致：Vue 3 + Pinia 组件结构、Zod 校验、`i18n/` 双语文案同步、iOS 组件复用。
3. 涉及用户可见变更时，请同步更新 `manifest.json` 的 `version` 与 `changelog`。

> 由于本仓库为发布产物仓库，源码相关贡献请联系作者 **猫猫 (mzrodyu)**。

---

## 许可证 / License

本项目采用 **Aladdin Free Public License (AFPL) Version 9**。

该许可证**允许免费使用与分发，但禁止任何商业用途与倒卖**，与面板内提示「禁止商业化 | 禁止倒卖」一致。完整条款见 [LICENSE](LICENSE)。

---

<sub>作者 / Author：猫猫 (mzrodyu) · 仓库 / Repo：https://github.com/mzrodyu/maomaomz-release</sub>
