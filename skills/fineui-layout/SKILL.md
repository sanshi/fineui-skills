---
name: fineui-layout
description: >
  帮助开发者使用 FineUI 的布局与容器：Panel（面板，含工具栏/折叠/标题工具图标）、TabStrip（选项卡，含 iframe 页、动态增删）、
  以及布局类型（Fit / Region 区域 / HBox / VBox / Column / Anchor）与视口自适应（IsViewPort / AutoSizePanelID）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper）。
  Trigger phrases（触发词）: "FineUI 布局", "F.Panel", "Panel", "面板", "工具栏", "Toolbar",
  "TabStrip", "选项卡", "标签页", "动态选项卡", "addTab", "iframe 选项卡",
  "Region 布局", "区域布局", "HBox", "VBox", "Fit 布局", "IsViewPort", "视口自适应", "RegionPosition".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 布局技能（Layout）

## 何时使用（When to Use）

- 面板容器：标题、工具栏（顶/底）、折叠、标题栏工具图标
- 选项卡：静态标签、iframe 标签、动态增删标签
- 页面布局：区域布局（上下左右中）、Fit 填充、HBox/VBox/Column 分栏、视口自适应

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages（判定见 `fineui-foundation`）。
2. **选哪种布局？** 见下表（`Layout` 属性）。整页骨架（ViewPort + Region）另见 `fineui-foundation` 的 page-scaffold.md。

## 布局类型速查（`Layout` 属性）

| 值 | 用途 |
|----|------|
| `Container`（默认） | 子控件按自身高度依次排列 |
| `Fit` | **唯一**子控件铺满容器（放一个 Grid/Form 常用） |
| `Region` | 区域布局：子面板用 `RegionPosition` 定位 Top/Left/Center/Right/Bottom（Center 必填，占满剩余） |
| `HBox` / `VBox` | 水平 / 垂直盒子，子用 `BoxFlex` 弹性分配 |
| `Column` | 列布局，子用列宽（官方推荐改用 HBox） |
| `Anchor` | 锚点布局（表单默认），子用 `AnchorValue`（如 `"100% 70%"`） |
| `Accordion` / `Card` / `Table` / `Absolute` / `Block` | 手风琴 / 卡片(TabStrip) / 表格 / 绝对 / 响应式块 |

## 各写法速览（Panel + 顶部工具栏）

```javascript
// ① F.js —— bars 放工具栏
F.create({ type: 'Panel', isFluid: true, id: 'Panel1', renderTo: '#wrap', title: '面板', bodyPadding: 10, collapsible: true,
    bars: [{ type: 'Toolbar', position: 'top', items: [
        { type: 'ToolbarText', text: '文本' }, '-', { type: 'Button', text: '按钮' }, '->', { type: 'ToolbarText', text: '右侧' }
    ] }],
    items: [{ type: 'Panel', title: '内容面板', height: 200, autoScroll: true, contentEl: '#c1' }] });
```
```aspx
<%-- ② Pro（WebForms）--%>
<f:Panel ID="Panel1" runat="server" IsFluid="true" Title="面板" BodyPadding="10px" EnableCollapse="true">
    <Toolbars>
        <f:Toolbar ID="Toolbar1" Position="Top" runat="server"><Items>
            <f:ToolbarText Text="文本" runat="server" /><f:ToolbarSeparator runat="server" />
            <f:Button Text="按钮" runat="server" /><f:ToolbarFill runat="server" />
        </Items></f:Toolbar>
    </Toolbars>
    <Items><f:ContentPanel Title="内容面板" Height="200px" AutoScroll="true" runat="server">内容</f:ContentPanel></Items>
</f:Panel>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Panel().IsFluid(true).ID("Panel1").Title("面板").BodyPadding(10).EnableCollapse(true)
    .Toolbars(F.Toolbar().Position(ToolbarPosition.Top).Items(
        F.ToolbarText().Text("文本"), F.ToolbarSeparator(), F.Button().Text("按钮"), F.ToolbarFill()))
    .Items(F.Panel().Title("内容面板").Height(200).AutoScroll(true).Content("内容")))
```
```html
<!-- ④ Core-TagHelper（RazorForms / RazorPages，写法相同）-->
<f:Panel ID="Panel1" IsFluid="true" Title="面板" BodyPadding="10" EnableCollapse="true">
    <Toolbars><f:Toolbar ID="Toolbar1" Position="Top"><Items>
        <f:ToolbarText Text="文本"></f:ToolbarText><f:ToolbarSeparator></f:ToolbarSeparator>
        <f:Button Text="按钮"></f:Button><f:ToolbarFill></f:ToolbarFill>
    </Items></f:Toolbar></Toolbars>
    <Items><f:Panel Title="内容面板" Height="200" AutoScroll="true"></f:Panel></Items>
</f:Panel>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/panel-tab.md](references/panel-tab.md) | Panel（工具栏/折叠/标题工具）、TabStrip（静态/iframe/动态增删标签） |
| [references/layout-types.md](references/layout-types.md) | Fit / Region / HBox / VBox / Column / Anchor + 视口自适应 |

## 相关技能（Related Skills）

- `fineui-foundation`：整页骨架（ViewPort + Region + PageManager 位置）
- `fineui-grid` / `fineui-form`：放进 Fit 面板或 Tab 的内容

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js camelCase（`collapsible`/`bodyPadding`）；C# PascalCase（`EnableCollapse`/`BodyPadding`）。工具栏 F.js 放 `bars`，C# 放 `<Toolbars>`。
2. **Fit 只放一个子控件**：`Layout="Fit"` 时容器只应有一个子控件（它铺满）。
3. **Region 的 Center 必填**：区域布局必须有一个 `RegionPosition="Center"` 的面板占满剩余空间；左右用 `Width`、上下用 `Height`；`RegionSplit="true"` 加拖动条。
4. **视口自适应两种入口**：根面板 `IsViewPort="true"`，或 `<f:PageManager AutoSizePanelID="Panel1">`（Pro）。
5. **动态选项卡各写法不同**：F.js `F.ui.TabStrip1.addTab({...})`；Core-MVC/RazorPages 服务端 `UIHelper.TabStrip("id").AddTab(...)`；Pro/RazorForms 服务端 `RegisterStartupScript(TabStrip1.GetAddTabReference(...))`。详见 [references/panel-tab.md](references/panel-tab.md)。
6. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
