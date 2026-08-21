# Panel 面板

容器属性：`Title`、`BodyPadding`、`EnableCollapse`（折叠）、`IconFont`/`IconUrl`、`ShowBorder`、`ShowHeader`、`AutoScroll`、`Height`、`Layout`（子控件排布，见 `fineui-layout`）。工具栏放 `<Toolbars>`（C#）/ `bars`（F.js），内容放 `<Items>` / `items`。纯内容用 `<f:ContentPanel>`。

> **Java（Thymeleaf 方言）**：标签/属性全 kebab-case——`<f:panel title= body-padding= enable-collapse= icon-font= show-border= show-header= auto-scroll= height= is-fluid= layout=>`；工具栏放 `<f:toolbars>`/`<f:toolbar position="Top">`/`<f:items>`，内容放 `<f:items>`，纯内容用 `<f:content-panel>`。属性「值」（图标 `icon-font="_VolumeUp"`、`icon="TagBlue"`）仍 PascalCase。服务端读写走 Bean 方法：`Panel1.setTitle(...)` / `Panel1.setContent(...)` / `Panel1.isCollapsed()` / `Panel1.setCollapsed(...)` / `Panel1.setIconFont(IconFont._VolumeUp)`。

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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:toolbars>
    <f:toolbar id="Toolbar1" position="Top">
        <f:items>
            <f:toolbar-text id="ToolbarText1" text="文本"></f:toolbar-text>
            <f:toolbar-separator id="ToolbarSeparator1"></f:toolbar-separator>
            <f:button id="btn1" text="按钮"></f:button>
            <f:toolbar-fill id="ToolbarFill1"></f:toolbar-fill>
        </f:items>
    </f:toolbar>
</f:toolbars>
```

> 独立 Toolbar 只存在于容器的 `<Toolbars>` / `<f:toolbars>`（Panel/Grid/TabStrip），不直接放在页面上；一个容器可放多个 Toolbar（`Position` + `ToolbarAlign`）。服务端隐藏/显示工具项：Java `ToolbarText1.setHidden(true)` / `Toolbar1.setHidden(true)`。

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
```html
<!-- FineUIJava（Thymeleaf 方言）：折叠/展开服务端事件直接写 on-collapse / on-expand（无需 EnableCollapseEvent 开关）-->
<f:panel id="Panel1" enable-collapse="true" on-collapse="Panel1_CollapseExpand" on-expand="Panel1_CollapseExpand">
    <f:items> ... </f:items>
</f:panel>
```
```java
// FineUIJava 页面类：读折叠状态 / 服务端切换折叠
public void Panel1_CollapseExpand(Object sender, EventArgs e) {
    showNotify("面板处于" + (Panel1.isCollapsed() ? "折叠" : "展开") + "状态");
}
public void Button3_Click(Object sender, EventArgs e) {
    Panel2.setCollapsed(!Panel2.isCollapsed());   // 服务端展开/折叠
}
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
```html
<!-- FineUIJava（Thymeleaf 方言）：<f:tools> 内放 <f:tool>，可带 text 显示文字 -->
<f:panel id="Panel1" title="面板" icon-url="~/res/images/16/8.png">
    <f:items> ... </f:items>
    <f:tools>
        <f:tool icon-font="_Gear" tool-tip="设置">
            <f:listeners><f:listener event="click" handler="onToolClick" /></f:listeners>
        </f:tool>
        <f:tool id="Tool4" icon-font="_Save" text="保存" tool-tip="保存">
            <f:listeners><f:listener event="click" handler="onToolClick" /></f:listeners>
        </f:tool>
    </f:tools>
</f:panel>
```

> Tools 图标的 `click` 处理器是**客户端 JS**（与 F.js 一致，如 `function onToolClick(){ var iconFont=this.iconFont; ... }`），不重复贴。

## ContentPanel（纯内容面板）

```aspx
<f:ContentPanel ID="cp1" Title="内容面板" ShowBorder="true" Height="200px" AutoScroll="true" BodyPadding="10px" runat="server">
    可放 HTML 或其它控件的内容
</f:ContentPanel>
```
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:content-panel id="cp1" title="内容面板" show-border="true" height="200" auto-scroll="true" body-padding="10">
    可放 HTML 或其它控件的内容
</f:content-panel>
```

## See also

- [tab.md](tab.md)：TabStrip 选项卡
- [accordion.md](accordion.md)：Accordion 手风琴
- `fineui-layout`：Panel 的 `Layout` 子控件排布（Fit/Region/HBox/VBox/Block）
