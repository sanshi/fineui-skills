---
name: fineui-panel
description: >
  帮助开发者使用 FineUI 的容器组件：Panel（面板，含工具栏/折叠/标题工具图标/ContentPanel）、
  TabStrip（选项卡，含 iframe 页、动态增删）、Accordion（手风琴）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper）。
  Trigger phrases（触发词）: "FineUI 面板", "F.Panel", "Panel", "ContentPanel", "工具栏面板",
  "TabStrip", "选项卡", "标签页", "动态选项卡", "iframe 选项卡", "Accordion", "手风琴", "AccordionPane",
  "折叠面板", "Tools 标题图标".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 容器组件技能（Panel / Tab / Accordion）

> 本技能讲**容器组件本身**（Panel/选项卡/手风琴）；容器内部子控件**如何排布**（Fit/Region/HBox/VBox/Block…）见 `fineui-layout` 技能。

## 何时使用（When to Use）

- **Panel**：带标题/工具栏/折叠的面板容器；纯内容用 ContentPanel
- **TabStrip**：选项卡（静态标签、iframe 标签、动态增删）
- **Accordion**：手风琴（同一时刻展开一个面板）

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages（判定见 `fineui-foundation`）。
2. **要哪种容器？** Panel（通用）/ TabStrip（分页）/ Accordion（折叠面板组）。

## 各写法速览（带顶部工具栏的面板）

```javascript
// ① F.js —— bars 放工具栏，items 放内容
F.create({ type: 'Panel', isFluid: true, id: 'Panel1', renderTo: '#wrap', title: '面板', bodyPadding: 10, collapsible: true,
    bars: [{ type: 'Toolbar', position: 'top', items: [{ type: 'Button', text: '按钮' }, '->', { type: 'ToolbarText', text: '右侧' }] }],
    items: [{ type: 'Panel', title: '内容面板', height: 200, autoScroll: true }] });
```
```aspx
<%-- ② Pro（WebForms）--%>
<f:Panel ID="Panel1" runat="server" IsFluid="true" Title="面板" BodyPadding="10px" EnableCollapse="true">
    <Toolbars><f:Toolbar Position="Top" runat="server"><Items>
        <f:Button Text="按钮" runat="server" /><f:ToolbarFill runat="server" />
    </Items></f:Toolbar></Toolbars>
    <Items><f:ContentPanel Title="内容面板" Height="200px" AutoScroll="true" runat="server">内容</f:ContentPanel></Items>
</f:Panel>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Panel().IsFluid(true).ID("Panel1").Title("面板").BodyPadding(10).EnableCollapse(true)
    .Toolbars(F.Toolbar().Position(ToolbarPosition.Top).Items(F.Button().Text("按钮"), F.ToolbarFill()))
    .Items(F.Panel().Title("内容面板").Height(200).AutoScroll(true)))
```
```html
<!-- ④ Core-TagHelper（RazorForms / RazorPages，写法相同）-->
<f:Panel ID="Panel1" IsFluid="true" Title="面板" BodyPadding="10" EnableCollapse="true">
    <Toolbars><f:Toolbar Position="Top"><Items><f:Button Text="按钮"></f:Button><f:ToolbarFill></f:ToolbarFill></Items></f:Toolbar></Toolbars>
    <Items><f:Panel Title="内容面板" Height="200" AutoScroll="true"></f:Panel></Items>
</f:Panel>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/panel.md](references/panel.md) | Panel / ContentPanel：工具栏、折叠与折叠事件、标题栏工具图标 |
| [references/tab.md](references/tab.md) | TabStrip：静态/iframe 标签、活动页、动态增删标签 |
| [references/accordion.md](references/accordion.md) | Accordion / AccordionPane：面板组、活动面板、切换事件 |

## 相关技能（Related Skills）

- `fineui-layout`：容器内部布局（Fit / Region / HBox / VBox / Block…）
- `fineui-grid` / `fineui-form`：放进 Fit 面板或 Tab / Pane 的内容
- `fineui-buttons-toolbar`：工具栏里的按钮/菜单
- `fineui-window`：Tab / 面板里嵌 iframe 编辑页

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js camelCase（`collapsible`/`bodyPadding`）；C# PascalCase（`EnableCollapse`/`BodyPadding`）。工具栏 F.js 放 `bars`，C# 放 `<Toolbars>`。
2. **纯内容用 ContentPanel**（C#）：只放内容、无需再嵌套时用 `<f:ContentPanel>`。
3. **动态选项卡各写法不同**：F.js `F.ui.TabStrip1.addTab({...})`；Core-MVC/RazorPages `UIHelper.TabStrip("id").AddTab(...)`；Pro/RazorForms `RegisterStartupScript(TabStrip1.GetAddTabReference(...))`。详见 [references/tab.md](references/tab.md)。
4. **Accordion 面板用 `<Panes>`/`AccordionPane`**（不是 `<Items>`/Tab）；面板切换服务端事件（`OnPaneIndexChanged`）仅 Pro 有，Core 三模式读客户端 `getActivePaneIndex()`。详见 [references/accordion.md](references/accordion.md)。
5. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
