---
name: fineui-panel
description: >
  帮助开发者使用 FineUI 的容器组件：Panel（面板，含工具栏/折叠/标题工具图标/ContentPanel/GroupPanel）、
  TabStrip（选项卡，含 iframe 页、动态增删）、Accordion（手风琴）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper），
  以及 FineUIJava（Spring Boot + Thymeleaf 方言标签，kebab-case）。
  Trigger phrases（触发词）: "FineUI 面板", "F.Panel", "Panel", "ContentPanel", "工具栏面板",
  "TabStrip", "选项卡", "标签页", "动态选项卡", "iframe 选项卡", "Accordion", "手风琴", "AccordionPane",
  "折叠面板", "GroupPanel", "分组面板", "Tools 标题图标", "FineUIJava", "Spring Boot", "Thymeleaf", "@FineUIPage",
  "f:panel", "f:tab-strip", "f:accordion", "f:region-panel".
metadata:
  author: FineUI
  version: "16.0"
  compatibility: FineUI v16.0（事件驱动变化回发）
---

# FineUI 容器组件技能（Panel / Tab / Accordion）

> 本技能讲**容器组件本身**（Panel/选项卡/手风琴）；容器内部子控件**如何排布**（Fit/Region/HBox/VBox/Block…）见 `fineui-layout` 技能。

## 何时使用（When to Use）

- **Panel**：带标题/工具栏/折叠的面板容器；纯内容用 ContentPanel
- **GroupPanel**：在表单或页面中把一组相关控件放进带标题边框的可折叠区域
- **TabStrip**：选项卡（静态标签、iframe 标签、动态增删）
- **Accordion**：手风琴（同一时刻展开一个面板）

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / Java（判定见 `fineui-foundation`）。
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
```html
<!-- ⑤ FineUIJava（Thymeleaf 方言：标签/属性全 kebab-case，工具栏用 <f:toolbars>/<f:toolbar>）-->
<f:panel id="Panel1" is-fluid="true" title="面板" body-padding="10" enable-collapse="true">
    <f:toolbars>
        <f:toolbar id="Toolbar1" position="Top">
            <f:items><f:button id="btn1" text="按钮"></f:button><f:toolbar-fill></f:toolbar-fill></f:items>
        </f:toolbar>
    </f:toolbars>
    <f:items><f:panel title="内容面板" height="200" auto-scroll="true"></f:panel></f:items>
</f:panel>
```

> 客户端 F.js 运行时四栈完全相同（`F.ui.Panel1.xxx()`、监听器 JS 一字不差），Java 只是模板语法/大小写与页面类语言不同。详见 `fineui-foundation` 的 stacks.md。

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/panel.md](references/panel.md) | Panel / ContentPanel：工具栏、折叠与折叠事件、标题栏工具图标 |
| [references/group-panel.md](references/group-panel.md) | GroupPanel 分组面板：嵌套控件、折叠、工具栏与服务端控制 |
| [references/tab.md](references/tab.md) | TabStrip：静态/iframe 标签、活动页、动态增删标签 |
| [references/accordion.md](references/accordion.md) | Accordion / AccordionPane：面板组、活动面板、切换事件 |

## 相关技能（Related Skills）

- `fineui-layout`：容器内部布局（Fit / Region / HBox / VBox / Block…）
- `fineui-grid` / `fineui-form`：放进 Fit 面板或 Tab / Pane 的内容
- `fineui-button`：工具栏里的按钮及其下拉菜单
- `fineui-window`：Tab / 面板里嵌 iframe 编辑页

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js camelCase（`collapsible`/`bodyPadding`）；C# PascalCase（`EnableCollapse`/`BodyPadding`）；**Java kebab-case（`enable-collapse`/`body-padding`）**。工具栏 F.js 放 `bars`，C# 放 `<Toolbars>`，**Java 放 `<f:toolbars>`/`<f:toolbar>`/`<f:items>`（内含 `<f:toolbar-text>`/`<f:toolbar-separator>`/`<f:button>`/`<f:toolbar-fill>`）**。
2. **纯内容用 ContentPanel**：只放内容、无需再嵌套时用 `<f:ContentPanel>`（C#）/ **`<f:content-panel>`（Java）**。
3. **分组内容用 GroupPanel**：需要带标题边框组织一组表单字段或子控件时用 GroupPanel；它可以折叠、包含 Items 和 Toolbars，不是表单字段本身。
4. **动态选项卡各写法不同**：F.js `F.ui.TabStrip1.addTab({...})`；Core-MVC/RazorPages `UIHelper.TabStrip("id").AddTab(...)`；Pro/RazorForms `RegisterStartupScript(TabStrip1.GetAddTabReference(...))`；**Java 直接在控件字段上调 `TabStrip1.addTab(id, url, title, iconUrl, closable)` / `TabStrip1.hideTab(id)`**。详见 [references/tab.md](references/tab.md)。
5. **Accordion 面板用 `<Panes>`/`AccordionPane`**（Java `<f:panes>`/`<f:accordion-pane>`；不是 `<Items>`/Tab）；面板切换**服务端事件** `OnPaneIndexChanged`（Java `on-pane-index-changed`）在 **Pro / Core-RazorForms / Java** 上都能直接声明。Pro 推荐设置 `EnableImplicitChangeEvents="false"`，声明事件即可自动回发；Core-MVC/RazorPages 读客户端 `getActivePaneIndex()`。详见 [references/accordion.md](references/accordion.md)。
6. **Java 折叠事件直接写属性**：`on-collapse`/`on-expand`（不像 Pro 需 `EnableCollapseEvent`/`EnableExpandEvent` 开关）。Region 布局 Java 另有 `<f:region-panel>`/`<f:regions>`/`<f:region>` 便捷控件（见 `fineui-layout`）。
7. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc；Java 属性名 = Core 属性名转 kebab-case，属性「值」（枚举/图标）仍 PascalCase。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- **FineUIJava**：控件属性语义同 Core（属性名转 kebab-case、值保持 PascalCase），客户端 F.js API 与 JS 端完全相同；查属性先看 Core API 再按命名约定转写。
