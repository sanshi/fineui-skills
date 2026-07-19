# 布局类型（Layout Types）

容器的 `Layout` 属性决定子控件排布。F.js 小写字符串（`layout:'fit'`），C# 用枚举（`Layout="Fit"` / `.Layout(LayoutType.Fit)`）。

## 取值一览（`LayoutType` 枚举）

`Container`（默认，依次排列）、`Fit`、`Region`、`Block`（响应式）、`HBox`、`VBox`、`Column`、`Anchor`、`Accordion`、`Card`（TabStrip 用）、`Table`、`Absolute`、`InlineBlock`。

> **HBox / VBox 是重点，单独一篇**：见 [hbox-vbox.md](hbox-vbox.md)。

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

## Block —— 响应式栅格（12 块，随屏幕断点变化）

类似 Bootstrap 栅格但**纯 JS 实现**。容器 `Layout="Block"`；子面板用 `Block`/`BlockSM`/`BlockMD`/`BlockLG`（值 1–12，一行总和 12，超出换行）。间距 `BlockConfigSpace`；栅格总数默认 12，可用 `BlockConfigBlockCount` 自定义。

**断点**：`Block`(<768，始终水平) / `BlockSM`(≥768) / `BlockMD`(≥992) / `BlockLG`(≥1200)——屏幕小于该档时层叠排列。

```javascript
// F.js —— layout:'block'（或 layout:{ type:'block', space:10 }）；子项 blockMD/blockLG 等
F.create({ type: 'Panel', renderTo: '#wrap', layout: 'block', isFluid: true, header: false, bodyPadding: 10,
    items: [
        { type: 'Panel', blockMD: 6, blockLG: 4, header: false, items: [{ type: 'Label', value: 'MD=6 LG=4' }] },
        { type: 'Panel', blockMD: 6, blockLG: 4, header: false, items: [{ type: 'Label', value: 'MD=6 LG=4' }] },
        { type: 'Panel', blockMD: 12, blockLG: 4, header: false, items: [{ type: 'Label', value: 'MD=12 LG=4' }] }
    ] });
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel IsFluid="true" runat="server" Layout="Block" BlockConfigSpace="10px" ShowHeader="false">
    <Items>
        <f:Panel Block="6" BlockMD="9" BlockLG="4" runat="server" ShowHeader="false"><Items>
            <f:Label EncodeText="false" Text="Block=6<br/>MD=9<br/>LG=4" runat="server" /></Items></f:Panel>
        <f:Panel Block="6" BlockMD="3" BlockLG="4" runat="server" ShowHeader="false"> ... </f:Panel>
        <f:Panel Block="12" BlockMD="12" BlockLG="4" runat="server" ShowHeader="false"> ... </f:Panel>
    </Items>
</f:Panel>
```
```csharp
// Core-MVC（Fluent）
@(F.Panel().IsFluid(true).Layout(LayoutType.Block).BlockConfigSpace(10).ShowHeader(false)
    .Items(
        F.Panel().Block(6).BlockMD(9).BlockLG(4).ShowHeader(false).Items(F.Label().EncodeText(false).Text("Block=6 MD=9 LG=4")),
        F.Panel().Block(6).BlockMD(3).BlockLG(4).ShowHeader(false).Items(F.Label().Text("...")),
        F.Panel().Block(12).BlockMD(12).BlockLG(4).ShowHeader(false).Items(F.Label().Text("..."))
    ))
```

> `BlockConfigBlockCount="20"` 可把栅格总数从 12 改成别的。**`InlineBlock` 是另一种（非响应式）布局，别与 `Block` 混淆。**

---

## Column / Anchor

- **Column**：`Layout="Column"`，子面板用 `ColumnWidth="60%"`（F.js `columnWidth: 0.6`）或固定 `Width`。官方推荐改用 HBox（见 [hbox-vbox.md](hbox-vbox.md)）。
- **Anchor**：`Layout="Anchor"`（表单默认），子控件用 `AnchorValue="100% 70%"`（百分比）或 `"100% -72"`（百分比 + 像素偏移）。

## See also

- [hbox-vbox.md](hbox-vbox.md)：HBox / VBox 弹性盒子（重点）
- `fineui-panel`：Panel / TabStrip / Accordion 容器
- `fineui-foundation` 的 page-scaffold.md：整页 ViewPort + Region 骨架
