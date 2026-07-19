# Grid 行相关功能（扩展列 / 窗口列 / 行命令 / 行事件 / 行样式 / 行高）

围绕"行"的一组能力。**保真重点：C# 事件参数类型按栈不同**（同一个行事件，Pro 与 RazorForms 的 `EventArgs` 类名不一样），别照抄错。

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC / TagHelper |
|------|------|----------------|----------------------|
| 行扩展列 | Grid `rowExpander: { field, renderer }` | `<f:TemplateField RenderAsRowExpander="true">` | `RenderField` + `RenderAsRowExpander="true"` + `RendererFunction` |
| 展开全部扩展列 | `grid.expandRowExpanders()` | `ExpandAllRowExpanders="true"` | 同 Pro 属性 |
| 弹窗列 | 列 `renderer` + `F.ui.Window1.show(...)` | 专用 `<f:WindowField>` | `RendererFunction` + `F.ui.Window1.show` / RF 用 `<f:Command WindowID=...>` |
| 行命令列 | 列 `renderer` 返 `<a class>` | `<f:LinkButtonField CommandName>` | `RendererFunction` / RF 用 `<f:Command CommandName>` |
| 行单击/双击/选中事件 | listener `rowclick`/`rowdblclick`/`rowselect` | `EnableRowClickEvent`+`OnRowClick` 等 | 客户端 listener + 回发 / RF 服务端 `OnRowClick` |
| 整行样式 | `rowRenderer` / `rowDataBound` | 服务端 `OnRowDataBound` → `e.RowCssClass` | 客户端 `RowRendererFunction` / `RowDataBoundFunction` |
| 固定行高 / 行高行数 | `fixedRowHeight` / `rowHeightLines` | `FixedRowHeight` / `RowHeightLines` | `.FixedRowHeight()` / `.RowHeightLines()` |

---

## 1. 行扩展列（RowExpander，点击展开一块自定义内容）

```javascript
// F.js —— Grid 级 rowExpander，renderer 返回 HTML
rowExpander: {
    field: 'Desc',
    renderer: function (value, params) {
        return '<p>姓名：' + params.rowValue['Name'] + '</p><p>简介：' + value + '</p>';
    }
}
// 展开/收起全部：grid.expandRowExpanders() / grid.collapseRowExpanders()
```
```aspx
<%-- Pro —— 用 TemplateField 服务端模板 --%>
<f:Grid ... ExpandAllRowExpanders="true">
    <Columns>
        <f:TemplateField RenderAsRowExpander="true">
            <ItemTemplate><strong>姓名：</strong><%# Eval("Name") %> <strong>简介：</strong><%# Eval("Desc") %></ItemTemplate>
        </f:TemplateField>
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC / TagHelper —— RenderField + 客户端渲染函数
F.RenderField().RenderAsRowExpander(true).RendererFunction("renderExpander")   // MVC
// TagHelper: <f:RenderField RenderAsRowExpander="true" RendererFunction="renderExpander" />
```

> Pro 用服务端 `<ItemTemplate>`；F.js/Core 用客户端 `renderer`/`RendererFunction` 返回 HTML。**行扩展列与单元格合并互斥**（见 [advanced.md](advanced.md)）。

---

## 2. 弹出窗体列（点击行内链接开 iframe 窗口）

机制各栈差别大：**Pro 有专用 `<f:WindowField>` 声明式**；**RazorForms 用 `<f:Command WindowID=...>`**；**F.js / MVC / RazorPages 手写 `renderer` + `F.ui.Window1.show(url, title)`**。

```aspx
<%-- Pro —— WindowField 声明式绑定 iframe 地址与标题 --%>
<f:WindowField ColumnID="myWindowField" WindowID="Window1" HeaderText="窗口列" Text="编辑"
    DataIFrameUrlFields="Id,Name" DataIFrameUrlFormatString="grid_iframe_window.aspx?id={0}&name={1}"
    DataWindowTitleField="Name" DataWindowTitleFormatString="编辑 - {0}" />
<f:Window ID="Window1" runat="server" EnableIFrame="true" IsModal="true" CloseAction="HidePostBack" />
```
```html
<!-- Core-RazorForms —— 命令列 + Window 绑定 -->
<f:RenderField HeaderText="操作">
    <Commands>
        <f:Command CommandName="Edit" Text="编辑" WindowID="Window1"
            WindowIFrameUrlFormatString="~/Grid/IFrameWindow?id={0}&name={1}" _WindowIFrameUrlFields="Id,Name"
            WindowTitleFormatString="编辑 - {0}" _WindowTitleFields="Name" />
    </Commands>
</f:RenderField>
```
```javascript
// F.js / Core-MVC / RazorPages —— renderer 返回带 class 的链接，点击调 Window.show
{ text: '窗口列', field: 'Id', renderer: function (v, params) { return '<a class="mywindowfield">编辑</a>'; } }
// 页面 JS：$(grid.el).on('click', 'a.mywindowfield', function(){ F.ui.Window1.show(url, title); });
```

**双击行打开**：Pro `EnableRowDoubleClickEvent="true" OnRowDoubleClick="Grid1_RowDoubleClick"` →

```csharp
// Pro —— 注意 GridRowClickEventArgs
protected void Grid1_RowDoubleClick(object sender, GridRowClickEventArgs e) {
    object[] keys = Grid1.DataKeys[e.RowIndex];
    PageContext.RegisterStartupScript(GetEditUrl(keys[0], keys[1]));
}
// RazorForms 同名事件但参数是 GridRowEventArgs：Grid1_RowDoubleClick(object sender, GridRowEventArgs e)
```

---

## 3. 行内命令按钮（RowCommand）

行里放"编辑/删除"等按钮，点击回发命令。**Pro/RazorForms 有真正的服务端命令事件；MVC/RazorPages 基础版是纯客户端，服务端交互走 `F.doPostBack` 自定义参数。**

```aspx
<%-- Pro —— LinkButtonField + OnRowCommand --%>
<f:Grid ... OnRowCommand="Grid1_RowCommand">
    <Columns>
        <f:LinkButtonField CommandName="Action1" Text="按钮" Width="60px" />
        <f:LinkButtonField CommandName="Action3" IconFont="_Close" ConfirmText="确定要删除本行？" ConfirmTarget="Top" />
    </Columns>
</f:Grid>
```
```csharp
// Pro —— GridCommandEventArgs（读 e.CommandName / e.RowIndex / e.ColumnIndex）
protected void Grid1_RowCommand(object sender, GridCommandEventArgs e) {
    if (e.CommandName == "Action1" || e.CommandName == "Action3") {
        object[] keys = Grid1.DataKeys[e.RowIndex];
        ShowNotify($"第 {e.RowIndex + 1} 行，命令 {e.CommandName}，ID {keys[0]}");
    }
}
```
```html
<!-- Core-RazorForms —— <Commands> 声明式命令（一列可多个 <f:Command>）+ 服务端 OnRowCommand -->
<f:Grid ... OnRowCommand="Grid1_RowCommand">
    <Columns>
        <f:RenderField HeaderText="操作">
            <Commands>
                <f:Command CommandName="Action1" Text="编辑" />
                <f:Command CommandName="Action3" IconFont="_Close" ConfirmText="确定删除？" ConfirmTarget="Top" />
            </Commands>
        </f:RenderField>
    </Columns>
</f:Grid>
```
```csharp
// Core-RazorForms —— 注意 GridRowCommandEventArgs（与 Pro 的 GridCommandEventArgs 不同名！）
protected void Grid1_RowCommand(object sender, GridRowCommandEventArgs e) {
    object[] keys = Grid1.DataKeys[e.RowIndex];
    ShowNotify($"命令 {e.CommandName}，第 {e.RowIndex + 1} 行");
}
```
```csharp
// Core-MVC —— 客户端渲染按钮 + F.doPostBack 回发自定义参数（非 OnRowCommand）
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_RowCommand(string rowId, string rowText, int rowIndex, int columnIndex) {
    ShowNotify($"第 {rowIndex + 1} 行，ID {rowId}");
    return UIHelper.Result();
}
// RazorPages: public IActionResult OnPostGrid1_RowCommand(string rowId, string rowText, int rowIndex, int columnIndex)
```

客户端 `rowcommand` listener 签名：`function onGrid1RowCommand(event, rowId, rowIndex, columnId, commandName)`。

---

## 4. 行事件（单击 / 双击 / 选中）

```javascript
// F.js
listeners: {
    rowclick:    function (event, rowId) { var d = this.getRowData(rowId); },
    rowdblclick: function (event, rowId) { },
    rowselect:   function (event, rowId) { }
}
```

**C# 事件参数类型按栈不同（重点）：**

```csharp
// Pro —— 启用属性 + 三种不同 EventArgs
// 标签：EnableRowClickEvent="true" OnRowClick="Grid1_RowClick"
protected void Grid1_RowClick(object sender, GridRowClickEventArgs e)   { object[] k = Grid1.DataKeys[e.RowIndex]; }
protected void Grid1_RowSelect(object sender, GridRowSelectEventArgs e) { }   // EnableRowSelectEvent
protected void Grid1_RowDeselect(object sender, GridRowSelectEventArgs e) { } // EnableRowDeselectEvent
```
```csharp
// Core-RazorForms —— 服务端属性 OnRowClick，统一用 GridRowEventArgs
protected void Grid1_RowClick(object sender, GridRowEventArgs e)  { object[] k = Grid1.DataKeys[e.RowIndex]; }
protected void Grid1_RowSelect(object sender, GridRowEventArgs e) { }
```
```csharp
// Core-MVC —— 客户端 .Listener("rowclick","onGrid1RowClick") + F.doPostBack 回发
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_RowClick(string rowId, string rowText, int rowIndex, string columnText) { return UIHelper.Result(); }
public IActionResult Grid1_RowSelect(string rowId, string rowText, int rowIndex, string columnText, bool isDeselect) { return UIHelper.Result(); }
// RazorPages: OnPostGrid1_RowClick(string rowId, string rowText, int rowIndex, string columnText)
```

---

## 5. 行与单元格样式

**整行样式**——F.js/Core 客户端；Pro/RazorForms 亦可服务端：

```javascript
// F.js / Core（RowRendererFunction / RowDataBoundFunction 指向的 JS）
rowRenderer:  function (params)  { if (params.rowData.values['EntranceYear'] === 2008) params.rowCls = 'color1'; }
rowDataBound: function (rowData) { if (rowData.values['EntranceYear'] === 2008) rowData.cls = 'color1'; }
```
```csharp
// Pro —— 服务端 OnRowDataBound，GridRowEventArgs
protected void Grid1_RowDataBound(object sender, GridRowEventArgs e) {
    var row = e.DataItem as DataRowView;
    if ((int)row["EntranceYear"] == 2008) e.RowCssClass = "color1";       // 整行样式
    e.SetCellCssClass("cGender", "color1");                              // 单元格样式
    // e.RowAttributes["data-color"] = "color1"; e.SetCellAttribute("cGender","data-color","x");
}
// RazorForms 服务端同名事件，但参数是 GridRowDataBoundEventArgs（不是 GridRowEventArgs）：
//   protected void Grid1_RowDataBound(object sender, GridRowDataBoundEventArgs e) { e.RowCssClass = "color1"; }
```

**单元格样式**（客户端）：在列 `renderer` 内 `params.cellCls = 'special'` / `params.cellAttrs['data-color'] = 'x'`。纯 CSS 命中：`.f-grid-cell-<字段/ColumnID>`。

> MVC/RazorPages 的行样式示例是**客户端** `RowDataBoundFunction`；服务端 `OnRowDataBound` 行样式只有 Pro 与 RazorForms 演示。

---

## 6. 行密度与行高

**行密度**（无静态属性，用 Tool + MenuCheckBox + `setRowDensity`）：

```javascript
// 密度菜单项 click（各栈一致）；取值 small / normal / large / xlarge
function onRowDensityChange(event) {
    F.ui.Grid1.setRowDensity(this.el.attr('data-tag'));
}
```

**固定行高 / 多行行高**：

```javascript
// F.js
{ type: 'Grid', fixedRowHeight: true, rowHeightLines: 3, ... }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... FixedRowHeight="true" RowHeightLines="3"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().FixedRowHeight(true).RowHeightLines(3) ...)
```

> Pro 另有像素级行高属性（`RowHeight` / `RowHeightCompact` / `RowHeightSmall` / `RowHeightLarge` / `RowHeightLargeSpace`），Core/F.js 示例未用。

---

## 关键约束

1. **C# 事件 EventArgs 按栈不同**：行单击 Pro `GridRowClickEventArgs` / RazorForms `GridRowEventArgs`；行选中 Pro `GridRowSelectEventArgs` / RazorForms `GridRowEventArgs`；行命令 Pro `GridCommandEventArgs` / RazorForms `GridRowCommandEventArgs`；行数据绑定 Pro `GridRowEventArgs` / RazorForms `GridRowDataBoundEventArgs`。别照抄错类名。
2. **行命令服务端仅 Pro / RazorForms**：MVC/RazorPages 基础版纯客户端，服务端走 `F.doPostBack` 自定义参数（非 `OnRowCommand`）。
3. **弹窗列三套机制**：Pro `<f:WindowField>`；RazorForms `<f:Command WindowID=...>`；F.js/MVC/RazorPages 手写 `renderer` + `F.ui.Window1.show`。
4. **行扩展列与单元格合并互斥**（见 [advanced.md](advanced.md)）。
5. **行样式服务端事件仅 Pro / RazorForms**；MVC/RazorPages 用客户端 `RowDataBoundFunction`。

## See also

- [selection.md](selection.md)：行选择（`rowselect` 与选中读取）
- [columns.md](columns.md)：列渲染 `RendererFunction`（弹窗列/命令列/单元格样式都基于它）
- [advanced.md](advanced.md)：单元格合并（与行扩展互斥）、大数据（固定行高）
- `fineui-window`：弹窗列用到的 Window 组件
