# Panel 面板

容器属性：`Title`、`BodyPadding`、`EnableCollapse`（折叠）、`IconFont`/`IconUrl`、`ShowBorder`、`ShowHeader`、`AutoScroll`、`Height`、`Layout`（子控件排布，见 `fineui-layout`）。工具栏放 `<Toolbars>`（C#）/ `bars`（F.js），内容放 `<Items>` / `items`。纯内容用 `<f:ContentPanel>`。

## 工具栏（顶部/底部）

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

> 独立 Toolbar 只存在于容器的 `<Toolbars>`（Panel/Grid/TabStrip），不直接放在页面上；一个容器可放多个 Toolbar（`Position` + `ToolbarAlign`）。

## 折叠与折叠事件

```javascript
// F.js —— collapsible + 方法
{ type: 'Panel', collapsible: true, ... }   // F.ui.Panel1.toggleCollapse() / isCollapsed()
```
```aspx
<%-- Pro / Core-TagHelper：折叠/展开触发服务端事件 --%>
<f:Panel runat="server" EnableCollapse="true" EnableCollapseEvent="true" OnCollapse="Panel1_Collapse"
         EnableExpandEvent="true" OnExpand="Panel1_Expand"> ... </f:Panel>
```
```csharp
// Core-MVC：客户端读折叠状态回发
@(F.Button().Text("展开/折叠").OnClick(Url.Action("btn_Click"), new Parameter("collapsed", "F.ui.Panel2.collapsed")))
// Controller: UIHelper.Panel("Panel2").Collapsed(!collapsed);
```

## 标题栏工具图标（Tools）

标题右侧的小图标按钮（设置/刷新等）：

```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel ... runat="server">
    <Items> ... </Items>
    <Tools>
        <f:Tool IconFont="_Gear" ToolTip="设置" EnablePostBack="false" runat="server">
            <Listeners><f:Listener Event="click" Handler="onToolClick" /></Listeners>
        </f:Tool>
    </Tools>
</f:Panel>
```

## ContentPanel（纯内容面板）

```aspx
<f:ContentPanel ID="cp1" Title="内容面板" ShowBorder="true" Height="200px" AutoScroll="true" BodyPadding="10px" runat="server">
    可放 HTML 或其它控件的内容
</f:ContentPanel>
```

## See also

- [tab.md](tab.md)：TabStrip 选项卡
- [accordion.md](accordion.md)：Accordion 手风琴
- `fineui-layout`：Panel 的 `Layout` 子控件排布（Fit/Region/HBox/VBox/Block）
