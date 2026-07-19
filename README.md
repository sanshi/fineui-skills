# FineUI Agent Skills

官方 AI 技能（Agent Skills）集合，让 **Claude Code / GitHub Copilot / Cursor / OpenCode / Codex** 等 AI 编程助手**准确生成使用 FineUI 的代码**——用对 API、用对版本、不瞎编、不混用竞品，并覆盖 FineUI 的各种写法（F.js、Pro，以及 Core 的 MVC / RazorForms / RazorPages）。

> **这些技能是给 AI 编程助手用的，不是给人直接阅读的运行时库。** 内容全部是「如何使用 FineUI」的公开知识（等同官网文档/示例），**不含 FineUI 源码或内部实现**。

## FineUI 部署栈与开发模式

FineUI 有 **3 部署栈**，其中 **Core 含 3 种开发模式**。本技能集为每种写法给出可运行的最小代码：

| 部署栈 | 开发模式 | 前台写法 |
|--------|---------|---------|
| **F.js** | —（纯 JavaScript / jQuery） | `F.create({ type: 'Grid', ... })` |
| **Pro** | WebForms | `<f:Grid runat="server">` + 后置代码 |
| **Core** | **MVC** | `Html.F().Grid()...`（**Fluent API**） |
| **Core** | **RazorForms**（Core 推荐） | `<f:Grid>` **TagHelper**（数据在后台 `Page_Load` 绑定） |
| **Core** | **RazorPages** | `<f:Grid>` **TagHelper**（数据在标签内联 `DataSource`） |

> RazorForms 与 RazorPages 共用 TagHelper 标签，差异在数据初始化与事件（见 `fineui-grid` 技能）。

## 前置要求

- **FineUI v15.2+**（ESM + ES2022 class 架构，RawHtml 安全模型）。旧版本请使用与之匹配的技能 tag。

## 安装

技能遵循开放的 **Agent Skills（SKILL.md）标准**，一份内容可用于 75+ 个 agent。三种安装方式：

### 方式一：CLI（推荐）

```bash
# GitHub（主仓）
npx skills add fineui/fineui-skills

# 指定 agent（自动拷到对应目录）
npx skills add fineui/fineui-skills -a claude-code -a cursor -a opencode -a codex

# 国内镜像 Gitee（用完整 git URL）
npx skills add https://gitee.com/fineui/fineui-skills.git
```

> `npx skills add owner/repo` 简写仅 GitHub 支持；Gitee 用完整 `.git` URL。

### 方式二：手动复制

把 `skills/` 下需要的技能文件夹拷到你所用 agent 的技能目录：

| Agent | 项目级目录 | 全局目录 |
|-------|-----------|----------|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| GitHub Copilot | `.github/skills/` | `~/.copilot/skills/` |
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| OpenCode | `.opencode/skills/`（也读 `.claude/skills/`） | `~/.config/opencode/skills/` |
| Codex | `.codex/skills/` | — |

### 方式三：Claude Code 插件市场

> 规划中（后续补 `.claude-plugin/marketplace.json`）。

## 更新

```bash
npx skills update
```

技能跟随 FineUI 版本发布；升级 FineUI 后请同步更新技能。

## 已含技能

| 技能 | 说明 | 状态 |
|------|------|------|
| `fineui-foundation` | 地基：写法判定 / `F.create` / PageManager / 页面骨架 / RawHtml 安全模型 / 命名约定 | ✅ v0.1 |
| `fineui-grid` | 表格（Grid）：列配置、数据加载、编辑、选择、分页、工具栏等 | ✅ v0.1 |
| `fineui-form` | 表单：Form/SimpleForm 容器、字段、多列布局、字段/整表校验、读值 | ✅ v0.1 |
| `fineui-window` | 窗口与消息框：Window（内联/iframe）、开关、closeArgument 回传、Alert/Confirm/Notify | ✅ v0.1 |
| `fineui-upgrade` | 版本升级（v10+）：识别破坏性变更、生成迁移清单（源自 release_history） | ✅ v0.1 |

## 用法

装好后，在 AI 编程助手里正常提需求即可，例如：

- “用 FineUICore 的 TagHelper 写一个带复选框多选、服务端读取选中行的员工表格”
- “把这个 Grid 加一个日期格式化列，显示成 yyyy/MM/dd”
- “F.js 里怎么给 Grid 列写自定义渲染函数”

AI 会自动命中相关技能，按 FineUI 官方写法生成对应端的代码。

## 反馈

发现技能内容有误或缺失，欢迎提 issue。

## License

[MIT](./LICENSE)
