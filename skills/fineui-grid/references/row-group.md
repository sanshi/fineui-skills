# Grid 行分组（Row Grouping）

按某字段把数据行分组，每组一个分组头（可折叠），并可给每组算合计。**注意与"多表头"（列的分组，见 [header.md](header.md)）、"树表格"（父子层级，见 [tree-grid.md](tree-grid.md)）是三回事。**

> 结构差异：**F.js 把分组选项嵌在 `rowGroup: {}` 对象里**；**四个 .NET 栈把每个选项拍平成 `RowGroup*` / `EnableRowGroup` 独立属性**。

## 概念 → 各写法属性名对照

| 概念 | F.js | .NET（Pro / Core-MVC / Core-TagHelper） |
|------|------|------------------------------------------|
| 开启行分组 | `rowGroupField: 'X'`（+`rowGroup:{collapsible:true}`） | `EnableRowGroup="true"` + `DataRowGroupField="X"` |
| 分组头渲染 | `rowGroup: { renderer: fn }` | `RowGroupRendererFunction="fn名"` |
| 初始全部折叠 | `rowGroup: { expanded: false }` | `ExpandAllRowGroups="false"` |
| 开启分组合计 | `rowGroup: { summary: true }` | `RowGroupSummary="true"` |
| 列的分组合计类型 | 列 `rowGroupSummaryType: 'avg'` | 列 `RowGroupSummaryType="Avg"` |
| 列的分组合计文本 | 列 `rowGroupSummaryText: '...'` | 列 `RowGroupSummaryText="..."` |
| 总合计（配合分组） | Grid `summary: true` | `EnableSummary="true"` |

---

## 1. 开启行分组

```javascript
// F.js —— rowGroupField 指定分组字段，rowGroup 放分组选项
F.create({ type: 'Grid', id: 'grid1', isFluid: true, renderTo: '#wrap',
    rowGroupField: 'EntranceYear',
    rowGroup: { collapsible: true },     // 可折叠
    columns: [ { text: '姓名', field: 'Name' }, { text: '所学专业', field: 'Major' } ],
    idField: 'Id', data: rows });
```
```aspx
<%-- Pro / Core-TagHelper —— 拍平成两个属性 --%>
<f:Grid ID="Grid1" runat="server" EnableRowGroup="true" DataRowGroupField="EntranceYear"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().ID("Grid1").EnableRowGroup(true).DataRowGroupField("EntranceYear").Columns( ... ).DataSource(ViewBag.Grid1DataSource))
```

---

## 2. 分组头文字渲染

F.js 传内联函数；.NET 栈用 `RowGroupRendererFunction` 指向页面里的 JS 函数。签名 `(groupValue, rowData)`，`rowData.children` 是该组的行。

```javascript
// F.js
rowGroup: {
    collapsible: true,
    renderer: function (groupValue, rowData) {
        var male = rowData.children.filter(function (r) { return r.Gender == 1; }).length;
        return F.formatString('入学年份：{0}，男：{1}，女：{2}', groupValue, male, rowData.children.length - male);
    }
}
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... EnableRowGroup="true" DataRowGroupField="EntranceYear" RowGroupRendererFunction="onGrid1RowGroupRenderer"> ... </f:Grid>
<script> function onGrid1RowGroupRenderer(groupValue, rowData) { /* 同 F.js renderer */ } </script>
```

---

## 3. 初始展开/折叠

```javascript
// F.js —— 全部折叠
rowGroup: { collapsible: true, expanded: false }
```
```aspx
<%-- Pro / Core-TagHelper —— 全部折叠 --%>
<f:Grid ... EnableRowGroup="true" DataRowGroupField="EntranceYear" ExpandAllRowGroups="false"> ... </f:Grid>
```

**只折叠某些组**：用 `rowDataBound`（.NET `RowDataBoundFunction`）在分组行上按值设 `expanded`：

```javascript
function onGrid1RowDataBound(rowData) {
    if (rowData.isRowGroup && (rowData.rowGroup === '2000' || rowData.rowGroup === '2008')) {
        rowData.expanded = false;
    }
}
```

---

## 4. 分组合计（组内小计）

开 `RowGroupSummary`，在每个要合计的列上设 `RowGroupSummaryType`（`Sum`/`Avg`/`Count`/`Max`/`Min`）；标签文本列设 `RowGroupSummaryText`。

```javascript
// F.js
rowGroup: { collapsible: true, summary: true },
columns: [
    { text: '所学专业', field: 'Major', rowGroupSummaryText: '平均（分组）：' },
    { text: '语文成绩', field: 'ChineseScore', fieldType: 'int', rowGroupSummaryType: 'avg' },
    { text: '数学成绩', field: 'MathScore', fieldType: 'int', rowGroupSummaryType: 'avg' }
]
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... EnableRowGroup="true" DataRowGroupField="EntranceYear" RowGroupSummary="true">
    <Columns>
        <f:RenderField DataField="Major" ColumnID="Major" HeaderText="所学专业" RowGroupSummaryText="平均（分组）：" />
        <f:RenderField DataField="ChineseScore" ColumnID="ChineseScore" FieldType="Int" HeaderText="语文成绩" RowGroupSummaryType="Avg" />
        <f:RenderField DataField="MathScore" ColumnID="MathScore" FieldType="Int" HeaderText="数学成绩" RowGroupSummaryType="Avg" />
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC（Fluent）
F.RenderField().DataField("ChineseScore").ColumnID("ChineseScore").FieldType(FieldType.Int).HeaderText("语文成绩").RowGroupSummaryType(SummaryType.Avg)
```

**总合计（页脚总计，与分组合计并用）**：Grid 开 `EnableSummary="true"` + `SummaryPosition="Bottom"`，列上用 `SummaryType` / `SummaryText`（注意这套是"总合计"属性，与 `RowGroup*` 那套并列，见 [summary.md](summary.md)）。

**多行合计**：`RowGroupSummaryRowCount="3"`（F.js `rowGroup:{ summaryRowCount:3 }`）+ 列 `RowGroupSummaryRendererFunction`，渲染签名 `(summaryRowIndex, cellValue, params)`，值用 `grid.calcSummaryValue('ChineseScore', 'min', params.rowGroupData)` 算。

**隐藏某些组的合计**（如单行组）：用 `RowRendererFunction`，`if (params.rowData.isRowGroupSummary) params.rowCls = 'f-hidden';`。

---

## 5. 分组 + 排序 / 数据库分页

**没有分组专用排序属性**——直接叠加 Grid 常规的排序/分页属性，返回的排序数据会重新分组。

```javascript
// F.js —— 数据库分页 + 服务端排序 + 行分组
paging: true, databasePaging: true, pageSize: 10,
sorting: true, databaseSorting: true, sortField: 'Name', sortDirection: 'ASC',
rowGroupField: 'EntranceYear', rowGroup: { collapsible: true }
```
```aspx
<%-- Core-TagHelper（RazorForms/RazorPages）--%>
<f:Grid ... AllowPaging="true" PageSize="10" IsDatabasePaging="true" OnPageIndexChanged="Grid1_PageIndexChanged"
        AllowSorting="true" SortField="Name" SortDirection="ASC" OnSort="Grid1_Sort"
        EnableRowGroup="true" DataRowGroupField="EntranceYear"> ... </f:Grid>
```

> 排序/分页事件的 C# 处理见 [sorting.md](sorting.md) 与 [data-loading.md](data-loading.md)（分组不改变这套回发范式）。

---

## 关键约束

1. **F.js 嵌套 vs .NET 拍平**：F.js 所有分组选项在 `rowGroup:{}` 内（`collapsible`/`expanded`/`summary`/`summaryRowCount`/`renderer`）；.NET 是一堆 `RowGroup*` / `EnableRowGroup` / `ExpandAllRowGroups` 独立属性。
2. **两套合计属性别混**：组内小计 `RowGroupSummary*`；页脚总计 `EnableSummary` + `Summary*`（见 [summary.md](summary.md)）。
3. **渲染函数**：F.js 内联函数；.NET 用 `*RendererFunction` 字符串指向页面 JS 函数。
4. **RazorForms 与 RazorPages 的 `.cshtml` 几乎逐字节相同**（仅 `@model` 与后台命名空间不同）。

## See also

- [summary.md](summary.md)：页脚总合计行（当前页/全部、浮动、多行）
- [header.md](header.md)：多表头（列分组，与行分组不同）
- [tree-grid.md](tree-grid.md)：树表格（父子层级，与行分组不同）
