# Grid 表头与多表头（Header Options & Group Header）

表头相关：隐藏表头、表头菜单开关、列管理菜单、表头提示、列自定义属性、**多表头（分组表头）**、**动态创建列**。

> 命名规律：**F.js camelCase，四个 .NET 栈都是同词的 PascalCase**，且列宽/菜单几个名字 F.js 与 .NET **不同源**（下表已标注），别机械套用。

## 概念 → 各写法属性名对照

| 概念 | F.js | .NET（Pro / Core-MVC / Core-TagHelper） |
|------|------|------------------------------------------|
| 隐藏列头行 | `gridHeader: false` | `ShowGridHeader="false"` / `.ShowGridHeader(false)` |
| 隐藏面板标题栏 | `header: false` | `ShowHeader="false"` / `.ShowHeader(false)` |
| 关表头下拉菜单 | `columnMenu: false` | `EnableHeaderMenu="false"` |
| 关列宽拖动 | `columnResizable: false` | `EnableColumnResize="false"` |
| 关表头排序菜单项 | `columnMenuSort: false` | `EnableHeaderMenuSort="false"` |
| 表头提示 / 位置 | `headerTooltip` / `headerTooltipPosition` | `HeaderToolTip` / `HeaderToolTipPosition` |
| 列自定义属性 | 列 `attrs: { ... }` | 见 §5（三栈写法不同） |
| 多表头（分组） | 列嵌套 `columns: [ ... ]` | `<f:GroupField>` + 嵌套 `<Columns>` / `F.GroupField()` |
| 运行时重配列 | `grid.configColumns(...)` | 服务端 `Grid.Columns.Add(...)` |

---

## 1. 隐藏表头

两个独立开关：**列头行**（`ShowGridHeader`）和**面板标题栏**（`ShowHeader`）互不影响。

```javascript
// F.js —— 同时隐藏列头行、去掉行线与隔行变色
{ type: 'Grid', header: false, gridHeader: false, rowLines: false, altRowColor: false, ... }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... ShowGridHeader="false" EnableRowLines="false" EnableAlternateRowColor="false"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().ShowGridHeader(false).EnableRowLines(false).EnableAlternateRowColor(false) ...)
```

---

## 2. 表头文字换行（无专用属性，用 CSS）

**任何栈都没有"表头换行"属性**——给该列固定 `Width`，再写 CSS 命中该列头文本元素（选择器按列 ID：`.f-grid-colheader-<ColumnID> .f-grid-colheader-text`）。

```css
/* 列 ColumnID=Major 的表头换行；加下面 4 行可限制最多 3 行省略 */
.f-grid-colheader-Major .f-grid-colheader-text { white-space: normal; word-break: break-all; }
.f-grid-colheader-Major .f-grid-colheader-text {
    display: -webkit-box; -webkit-box-orient: vertical; -webkit-line-clamp: 3; overflow: hidden;
}
```

---

## 3. 禁用表头菜单与列宽调整

```javascript
// F.js —— 注意 F.js 名字是 columnMenu / columnResizable（与 .NET 不同源）
{ type: 'Grid', columnMenu: false, columnResizable: false, ... }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... EnableHeaderMenu="false" EnableColumnResize="false"> ... </f:Grid>
```

**只关排序菜单项、保留菜单**：`columnMenu: true` + `columnMenuSort: false`（.NET `EnableHeaderMenu="true"` + `EnableHeaderMenuSort="false"`）。

---

## 4. 列管理菜单（显示/隐藏列）

范式：关掉内置表头菜单 → 加一个标题栏 `Tool`（图标 `_ColumnsAlt`）→ 点击时调用 Grid 方法 **`showColumnsMenu(toolEl)`** 弹出内置的列显隐菜单。

```javascript
// F.js
{ type: 'Grid', columnMenu: false, columns: [ ... ],
  tools: [{ type: 'Tool', id: 'toolColumns', iconFont: 'f-iconfont-columns-alt', text: '管理列',
    tooltip: '显示隐藏列', tabIndex: 0,
    listeners: { click: function () { F.ui.grid1.showColumnsMenu(this.el); } } }] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ID="Grid1" ... EnableHeaderMenu="false">
    <Columns> ... </Columns>
    <Tools>
        <f:Tool ID="toolColumns" IconFont="_ColumnsAlt" Text="管理列" ToolTip="显示隐藏列" runat="server">
            <Listeners><f:Listener Event="click" Handler="onToolColumnsClick" /></Listeners>
        </f:Tool>
    </Tools>
</f:Grid>
<script> function onToolColumnsClick(event) { F.ui.Grid1.showColumnsMenu(this.el); } </script>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().ID("Grid1").EnableHeaderMenu(false).Columns( ... )
    .Tools(F.Tool().ID("toolColumns").IconFont(IconFont._ColumnsAlt).Text("管理列").ToolTip("显示隐藏列").TabIndex(0)
        .Listener("click", "onToolColumnsClick")))
```

> 相关 API：`grid.hideColumn(event, column, true/false)`、列 `hideable` / `hidden`、`tool.setMenu(menu)`。

---

## 5. 表头提示与列自定义属性

**表头提示**（列级）：

```javascript
// F.js（注意 F.js 是 headerTooltip，小写 t）
{ text: '所学专业', field: 'Major', headerTooltip: '这是所学专业列', headerTooltipPosition: 'top' }
```
```aspx
<f:RenderField DataField="Major" HeaderText="所学专业" HeaderToolTip="这是所学专业列" HeaderToolTipPosition="Top" />
```

**列自定义 HTML 属性**（给列头挂 `data-*` 等，三栈写法不同）：

```javascript
// F.js —— 列 attrs
{ text: '姓名', field: 'Name', attrs: { "data-header-color": "color1" } }
```
```csharp
// Pro —— 后台代码（不是标签！）：FindColumn 后设 Attributes
var col = Grid1.FindColumn("Name") as FineUIPro.BoundField;
col.Attributes["data-header-color"] = "color1";
```
```csharp
// Core-MVC（Fluent）—— .Attribute(key, value)
F.RenderField().HeaderText("姓名").DataField("Name").Attribute("data-header-color", "color1")
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages）—— 嵌套 <Attributes> -->
<f:RenderField HeaderText="姓名" DataField="Name">
    <Attributes><f:Attribute Key="data-header-color" Value="color1" /></Attributes>
</f:RenderField>
```

---

## 6. 多表头（分组表头 / Group Header）

**没有 `columnType:'groupfield'` 也没有 `ColumnGroupField`**——分组靠**嵌套列**：F.js 普通列里嵌 `columns: []`（可多层，分组节点 `align:'center'`，只有叶子有 `field`）；.NET 用 **`<f:GroupField>`** 包一个嵌套 `<Columns>`。

```javascript
// F.js —— 三级嵌套（省/市/数据）
columns: [
    { text: '统计年份', field: 'year' },
    { text: '安徽省', align: 'center', columns: [
        { text: '合肥市', align: 'center', columns: [
            { text: '数据一', field: 'hefei_data1' },
            { text: '数据二', field: 'hefei_data2' }
        ] }
    ] }
]
```
```aspx
<%-- Pro / Core-TagHelper —— GroupField 嵌套（叶子 Core 用 RenderField；Pro 也可用 BoundField）--%>
<Columns>
    <f:RenderField ColumnID="Year" DataField="Year" HeaderText="统计年份" />
    <f:GroupField ColumnID="Anhui" HeaderText="安徽省" TextAlign="Center">
        <Columns>
            <f:GroupField ColumnID="Hefei" HeaderText="合肥市" TextAlign="Center">
                <Columns>
                    <f:RenderField DataField="AHData1" HeaderText="数据一" />
                    <f:RenderField DataField="AHData2" HeaderText="数据二" />
                </Columns>
            </f:GroupField>
        </Columns>
    </f:GroupField>
</Columns>
```
```csharp
// Core-MVC（Fluent）
.Columns(
    F.RenderField().HeaderText("统计年份").DataField("Year"),
    F.GroupField().HeaderText("安徽省").TextAlign(TextAlign.Center).Columns(
        F.GroupField().HeaderText("合肥市").TextAlign(TextAlign.Center).Columns(
            F.RenderField().HeaderText("数据一").DataField("AHData1"),
            F.RenderField().HeaderText("数据二").DataField("AHData2"))))
```

- **排序**：在 Grid 上开 `AllowSorting`（F.js 列 `sortable:true`），叶子列设 `SortField`；分组节点本身不排序。
- **初始隐藏某分组**：分组节点 `Hidden="true"`（F.js `hidden:true`）；运行时 `F.ui.grid1.getColumn('anhui').toggleVisible()`。

---

## 7. 动态创建列（运行时/服务端建列）

### F.js —— `configColumns()` 运行时重配

```javascript
// 初始 columns: createGrid1Columns()；之后重配（可传含嵌套 columns 的多表头数组）
F.ui.grid1.configColumns(createGrid2Columns(), { idField: 'Id', checkboxSelect: true, rowExpander: true });
```

### .NET —— C# 循环 new 列对象加入 `Grid.Columns`

| 栈 | 建列位置 | 列类型 | 挂载方式 |
|----|----------|--------|----------|
| **Pro** | `Page_Init`（**不能放 `Page_Load`**，回发时不支持动态建列） | `new FineUIPro.BoundField()` / `CheckBoxField` | `Grid1.Columns.Add(col)` |
| **Core-MVC** | Controller `Index()` | `new RenderField()` / `RenderCheckField` / `RowNumberField` | `ViewBag.Grid1Columns = list.ToArray()` → View `.Columns(ViewBag.Grid1Columns)` |
| **Core-RazorPages** | `OnGet()` | 同 MVC | `ViewBag.Grid1Columns` → 标签 `Columns="@ViewBag.Grid1Columns"` |
| **Core-RazorForms** | `Page_Load`（`!IsPostBack`） | 同 MVC | `Grid1.Columns.Clear(); ...Add(col); Grid1.DataBind();` |

```csharp
// Pro —— 必须在 Page_Init
protected void Page_Init(object sender, EventArgs e) {
    var bf = new FineUIPro.BoundField { DataField = "Name", HeaderText = "姓名" };
    Grid1.Columns.Add(bf);
    var cf = new CheckBoxField { DataField = "AtSchool", HeaderText = "是否在校" };
    Grid1.Columns.Add(cf);
    Grid1.DataKeyNames = new string[] { "Id", "Name" };
}
```
```csharp
// Core-MVC / RazorPages —— 建 List<GridColumn> 交给 ViewBag
List<GridColumn> columns = new List<GridColumn>();
columns.Add(new RowNumberField());
columns.Add(new RenderField { HeaderText = "姓名", DataField = "Name" });
columns.Add(new RenderCheckField { HeaderText = "是否在校", DataField = "AtSchool", RenderAsStaticField = true });
ViewBag.Grid1Columns = columns.ToArray();   // View: .Columns(ViewBag.Grid1Columns) / Columns="@ViewBag.Grid1Columns"
```

标签里 `<Columns>` 留空。**Pro 只支持首次初始化建列、不支持回发动态建列**。

---

## See also

- [columns.md](columns.md)：普通列定义、列类型、格式化、渲染
- [sorting.md](sorting.md)：列排序（多表头叶子列的 `SortField`）
- [filter.md](filter.md)：列过滤（`EnableFilter`）
