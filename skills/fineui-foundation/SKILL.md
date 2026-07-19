---
name: fineui-foundation
description: >
  FineUI 地基技能：搭建 FineUI 页面的通用基础，覆盖 F.js（JavaScript）、Pro（WebForms），
  以及 FineUICore 的三种开发模式 MVC（Fluent API）/ RazorForms（TagHelper）/ RazorPages（TagHelper）。
  用于：判定项目属于哪种写法、F.create 工厂、PageManager、页面骨架与布局（Region/ViewPort）、
  可信 HTML（RawHtml）安全模型、属性命名约定、全局配置项。**做任何 FineUI 页面前先看本技能。**
  Trigger phrases（触发词）: "FineUI", "F.create", "PageManager", "FineUIPro", "FineUICore",
  "Fluent API", "TagHelper", "RazorForms", "RazorPages", "RawHtml", "F.rawHtml", "TextRawHtml",
  "FineUI 布局", "Region 布局", "ViewPort", "页面骨架", "FineUI 怎么用".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 地基技能（Foundation）

搭建任何 FineUI 页面的通用基础。**先用本技能判定“哪种写法”、搭好页面骨架、掌握可信 HTML；再进具体组件技能（如 `fineui-grid`）。**

## 术语：3 部署栈 + Core 3 种开发模式

| 部署栈 | 开发模式 | 前台写法 |
|--------|---------|---------|
| **F.js** | —（纯 JavaScript） | `F.create({ type:'Xxx', ... })` |
| **Pro** | WebForms | `<f:Xxx runat="server">` |
| **Core** | **MVC**（经典 MVC） | **Fluent API** `Html.F().Xxx()...` |
| **Core** | **RazorForms**（Core 推荐） | **TagHelper** `<f:Xxx>` |
| **Core** | **RazorPages** | **TagHelper** `<f:Xxx>` |

> `Fluent API` / `TagHelper` 是**前台写法**、不是模式名；模式说 MVC / RazorForms / RazorPages。RazorForms 与 RazorPages 共用 TagHelper 标签，但数据初始化/事件不同。详见 [references/stacks.md](references/stacks.md)。

## 何时使用（When to Use）

- 判定一个 FineUI 项目属于哪种写法，避免生成错端代码
- 从零搭一个页面（页面骨架、PageManager、Region/ViewPort 布局）
- 用 `F.create` / Fluent / TagHelper 创建组件
- 让文本按 HTML 原样输出（可信 HTML / RawHtml）
- 理解属性命名、全局配置项

## 开始前（Before You Start）—— 先判定写法

> 判定线索：纯前端 `.html` + `F.create` → **F.js**；`.aspx` + `<f:Xxx runat="server">` → **Pro**；`Controllers/` + `Views/*/Index.cshtml` 用 `Html.F()` → **Core-MVC**；`Pages/*.cshtml` + `<f:Xxx>` 标签 → **Core-RazorForms**（有 `.designer.cs`）或 **Core-RazorPages**（无 designer）。

## 各写法速览（创建一个组件）

```javascript
// F.js
F.create({ type: 'Panel', renderTo: '#wrap', id: 'Panel1', title: '面板', bodyPadding: 10 });
```
```aspx
<%-- Pro（WebForms）--%>
<f:Panel ID="Panel1" runat="server" Title="面板" BodyPadding="10px"></f:Panel>
```
```csharp
// Core-MVC（Fluent API）
@(Html.F().Panel().ID("Panel1").Title("面板").BodyPadding(10))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages 相同）-->
<f:Panel ID="Panel1" Title="面板" BodyPadding="10"></f:Panel>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/stacks.md](references/stacks.md) | 五写法差异总表、判定线索、命名约定、全局配置 |
| [references/page-scaffold.md](references/page-scaffold.md) | 最小页面骨架、PageManager 位置、Region/ViewPort 布局、共享 `_Layout` |
| [references/rawhtml.md](references/rawhtml.md) | 可信 HTML：`F.rawHtml` / `XxxRawHtml` / `new RawHtml(...)` |

## 相关技能（Related Skills）

- `fineui-grid`：数据表格（最复杂组件，五写法完整对照）
- `fineui-form` / `fineui-tree` / `fineui-window` / `fineui-panel` / `fineui-layout` / `fineui-buttons-toolbar` / `fineui-theming`：各组件与主题
- `fineui-upgrade`：版本升级（识别破坏性变更）

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：一个页面只用一种写法。F.js 属性 camelCase，C# 三模式 PascalCase；Fluent API（MVC）与 TagHelper（RazorForms/RazorPages）不要混写。
2. **PageManager 位置**：Pro 写在 aspx 页面内（`<f:PageManager>`）；**Core 三模式统一放在共享 `_Layout.cshtml` 的 `@F.PageManager`，业务页不写、也没有 `<f:PageManager>` 标签**。
3. **文本默认 HTML 编码（v15.2）**：要原样输出 HTML 必须声明可信 HTML（`F.rawHtml` / `XxxRawHtml` / `new RawHtml(...)`），来自用户输入/数据库的内容不要声明。详见 [references/rawhtml.md](references/rawhtml.md)。
4. **绝不编造 API**：不确定的属性/方法去查官网 API 或 `F/doc/` 的 JSDoc。不同写法名称不同，别硬套（如 Grid 列 F.js `text`/`field` ↔ C# `HeaderText`/`DataField`）。
5. **Core 启用 TagHelper**：`_ViewImports.cshtml` 需 `@addTagHelper *, FineUICore`；CSS/JS 用 `@F.RenderCss()` / `@F.RenderScript()`。
6. **消息框**：F.js `F.alert({ message: ... })` / `showNotify(...)`；C# `Alert.Show(...)` / `ShowNotify(...)`，Core 回发处理器结尾 `return UIHelper.Result();`。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- 官网与示例：https://www.fineui.com/
