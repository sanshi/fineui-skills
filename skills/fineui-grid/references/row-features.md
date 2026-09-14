# Grid 行相关功能（扩展列 / 窗口列 / 行命令 / 行事件 / 行样式 / 行高）

围绕"行"的一组能力。**保真重点：C# 事件参数类型按栈不同**（同一个行事件，Pro 与 RazorForms 的 `EventArgs` 类名不一样），别照抄错。

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC / TagHelper | Java（Thymeleaf 方言） |
|------|------|----------------|----------------------|------------------------|
| 行扩展列 | Grid `rowExpander: { field, renderer }` | 本技能不提供 Pro 专属旧列写法 | `RenderField` + `RenderAsRowExpander="true"` + `RendererFunction` | `<f:render-field render-as-row-expander="true" renderer-function="fn">` |
| 展开全部扩展列 | `grid.expandRowExpanders()` | `ExpandAllRowExpanders="true"` | 同 Pro 属性 | `expand-all-row-expanders="true"` |
| 弹窗列 | 列 `renderer` + `F.ui.Window1.show(...)` | `RenderField.Commands` | `RendererFunction` + `F.ui.Window1.show` / RF 用 `<f:Command WindowID=...>` | `<f:command window-id="Window1" window-iframe-url-format-string=…>`（同 RazorForms） |
| 行命令列 | 列 `renderer` 返 `<a class>` | `RenderField.Commands` | `RendererFunction` / RF 用 `<f:Command CommandName>` | `<f:commands><f:command command-name=…>` |
| 行单击/双击/选中事件 | listener `rowclick`/`rowdblclick`/`rowselect` | `EnableRowClickEvent`+`OnRowClick` 等 | 客户端 listener + 回发 / RF 服务端 `OnRowClick` | `on-row-click` / `on-row-double-click` 服务端事件 |
| 整行样式 | `rowRenderer` / `rowDataBound` | 服务端 `OnRowDataBound` → `e.RowCssClass` | 客户端 `RowRendererFunction` / `RowDataBoundFunction` | 客户端 `row-renderer-function` / `row-data-bound-function`；服务端 `on-row-data-bound` |
| 固定行高 / 行高行数 | `fixedRowHeight` / `rowHeightLines` | `FixedRowHeight` / `RowHeightLines` | `.FixedRowHeight()` / `.RowHeightLines()` | `fixed-row-height` / `row-height-lines` |

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
```csharp
// Core-MVC / TagHelper —— RenderField + 客户端渲染函数
F.RenderField().RenderAsRowExpander(true).RendererFunction("renderExpander")   // MVC
// TagHelper: <f:RenderField RenderAsRowExpander="true" RendererFunction="renderExpander" />
```
```html
<!-- FineUIJava（Thymeleaf 方言）—— render-as-row-expander + renderer-function（函数体同 F.js） -->
<f:render-field data-field="Desc" render-as-row-expander="true" renderer-function="renderExpander"></f:render-field>
<!-- 展开全部：<f:grid expand-all-row-expanders="true">；客户端 F.ui.Grid1.rowExpander.toggleVisible() -->
```

> 本节只给出 F.js、Core、Java 共有的客户端渲染思路，不把 Pro 专属旧列作为新代码模板。**行扩展列与单元格合并互斥**（见 [advanced.md](advanced.md)）。

---

## 2. 弹出窗体列（点击行内链接开 iframe 窗口）

机制各栈差别大：**Pro 与 RazorForms 新代码都用 `<f:Command WindowID=...>`**；**F.js / MVC / RazorPages 手写 `renderer` + `F.ui.Window1.show(url, title)`**。

```aspx
<%-- Pro —— RenderField.Commands 声明式绑定 iframe 地址与标题 --%>
<f:RenderField ColumnID="Actions" HeaderText="操作">
    <Commands>
        <f:Command CommandName="Edit" Text="编辑" WindowID="Window1"
            WindowIFrameUrlFields="Id,Name" WindowIFrameUrlFormatString="grid_iframe_window.aspx?id={0}&amp;name={1}"
            WindowTitleFields="Name" WindowTitleFormatString="编辑 - {0}" />
    </Commands>
</f:RenderField>
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
```html
<!-- FineUIJava（Thymeleaf 方言）—— 同 RazorForms：命令列 + Window 绑定（url 字段以逗号列出，无下划线前缀） -->
<f:render-field header-text="窗口列">
    <f:commands>
        <f:command command-name="Action1" text="编辑" css-class="mywindowfield" window-id="Window1"
            window-iframe-url-format-string="/grid/iframe-window?id={0}&amp;name={1}" window-iframe-url-fields="Id,Name"
            window-title-format-string="编辑 - {0}" window-title-fields="Name"></f:command>
    </f:commands>
</f:render-field>
<f:window id="Window1" title="编辑" hidden="true" enable-iframe="true" close-action="HidePostBack" on-close="Window1_Close"
          target="Top" is-modal="true" width="850" height="500"></f:window>
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
```java
// FineUIJava —— <f:grid data-key-names="Id,Name" on-row-double-click="Grid1_RowDoubleClick">；参数 GridRowEventArgs
public void Grid1_RowDoubleClick(Object sender, GridRowEventArgs e) {
    Object[] keys = Grid1.getDataKeys().get(e.getRowIndex());   // keys[0]=Id, keys[1]=Name
    // 窗体通信在被弹页面类里用 ActiveWindow.hidePostBack() / hideRefresh() / hideCallParentFunction("removeActiveTab")
}
```

---

## 3. 行内命令按钮（RowCommand）

行里放“编辑/删除”等按钮，点击回发命令。**Pro/RazorForms/Java 使用 `RenderField.Commands` 和真正的服务端命令事件；MVC/RazorPages 基础版是纯客户端，服务端交互走明确 action/handler。**

```aspx
<%-- Pro —— RenderField.Commands + OnRowCommand（Command 只属于 RenderField） --%>
<f:Grid ... OnRowCommand="Grid1_RowCommand">
    <Columns>
        <f:RenderField ColumnID="Actions" HeaderText="操作">
            <Commands>
                <f:Command CommandName="Action1" Text="编辑" />
                <f:Command CommandName="Action3" IconFont="_Close" ConfirmText="确定要删除本行？" ConfirmTarget="Top" />
            </Commands>
        </f:RenderField>
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

```html
<!-- FineUIJava（Thymeleaf 方言）—— <f:commands> 声明式命令；服务端事件用 on-row-command -->
<f:grid ... data-key-names="Id,Name" on-row-command="Grid1_RowCommand">
    <f:columns>
        <f:render-field header-text="">
            <f:commands>
                <f:command command-name="Action1" text="编辑"></f:command>
                <f:command command-name="Action3" icon-font="Remove" confirm-text="确定删除？" confirm-target="Top"></f:command>
            </f:commands>
        </f:render-field>
    </f:columns>
</f:grid>
```
```java
// FineUIJava —— 注意：行命令参数是 GridCommandEventArgs（同 Pro，不是 RazorForms 的 GridRowCommandEventArgs）
public void Grid1_RowCommand(Object sender, GridCommandEventArgs e) {
    Object[] keys = Grid1.getDataKeys().get(e.getRowIndex());
    showNotify(String.format("第 %d 行，命令 %s，ID %s", e.getRowIndex() + 1, e.getCommandName(), keys[0]));
    // e.getColumnIndex() 取命令所在列
}
```
> 纯客户端命令（不回发）：`<f:grid>` 上 `<f:listeners><f:listener event="rowcommand" handler="onGrid1RowCommand">`，JS 签名与 F.js 相同：`function onGrid1RowCommand(event, rowId, rowIndex, columnId, commandName, commandArgument)`。

客户端 `rowcommand` listener 签名：`function onGrid1RowCommand(event, rowId, rowIndex, columnId, commandName, commandArgument)`。有服务端 `OnRowCommand` 时，监听器显式返回 `false` 可阻止后续回发。

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
```java
// FineUIJava —— 服务端属性 on-row-click（需 data-key-names），参数 GridRowEventArgs（同 RazorForms）
// 标签：<f:grid data-key-names="Id,Name" on-row-click="Grid1_RowClick">（双击 on-row-double-click，选中 on-row-select）
public void Grid1_RowClick(Object sender, GridRowEventArgs e) {
    int rowIndex = e.getRowIndex();
    Object[] keys = Grid1.getDataKeys().get(rowIndex);
    String[] selectedCell = Grid1.getSelectedCell();   // [rowId, columnId]，可取当前单元格所在列
    showNotify("单击第 " + (rowIndex + 1) + " 行，ID：" + keys[0]);
}
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
```java
// FineUIJava —— 服务端 on-row-data-bound，参数 GridRowDataBoundEventArgs（同 RazorForms）
// 标签：<f:grid on-row-data-bound="Grid1_RowDataBound">
public void Grid1_RowDataBound(Object sender, GridRowDataBoundEventArgs e) {
    int year = ((Number) e.getFieldValue("EntranceYear")).intValue();   // 读本行字段值
    if (year == 2008) e.setRowCssClass("color1");                        // 整行样式
    // 客户端方式（同 F.js）：row-renderer-function / row-data-bound-function 设 params.rowCls / rowData.cls
}
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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:grid ... fixed-row-height="true" row-height-lines="3"> ... </f:grid>
```

> 行密度菜单项 click 里 `F.ui.Grid1.setRowDensity(...)`（`small`/`normal`/`large`/`xlarge`）与 F.js 完全相同。
> Pro 另有像素级行高属性（`RowHeight` / `RowHeightCompact` / `RowHeightSmall` / `RowHeightLarge` / `RowHeightLargeSpace`），Core/F.js 示例未用。

---

## 关键约束

1. **事件 EventArgs 按栈不同**：行单击 Pro `GridRowClickEventArgs` / RazorForms 与 **Java** `GridRowEventArgs`；行选中 Pro `GridRowSelectEventArgs` / RazorForms `GridRowEventArgs`；行命令 Pro 与 **Java** `GridCommandEventArgs` / RazorForms `GridRowCommandEventArgs`；行数据绑定 Pro `GridRowEventArgs` / RazorForms 与 **Java** `GridRowDataBoundEventArgs`（getter 取值：`e.getRowIndex()`/`e.getCommandName()`/`e.getFieldValue("列")`/`e.setRowCssClass(...)`）。别照抄错类名。
2. **行命令服务端**：Pro / RazorForms / **Java** 有真正的服务端命令事件（`OnRowCommand` / `on-row-command`），新代码统一用 `RenderField.Commands`。`Command` 只属于 `RenderField`。MVC/RazorPages 基础版纯客户端，服务端走明确 action/handler（非 `OnRowCommand`）。
3. **弹窗列多套机制**：Pro、RazorForms 与 **Java** 使用 `<f:Command WindowID=…>` / `<f:command window-id=…>`；F.js/MVC/RazorPages 手写 `renderer` + `F.ui.Window1.show`。
4. **行扩展列与单元格合并互斥**（见 [advanced.md](advanced.md)）。
5. **行样式服务端事件仅 Pro / RazorForms / Java**（`on-row-data-bound` → `GridRowDataBoundEventArgs`）；MVC/RazorPages 用客户端 `RowDataBoundFunction`。

## See also

- [selection.md](selection.md)：行选择（`rowselect` 与选中读取）
- [columns.md](columns.md)：列渲染 `RendererFunction`（弹窗列/命令列/单元格样式都基于它）
- [advanced.md](advanced.md)：单元格合并（与行扩展互斥）、大数据（固定行高）
- `fineui-window`：弹窗列用到的 Window 组件
