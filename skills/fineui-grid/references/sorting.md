# Grid 排序（Sorting）

客户端排序（前端按已加载数据排）与服务端排序（回发按字段重查）。多列排序、按别的字段排、自定义排序函数、初始排序、取消排序。

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent) | Core-TagHelper | Java（Thymeleaf 方言） |
|------|------|----------------|-------------------|----------------|------------------------|
| 开启排序（Grid） | `sorting: true` | `AllowSorting="true"` | `.AllowSorting(true)` | `AllowSorting="true"` | `allow-sorting="true"` |
| 某列可排序 | 列 `sortable: true` | 列 `SortField="Name"` | `.SortField("Name")` | `SortField="Name"` | 列 `sort-field="Name"` |
| 初始排序 | `sortField`/`sortDirection` | `SortField`/`SortDirection` | `.SortField().SortDirection()` | `SortField`/`SortDirection` | `sort-field`/`sort-direction` |
| 多列排序 | `sortingMulti: true` | `SortingMulti="true"` | `.SortingMulti(true)` | `SortingMulti="true"` | `sorting-multi="true"` |
| 初始多列排序 | `sortFields: ['Gender','ASC']` | `SortFieldArray="EntranceYear,DESC,Gender,ASC"` | `.SortFieldArray(...)` | `_SortFieldArray="..."` | `sort-field-array="EntranceYear,DESC,Gender,ASC"` |
| 按别的字段排 | 列 `sortField`（≠`field`） | 列 `SortField`（≠`DataField`） | `.SortField("Gender")` | `SortField="Gender"` | 列 `sort-field="Gender"` |
| 自定义排序函数 | 列 `sorter: fn` | **无**（Pro 排序恒回发） | `.SorterFunction("fn名")` | `SorterFunction="fn名"` | 列 `sorter-function="fn名"` |
| 排序提示 | `sortingTooltip: true` | `SortingToolTip="true"` | `.SortingToolTip(true)` | `SortingToolTip="true"` | `sorting-tool-tip="true"` |
| 允许取消排序 | `sortingCancel: true` | `SortingCancel="true"` | `.SortingCancel(true)` | `SortingCancel="true"` | `sorting-cancel="true"` |
| 服务端排序 | `databaseSorting: true` | `OnSort` 服务端事件 | `.OnSort(Url.Action(...),"Grid1")` | `OnSort=...` | `on-sort="Grid1_Sort"` 服务端事件 |

---

## 1. 客户端排序（前端排已加载数据）

```javascript
// F.js —— Grid 开 sorting，列开 sortable
{ type: 'Grid', sorting: true, sortField: 'Gender', sortDirection: 'ASC',
  columns: [ { text: '姓名', field: 'Name', sortable: true }, { text: '性别', field: 'Gender', sortable: true } ] }
```
```aspx
<%-- Pro / Core-TagHelper —— 列上 SortField 即可排序 --%>
<f:Grid ... AllowSorting="true" SortField="Gender" SortDirection="ASC">
    <Columns>
        <f:RenderField DataField="Name" HeaderText="姓名" SortField="Name" />
        <f:RenderField DataField="Gender" HeaderText="性别" SortField="Gender" />
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().AllowSorting(true).SortField("Gender").SortDirection("ASC")
    .Columns(F.RenderField().HeaderText("姓名").DataField("Name").SortField("Name")) ...)
```
```html
<!-- FineUIJava（Thymeleaf 方言）：列上 sort-field 即可排序；多列加 sorting-multi + sort-field-array -->
<f:grid ... allow-sorting="true" sort-field="Gender" sort-direction="ASC">
    <f:columns>
        <f:render-field header-text="姓名" data-field="Name" sort-field="Name"></f:render-field>
        <f:render-field header-text="性别" data-field="Gender" sort-field="Gender"></f:render-field>
    </f:columns>
</f:grid>
```

**多列排序**：Grid 加 `SortingMulti="true"`，初始顺序用 `SortFieldArray="EntranceYear,DESC,Gender,ASC"`（字段, 方向, 字段, 方向…）；**Java 为 `sorting-multi="true"` + `sort-field-array="EntranceYear,DESC,Gender,ASC"`**。

**自定义排序函数**（F.js 内联 / Core `SorterFunction` 指向 JS 函数，**Pro 无此能力**）：

```javascript
// F.js / 或 Core 的 SorterFunction 指向的 JS 函数（签名相同）
sorter: function (x, y) {           // Core：SorterFunction="majorSorter"，页面里定义 function majorSorter(x, y){...}
    return x.length - y.length;     // 按字符串长度排
}
```

---

## 2. 服务端排序（回发按字段重查）

排序时回发，**读取排序字段/方向 → 按字段查询 → 重绑**。各栈事件名与读取方式不同。

### Pro（WebForms）—— `OnSort` 服务端事件，读 `Grid1.SortField`

```aspx
<f:Grid ID="Grid1" runat="server" AllowSorting="true" OnSort="Grid1_Sort" DataKeyNames="Id"> ... </f:Grid>
```
```csharp
protected void Grid1_Sort(object sender, GridSortEventArgs e) {
    // e.SortField / e.SortDirection 已自动写入 Grid1.SortField / Grid1.SortDirection
    BindGrid();   // 内部按 Grid1.SortField/SortDirection 查询后 DataBind
}
```

### Core-MVC —— `.OnSort(Url.Action(...),"Grid1")`，回发参数 `Grid1_sortField`/`Grid1_sortDirection`

```csharp
// View
@(F.Grid().ID("Grid1").AllowSorting(true).OnSort(Url.Action("Grid1_Sort"), "Grid1").Columns( ... ).DataSource(ViewBag.Grid1DataSource))
// Controller
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_Sort(string[] Grid1_fields, string Grid1_sortField, string Grid1_sortDirection) {
    UIHelper.Grid("Grid1").DataSource(GetSortedDataTable(Grid1_sortField, Grid1_sortDirection), Grid1_fields);
    return UIHelper.Result();
}
// 多列排序：参数改为 string[] Grid1_sortFields
```

### Core-RazorForms —— 后台服务端事件（读 `Grid1.SortField`）

```csharp
protected void Grid1_Sort(object sender, GridSortEventArgs e) {
    LoadData();   // 读 Grid1.SortField / Grid1.SortDirection（多列读 Grid1.SortFieldArray）重绑
}
```

### Core-RazorPages —— `OnPost` 处理器 + `OnSortFields`

```html
<f:Grid ID="Grid1" AllowSorting="true" OnSort="@Url.Handler("Grid1_Sort")" OnSortFields="Grid1"> ... </f:Grid>
```
```csharp
public IActionResult OnPostGrid1_Sort(string[] Grid1_fields, string Grid1_sortField, string Grid1_sortDirection) {
    UIHelper.Grid("Grid1").DataSource(GetSortedDataTable(Grid1_sortField, Grid1_sortDirection), Grid1_fields);
    return UIHelper.Result();
}
```

### FineUIJava —— 后台服务端事件（读 `Grid1.getSortField()`）

同 RazorForms 范式：`on-sort` 指向 `void` 处理器，读控件的排序态 getter 后重绑（多列读 `getSortFieldArray()`）。

```java
// FineUIJava 页面类
public void Grid1_Sort(Object sender, GridSortEventArgs e) {
    loadData();   // 内部读 Grid1.getSortField() / Grid1.getSortDirection()（多列 Grid1.getSortFieldArray()）后重绑
}
private void loadData() {
    Grid1.setDataSource(getSortedRows(Grid1.getSortField(), Grid1.getSortDirection()));
    Grid1.dataBind();
}
```

### F.js —— `databaseSorting: true` + `dataFilter` 取服务端已排序数据

```javascript
{ type: 'Grid', sorting: true, databaseSorting: true, dataUrl: '/api/students',
  dataFilter: function () { return DATA_SERVER.getSortedData(this.url); } }   // 真实项目由服务端返回排序结果
// 服务端分页 + 服务端排序：再加 paging:true, databasePaging:true
```

**服务端主动设排序**（按钮里）：Pro/RazorForms `Grid1.SortField=f; Grid1.SortDirection=d;`；MVC/RazorPages `UIHelper.Grid("Grid1").SortField(f, d)`（多列 `.SortFieldArray(arr)`）；**Java `Grid1.setSortField(f); Grid1.setSortDirection(d);`（多列 `Grid1.setSortFieldArray(new String[]{"EntranceYear","ASC","Name","DESC"})`）后 `loadData()` 重绑**。

---

## 关键约束

1. **Pro 排序恒服务端**：Pro 没有客户端 `sorter`/`SorterFunction`，排序总触发 `OnSort` 回发；客户端自定义排序 F.js / Core 三模式 / Java 都有（Java 列 `sorter-function="fn名"`，函数体同 F.js）。
2. **事件读取差异**：Pro/RazorForms 读控件属性 `Grid1.SortField`/`SortDirection`/`SortFieldArray`；**Java 读 getter `Grid1.getSortField()`/`getSortDirection()`/`getSortFieldArray()`**；MVC/RazorPages 读回发参数 `Grid1_sortField`/`Grid1_sortDirection`/`Grid1_sortFields`。
3. **按别的字段排**：列 `SortField` 与 `DataField` 不同即"显示 A、按 B 排"。
4. **多列**：`SortingMulti` + `SortFieldArray`（字段/方向交替）。

## See also

- [data-loading.md](data-loading.md)：服务端分页（常与服务端排序合用，`grid_paging_database_sorting`）
- [summary.md](summary.md)：合计行
- [filter.md](filter.md)：过滤（同为"回发→重查→重绑"）
