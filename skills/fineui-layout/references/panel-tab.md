# Panel 面板 与 TabStrip 选项卡

## 一、Panel 面板

容器属性：`Title`、`BodyPadding`、`EnableCollapse`（折叠）、`IconFont`/`IconUrl`、`ShowBorder`、`ShowHeader`、`AutoScroll`、`Height`、`Layout`。工具栏放 `<Toolbars>`（C#）/ `bars`（F.js），内容放 `<Items>` / `items`。

### 工具栏（顶部/底部）

工具栏项：`ToolbarText`、`ToolbarSeparator`（分隔线）、`Button`、`ToolbarFill`（撑开右对齐）。F.js 的 `'-'` = 分隔线、`'->'` = 填充。`Position="Top"` / `"Bottom"`。

```javascript
// F.js
bars: [{ type: 'Toolbar', position: 'top', items: [
    { type: 'ToolbarText', text: '文本' }, '-', { type: 'Button', text: '按钮' }, '->', { type: 'ToolbarText', text: '右' }
] }]
```
```aspx
<%-- Pro / Core-TagHelper --%>
<Toolbars>
    <f:Toolbar Position="Top" runat="server"><Items>
        <f:ToolbarText Text="文本" runat="server" /><f:ToolbarSeparator runat="server" />
        <f:Button Text="按钮" runat="server" /><f:ToolbarFill runat="server" />
    </Items></f:Toolbar>
</Toolbars>
```
```csharp
// Core-MVC（Fluent）
.Toolbars(F.Toolbar().Position(ToolbarPosition.Top).Items(
    F.ToolbarText().Text("文本"), F.ToolbarSeparator(), F.Button().Text("按钮"), F.ToolbarFill()))
```

### 折叠与折叠事件

```javascript
// F.js
{ type: 'Panel', collapsible: true, ... }   // F.ui.Panel1.toggleCollapse() / isCollapsed()
```
```aspx
<%-- Pro / Core-TagHelper：折叠触发服务端事件 --%>
<f:Panel EnableCollapse="true" EnableCollapseEvent="true" OnCollapse="Panel1_Collapse"
         EnableExpandEvent="true" OnExpand="Panel1_Expand" runat="server"> ... </f:Panel>
```

### 标题栏工具图标（Tools）

```aspx
<%-- Pro / Core-TagHelper：标题右侧的小图标按钮 --%>
<f:Panel ... runat="server">
    <Items> ... </Items>
    <Tools>
        <f:Tool IconFont="_Gear" ToolTip="设置" EnablePostBack="false" runat="server">
            <Listeners><f:Listener Event="click" Handler="onToolClick" /></Listeners>
        </f:Tool>
    </Tools>
</f:Panel>
```

---

## 二、TabStrip 选项卡

容器：`Height`、`TabPosition="Top"`、`ActiveTabIndex`（默认激活）、`EnableTabCloseMenu`、`ShowBorder`。每个 `Tab` 有 `Title`/`TitleRawHtml`、`BodyPadding`、`Layout`、`Closable`、`Disabled`、`Icon`；内容用 `<Items>`/`items`，简单文本可用 `content`(F.js)/`Content`。

```javascript
// F.js
F.create({ type: 'TabStrip', isFluid: true, id: 'TabStrip1', renderTo: '#wrap', height: 350, tabPosition: 'top', activeTabIndex: 1,
    defaults: { autoScroll: true, bodyPadding: 10 },
    items: [
        { type: 'Tab', title: '标签一', layout: 'fit', items: [ /* 表单/表格 */ ] },
        { type: 'Tab', title: F.rawHtml('<span class="hot">标签二</span>'), content: '内容', closable: true },
        { type: 'Tab', title: '标签三', disabled: true, content: '内容' }
    ] });
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:TabStrip ID="TabStrip1" runat="server" IsFluid="true" Height="350px" TabPosition="Top" ActiveTabIndex="1">
    <Tabs>
        <f:Tab Title="标签一" BodyPadding="10px" Layout="Fit" runat="server"><Items> ... </Items></f:Tab>
        <f:Tab TitleRawHtml="<span class='hot'>标签二</span>" BodyPadding="10px" runat="server"><Items>
            <f:Button Text="按钮" runat="server" /></Items></f:Tab>
    </Tabs>
</f:TabStrip>
<%-- Core-TagHelper 富文本标题用下划线便捷形式：_TitleRawHtml="<span ...>标签二</span>" --%>
```
```csharp
// Core-MVC（Fluent）
@(F.TabStrip().IsFluid(true).ID("TabStrip1").Height(350).TabPosition(TabPosition.Top).ActiveTabIndex(1)
    .Tabs(
        F.Tab().Title("标签一").BodyPadding(10).Layout(LayoutType.Fit).Items( /* ... */ ),
        F.Tab().TitleRawHtml(new RawHtml("<span class='hot'>标签二</span>")).BodyPadding(10).Items(F.Button().Text("按钮"))
    ))
```

### Tab 内嵌 iframe

```aspx
<%-- Pro / Core-TagHelper --%>
<f:Tab Title="标签二（IFrame）" EnableIFrame="true" IFrameUrl="~/Panel/Group" runat="server">
    <Listeners><f:Listener Event="iframeload" Handler="onTabIFrameLoad" /></Listeners>
</f:Tab>
```
```csharp
// Core-MVC（Fluent）
F.Tab().Title("标签二（IFrame）").EnableIFrame(true).IFrameUrl(Url.Content("~/Panel/Group")).Listener("iframeload", "onTabIFrameLoad")
```

---

## 三、动态增删选项卡

### 客户端（各写法一致）

```javascript
F.ui.TabStrip1.addTab({ id: 'tab_x', iframe: true, iframeUrl: 'https://x.com/', title: '新标签', closable: true, icon: '/res/icon/x.png' });
F.ui.TabStrip1.closeTab('tab_x');
F.ui.TabStrip1.activeNextTab();
```

### 服务端

```csharp
// Core-MVC / RazorPages —— UIHelper
UIHelper.TabStrip("TabStrip1").AddTab("tab_x", "https://x.com/", "新标签", IconHelper.GetIconUrl(Icon.Application), true);
UIHelper.TabStrip("TabStrip1").CloseTab("tab_x");
```
```csharp
// Pro / Core-RazorForms —— 注册脚本 + GetAddTabReference
PageContext.RegisterStartupScript(TabStrip1.GetAddTabReference("tab_x", "https://x.com/", "新标签", IconHelper.GetIconUrl(Icon.Application), true));
// RazorForms 用 RegisterStartupScript(TabStrip1.GetAddTabReference(...))
```

> **相同 `id` 会复用同一个选项卡**（再次打开不新建）。动态 Tab 是客户端加的，服务端控件树取不到——非 Ajax 回发会丢失。

## See also

- [layout-types.md](layout-types.md)：Fit / Region / HBox / VBox
- `fineui-window`：Tab 里放 iframe 编辑页
