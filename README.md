# FineUI Agent Skills

FineUI 官方 Agent Skills 集合，为 **Claude Code、GitHub Copilot、Cursor、OpenCode、Codex** 等 AI 编程助手提供准确的 FineUI API、开发模式和代码示例，减少错误 API、版本混用和技术栈混用。

> 这些技能供 AI 编程助手使用，不是应用程序的运行时依赖。仓库只包含 FineUI 的公开用法、文档和示例知识，不含 FineUI 源码或内部实现。

## FineUI 部署栈与开发模式

FineUI 有 **4 部署栈**，其中 **Core 含 3 种开发模式**。本技能集为每种写法给出可运行的最小代码：

| 部署栈 | 开发模式 | 前台写法 |
|--------|---------|---------|
| **F.js** | —（纯 JavaScript / jQuery） | `F.create({ type: 'Grid', ... })` |
| **Pro** | WebForms | `<f:Grid runat="server">` + 后置代码 |
| **Core** | **MVC** | `Html.F().Grid()...`（**Fluent API**） |
| **Core** | **RazorForms**（Core 推荐） | `<f:Grid>` **TagHelper**（数据在后台 `Page_Load` 绑定） |
| **Core** | **RazorPages** | `<f:Grid>` **TagHelper**（数据在标签内联 `DataSource`） |
| **Java** | Spring Boot | `<f:grid>` **Thymeleaf 方言标签**（kebab-case，数据在页面类 `Page_Load` 绑定） |

> RazorForms 与 RazorPages 共用 TagHelper 标签，主要区别在数据初始化和事件处理，详见 `fineui-grid`。
>
> FineUIJava 与 Core RazorForms 都采用标签式有状态服务端组件。FineUIJava 基于 Spring Boot 和 Thymeleaf 方言，标签与属性使用 kebab-case，页面类使用 Java。

## 前置要求

- **FineUI v16.0**。技能中的客户端事件、回发语义和 RawHtml 安全模型均以此版本为准。

## 安装

技能采用开放的 **Agent Skills（`SKILL.md`）格式**，可供多种 AI 编程助手使用。

### 方式一：CLI（推荐）

```bash
npx skills add sanshi/fineui-skills
```

这是**项目级安装**。请先在终端进入需要使用这些技能的项目根目录，再执行安装命令。例如，要为 `D:\FineUI` 项目安装：

```powershell
cd D:\FineUI
npx skills add sanshi/fineui-skills
```

CLI 会检测可用的 Agent，并在需要时提示你选择技能、目标 Agent 和安装方式。安装结果只对当前项目生效。

如果不想在每个项目中分别安装，可以在任意目录执行全局安装：

```bash
npx skills add sanshi/fineui-skills -g
```

选择多个 Agent 时，CLI 默认推荐通过符号链接共享一份规范副本。只有选择复制方式或使用 `--copy` 时，才会为各 Agent 创建独立副本。

### 方式二：手动复制

也可以把 `skills/` 下需要的技能文件夹复制到 Agent 的技能目录。下表列出 `skills` CLI 当前使用的主要路径：

| Agent | 项目级目录 | 全局目录 |
|-------|-----------|----------|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |
| OpenCode | `.agents/skills/` | `~/.config/opencode/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |

## 更新

```bash
npx skills update
```

技能跟随 FineUI 版本发布；升级 FineUI 后请同步更新技能。

## 已含技能

| 技能 | 说明 |
|------|------|
| `fineui-foundation` | 基础：写法判定、`F.create`、PageManager、页面骨架、客户端事件与回发、RawHtml 安全模型、命名约定 |
| `fineui-grid` | 表格：列、数据、编辑、选择、分页、排序、合计行、过滤、多表头、分组、树表格、列锁定、合并、行扩展、事件、拖拽排序和大数据 |
| `fineui-form` | 表单：容器、常用字段、高级字段、字段校验、整表校验和取值 |
| `fineui-window` | 窗口与消息框：Window、iframe、关闭回传、Alert、Confirm 和 Notify |
| `fineui-tree` | 树：节点、图标、后台建树、数据绑定、复选框、级联、节点事件和异步加载 |
| `fineui-panel` | 容器：Panel、工具栏、折叠、Tools、TabStrip 和 Accordion |
| `fineui-layout` | 布局：Fit、Region、HBox、VBox、Block、Column、Anchor 和视口自适应 |
| `fineui-buttons-toolbar` | 按钮与菜单：语义色、图标、徽标、点击事件、确认按钮、LinkButton、ButtonGroup 和下拉菜单 |
| `fineui-theming` | 主题：CSS Variables、全局设置、运行时切换和自定义主题生成 |
| `fineui-upgrade` | 版本升级：识别 v10 以来的破坏性变更并生成迁移清单 |

## 用法

装好后，在 AI 编程助手里正常提需求即可，例如：

- “用 FineUICore 的 TagHelper 写一个带复选框多选、服务端读取选中行的员工表格”
- “把这个 Grid 加一个日期格式化列，显示成 yyyy/MM/dd”
- “F.js 里怎么给 Grid 列写自定义渲染函数”

AI 会自动命中相关技能，按 FineUI 官方写法生成对应端的代码。

## 反馈

发现技能内容有误或缺失，欢迎提交 [Issue](https://github.com/sanshi/fineui-skills/issues)。

## License

[MIT](./LICENSE)
