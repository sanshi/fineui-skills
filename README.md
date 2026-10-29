# FineUI Agent Skills

本仓库维护一个对用户可见的 `fineui` 技能及按任务分类的参考资料。使用下文的 `npx skills add` 命令安装；需要更详细的公开 API、示例或主题资料时，可沿 [官网 AI 文档索引](https://fineui.com/llms.txt)逐层读取。

FineUI 官方技能为 **Claude Code、GitHub Copilot、Cursor、OpenCode、Codex** 等 AI 编程助手提供开发模式、代码示例和资料导航，减少错误 API、版本混用和技术栈混用。

也可以使用接入 DeepSeek、GLM、Kimi、MiniMax、MiMo 等模型的编程 Agent；技能读取和工具调用由所用编程客户端及模型接入方式支持。

> 这些技能供 AI 编程助手使用，不是应用程序的运行时依赖。仓库只包含 FineUI 的公开用法、文档和示例知识，不含 FineUI 源码或内部实现。

<!-- fineui-community:start -->
## 加入 FineUI 社区

欢迎[加入 FineUI 社区](https://fineui.com/fans/)，一站式获取社区版、完整示例、空项目、快速入门和 AppBox 等配套资源，及时了解版本更新，交流控件用法与项目实践。**FineUI.Core、FineUI.Pro 与 FineUI.Java 社区版均可永久免费商用**，欢迎一起分享经验、讨论问题。
<!-- fineui-community:end -->

## FineUI 部署栈与开发模式

FineUI 有 **4 部署栈**，其中 **Core 含 3 种开发模式**。技能为每种写法给出可运行的最小代码：

| 部署栈 | 开发模式 | 前台写法 |
|--------|---------|---------|
| **F.js** | —（纯 JavaScript / jQuery） | `F.create({ type: 'Grid', ... })` |
| **Pro** | WebForms | `<f:Grid runat="server">` + 后置代码 |
| **Core** | **MVC** | `Html.F().Grid()...`（**Fluent API**） |
| **Core** | **RazorForms**（Core 推荐） | `<f:Grid>` **TagHelper**（数据在后台 `Page_Load` 绑定） |
| **Core** | **RazorPages** | `<f:Grid>` **TagHelper**（数据在标签内联 `DataSource`） |
| **Java** | Spring Boot | `<f:grid>` **Thymeleaf 方言标签**（kebab-case，数据在页面类 `Page_Load` 绑定） |

> RazorForms 与 RazorPages 共用 TagHelper 标签，主要区别在数据初始化和事件处理，详见[表格分类](skills/fineui/references/grid/overview.md)。
>
> FineUI.Java 与 Core RazorForms 都采用标签式有状态服务端组件。两端首屏用 `Page_Init`、控件绑定后用 `Page_Load`；FineUI.Java 基于 Spring Boot 和 Thymeleaf 方言，标签与属性使用 kebab-case，页面类使用 Java。

Core 三种模式与 Java 的 FineUI 首屏和 UI 回发均使用 JSON。官方 Core 示例、空项目、AppBox 和 QuickStart 显式启用严格 CSP；客户项目仍需自行配置 CSP，并把旧脚本字符串迁到控件动作、结构化命令或具名函数。

## 前置要求

- 当前技能源码的版本以 `plugin.json` 和技能顶部的 `metadata.version` 为准。旧版项目优先使用与项目版本一致的技能、源码和示例；不能把官网当前 API 直接当作旧版已支持的 API。

## 安装技能

官网[快速开始](https://fineui.com/docs/#/AgentSkills/quickstart)介绍安装、模式判定与在线资料的使用。安装后，技能选择适合当前任务的分类，按需读取官网文档。

### 项目级安装

这是**项目级安装**。请先在终端进入需要使用这些技能的项目根目录，再执行安装命令。例如，要为 `D:\YourProject` 项目安装：

```powershell
cd D:\YourProject
npx skills add sanshi/fineui-skills
```

CLI 会检测可用的 Agent，并在需要时提示你选择技能、目标 Agent 和安装方式。安装结果只对当前项目生效。

### 全局安装

如果不想在每个项目中分别安装，可以在任意目录执行全局安装：

```bash
npx skills add sanshi/fineui-skills -g
```

## 仅更新 Skills

```bash
npx skills update
```

技能跟随 FineUI 版本发布；升级 FineUI 后请同步更新技能。官网在线资料只对应当前版本，旧项目应先核对自身依赖与同版示例。

如果项目以前安装过 12 个独立的 `fineui-*` 技能，更新后运行 `npx skills list` 检查安装结果；全局安装用 `npx skills list -g`。若旧技能仍在，运行 `npx skills remove`（全局安装加 `-g`）选择这些旧入口，再安装或保留新的 `fineui` 技能，避免选择器同时出现新旧技能。

## 技能分类

安装时只显示一个 `fineui` 技能。技能内提供以下分类：

| 分类 | 说明 |
|------|------|
| [基础与模式](skills/fineui/references/foundation/overview.md) | 基础：写法判定、`F.create`、PageManager、页面骨架、客户端事件与回发、RawHtml 安全模型、命名约定 |
| [表格](skills/fineui/references/grid/overview.md) | 表格：列、数据、编辑、选择、分页、排序、合计行、过滤、多表头、分组、树表格、列锁定、合并、行扩展、事件、拖拽排序和大数据 |
| [表单](skills/fineui/references/form/overview.md) | 表单与日历：容器、常用字段、高级字段、Calendar、字段校验、整表校验和取值 |
| [数据列表](skills/fineui/references/datalist/overview.md) | 数据列表：静态项、数据绑定、图文内容、单选多选、分组、导航和加载更多 |
| [窗口](skills/fineui/references/window/overview.md) | 窗口与消息框：Window、iframe、关闭回传、Alert、Confirm 和 Notify |
| [树](skills/fineui/references/tree/overview.md) | 树：节点、图标、后台建树、数据绑定、复选框、级联、节点事件和异步加载 |
| [容器](skills/fineui/references/panel/overview.md) | 容器：Panel、ContentPanel、GroupPanel、工具栏、折叠、Tools、TabStrip 和 Accordion |
| [布局](skills/fineui/references/layout/overview.md) | 布局：Fit、Region、HBox、VBox、Block、Column、Anchor 和视口自适应 |
| [按钮](skills/fineui/references/button/overview.md) | 按钮：语义色、图标、徽标、点击事件、确认按钮、LinkButton、ButtonGroup 和下拉菜单 |
| [主题设置](skills/fineui/references/theming/overview.md) | 主题设置：全局默认、PageManager 覆盖和运行时切换 |
| [自定义主题](skills/fineui/references/custom-theme/overview.md) | 自定义主题：配色设计、配置生成、三栈接入、运行截图验证和主题包交付 |
| [版本升级](skills/fineui/references/upgrade/overview.md) | 版本升级：覆盖 v10 至 v16.1.0，按模式核对生命周期、API、空值与类型变化并生成迁移清单 |

## 用法

装好后，在 AI 编程助手里正常提需求即可，例如：

- “用 FineUI.Core 的 TagHelper 写一个带复选框多选、服务端读取选中行的员工表格”
- “把这个 Grid 加一个日期格式化列，显示成 yyyy-MM-dd”
- “F.js 里怎么给 Grid 列写自定义渲染函数”
- “在 FineUI.Pro 示例的 res/themes 中创建一个深梅紫与薄荷绿的主题，运行验证并打包”

AI 会自动命中相关技能，按 FineUI 官方写法生成对应端的代码。

## 反馈

发现技能内容有误或缺失，欢迎提交 [Issue](https://github.com/sanshi/fineui-skills/issues)。

## License

[MIT](./LICENSE)

### Core v16.1.1 兼容说明

[旧事件兼容参考](skills/fineui/references/foundation/references/razorforms-compatibility.md) 说明在 RazorForms 中保留本页 OnPost、Parameter 和 UIHelper 即时更新的边界，并包含对象绑定和混合事件示例。此项是 Core 的兼容能力，不扩展到 Java 或 Pro 的事件合同。
