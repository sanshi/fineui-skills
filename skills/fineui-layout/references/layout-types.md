# 布局类型（Layout Types）

容器的 `Layout` 属性决定子控件排布。F.js 小写字符串（`layout:'fit'`），C# 用枚举（`Layout="Fit"` / `.Layout(LayoutType.Fit)`）。

## 取值一览（`LayoutType` 枚举）

`Container`（默认，依次排列）、`Fit`、`Region`、`HBox`、`VBox`、`Column`、`Anchor`、`Accordion`、`Card`（TabStrip 用）、`Table`、`Absolute`、`Block`（响应式）。

---

## Fit —— 唯一子控件铺满

放**一个** Grid / Form / Panel，让它填满整个容器。

```javascript
// F.js
{ type: 'Panel', layout: 'fit', height: 300, items: [{ type: 'Grid', ... }] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel Layout="Fit" Height="300px" runat="server"><Items><f:Grid runat="server"> ... </f:Grid></Items></f:Panel>
```
```csharp
// Core-MVC（Fluent）
@(F.Panel().Layout(LayoutType.Fit).Height(300).Items(F.Grid().ID("Grid1")))
```

---

## Region —— 区域布局（上下左右中）

子面板用 `RegionPosition` 定位；**必须有一个 `Center`**（占满剩余）；左右用 `Width`、上下用 `Height`；`RegionSplit="true"` 加拖动分隔条；`Collapsed="true"` 初始折叠。

**视口自适应**：根面板 `IsViewPort="true"`（铺满浏览器），或 Pro 用 `<f:PageManager AutoSizePanelID="Panel1">`。

```javascript
// F.js
F.create({ type: 'Panel', renderTo: document.body, isViewPort: true, layout: 'region',
    items: [
        { type: 'Panel', region: 'top', height: 60, split: true, title: '顶部' },
        { type: 'Panel', region: 'left', width: 200, split: true, collapsible: true, title: '左侧' },
        { type: 'Panel', region: 'center', title: '中间' },
        { type: 'Panel', region: 'bottom', height: 60, split: true, title: '底部' }
    ] });
```
```aspx
<%-- Pro：PageManager.AutoSizePanelID 让根面板随窗口自适应 --%>
<f:PageManager ID="PageManager1" AutoSizePanelID="Panel1" runat="server" />
<f:Panel ID="Panel1" runat="server" ShowBorder="false" ShowHeader="false" Layout="Region">
    <Items>
        <f:Panel RegionPosition="Top" RegionSplit="true" Height="60px" Title="顶部" runat="server" />
        <f:Panel RegionPosition="Left" RegionSplit="true" Width="200px" EnableCollapse="true" Title="左侧" runat="server" />
        <f:Panel RegionPosition="Center" AutoScroll="true" Title="中间" runat="server" />
        <f:Panel RegionPosition="Bottom" RegionSplit="true" Height="60px" Title="底部" runat="server" />
    </Items>
</f:Panel>
```
```csharp
// Core-MVC（Fluent）：IsViewPort 铺满视口；RegionPosition 用 Position 枚举
@(F.Panel().ID("Panel1").ShowBorder(false).ShowHeader(false).Layout(LayoutType.Region).IsViewPort(true)
    .Items(
        F.Panel().RegionPosition(Position.Top).RegionSplit(true).Height(60).Title("顶部"),
        F.Panel().RegionPosition(Position.Left).RegionSplit(true).Width(200).EnableCollapse(true).Title("左侧"),
        F.Panel().RegionPosition(Position.Center).AutoScroll(true).Title("中间"),   // Center 不设宽高
        F.Panel().RegionPosition(Position.Bottom).RegionSplit(true).Height(60).Title("底部")
    ))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages）-->
<f:Panel ID="Panel1" ShowBorder="false" ShowHeader="false" Layout="Region" IsViewPort="true">
    <Items>
        <f:Panel RegionPosition="Top" RegionSplit="true" Height="60" Title="顶部"></f:Panel>
        <f:Panel RegionPosition="Left" RegionSplit="true" Width="200" EnableCollapse="true" Title="左侧"></f:Panel>
        <f:Panel RegionPosition="Center" AutoScroll="true" Title="中间"></f:Panel>
    </Items>
</f:Panel>
```

---

## HBox / VBox —— 弹性盒子

水平/垂直排列，子用 `BoxFlex` 弹性分配剩余空间（或固定 `Width`/`Height`）。容器 `BoxConfigAlign`（Stretch/Center/End…）、`BoxConfigPosition`（Start/Center/End）、`BoxConfigChildMargin`（子项间距 `"0 5 0 0"`）。

```javascript
// F.js —— HBox
{ type: 'Panel', layout: 'hbox', boxConfigAlign: 'stretch', items: [
    { type: 'Panel', boxFlex: 1, title: '左' }, { type: 'Panel', width: 200, title: '右固定' }
] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel Layout="HBox" BoxConfigAlign="Stretch" BoxConfigChildMargin="0 5 0 0" runat="server">
    <Items>
        <f:Panel BoxFlex="1" Title="左" runat="server" /><f:Panel Width="200px" Title="右固定" runat="server" />
    </Items>
</f:Panel>
```
```csharp
// Core-MVC（Fluent）
@(F.Panel().Layout(LayoutType.HBox).BoxConfigAlign(BoxLayoutAlign.Stretch).BoxConfigChildMargin("0 5 0 0")
    .Items(F.Panel().BoxFlex(1).Title("左"), F.Panel().Width(200).Title("右固定")))
```

VBox 同理，方向为垂直（子用 `BoxFlex` 分高）。

---

## Column / Anchor

- **Column**：`Layout="Column"`，子面板用 `ColumnWidth="60%"`（Pro/F.js `columnWidth: 0.6`）或固定 `Width`。官方推荐改用 HBox。
- **Anchor**：`Layout="Anchor"`（表单默认），子控件用 `AnchorValue="100% 70%"`（百分比）或 `"100% -72"`（百分比 + 像素偏移）。

## See also

- [panel-tab.md](panel-tab.md)：Panel 工具栏、TabStrip
- `fineui-foundation` 的 page-scaffold.md：整页 ViewPort + Region 骨架
