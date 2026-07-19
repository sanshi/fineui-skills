# Grid 合计行（Summary Row）

表格底部（或顶部）的汇总行：某列求和/平均，或自定义渲染。分**客户端合计**（前端按已加载行算）与**服务端合计**（C# 算好数字塞进去）；分**当前页合计**与**全部合计**。

> 这里是"整表页脚合计"。**分组内的小计**是另一套 `RowGroup*` 属性，见 [row-group.md](row-group.md)。

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent) | Core-TagHelper |
|------|------|----------------|-------------------|----------------|
| 开启合计 | `summary: true` | `EnableSummary="true"` | `.EnableSummary(true)` | `EnableSummary="true"` |
| 位置 | `summaryPosition:'bottom'` | `SummaryPosition="Bottom"` | `.SummaryPosition(SummaryPosition.Bottom)` | `SummaryPosition="Bottom"` |
| 列合计类型 | 列 `summaryType:'sum'` | `SummaryType="Sum"` | `.SummaryType(SummaryType.Sum)` | `SummaryType="Sum"` |
| 合计格式参数 | 列 `summaryTypeArgument:'N2'` | `SummaryTypeArgument="N2"` | `.SummaryTypeArgument("N2")` | `SummaryTypeArgument="N2"` |
| 标签文本单元格 | 列 `summaryText:'合计：'` | `SummaryText="合计："` | `.SummaryText("合计：")` | `SummaryText="合计："` |
| 自定义渲染 | 列 `summaryRenderer: fn` | `SummaryRendererFunction="fn名"` | `.SummaryRendererFunction("fn名")` | `SummaryRendererFunction="fn名"` |
| 服务端合计（1 行） | 服务端数据 | `Grid1.SummaryData = JObject` | `.SummaryData(ViewBag...)` | `SummaryData="@ViewBag..."` |
| 服务端合计（多行） | 服务端数据 | `Grid1.SummaryDataArray = JArray` | `.SummaryDataArray(ViewBag...)` | `ViewBag...` |
| 多行合计行数 | `summaryRowCount: 2` | `SummaryRowCount="2"` | `.SummaryRowCount(2)` | `SummaryRowCount="2"` |

`SummaryPosition` 取值：`Flow`（随行滚动）/ `Top`（浮动顶部）/ `Bottom`（浮动底部）。

---

## 1. 客户端合计（前端按已加载行算）

开 `EnableSummary`，在数值列上设 `SummaryType`（`Sum`/`Avg`），**不提供任何合计数据**，前端自动按当前行计算。

```javascript
// F.js
{ type: 'Grid', summary: true, summaryPosition: 'bottom',
  columns: [
      { text: '所学专业', field: 'Major', summaryText: '合计：' },
      { text: '学费', field: 'Fee', fieldType: 'int', summaryType: 'sum' },
      { text: '学杂费', field: 'ExtraFee', fieldType: 'int', summaryType: 'sum' }
  ] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... EnableSummary="true" SummaryPosition="Bottom">
    <Columns>
        <f:RenderField ColumnID="Major" DataField="Major" HeaderText="所学专业" SummaryText="合计：" />
        <f:RenderField ColumnID="Fee" DataField="Fee" FieldType="Int" HeaderText="学费" SummaryType="Sum" />
        <f:RenderField ColumnID="ExtraFee" DataField="ExtraFee" FieldType="Int" HeaderText="学杂费" SummaryType="Sum" />
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC（Fluent）—— 平均 + 格式化保留 2 位
F.RenderField().HeaderText("学费").DataField("Fee").FieldType(FieldType.Float).SummaryType(SummaryType.Avg).SummaryTypeArgument("N2")
```

> 声明式 `SummaryType` 示例中只见 `Sum` / `Avg`。**`min`/`max`/`count` 没有声明式列类型**，要用它们得在自定义 `summaryRenderer` 里调 `grid.calcSummaryValue('Fee','min'|'max'|'avg','N2')`（见 §3）。

---

## 2. 服务端合计（C# 算好塞进去）

服务端计算好每列的合计值，赋给 `SummaryData`（单行 `JObject`）或 `SummaryDataArray`（多行 `JArray`）。

```csharp
// Pro / RazorForms —— 单行合计（JObject，key = 列的 DataField）
JObject summary = new JObject();
summary.Add("Fee", feeTotal.ToString("F2"));
summary.Add("ExtraFee", extraFeeTotal.ToString("F2"));
Grid1.SummaryData = summary;
```
```csharp
// Core-MVC / RazorPages —— 通过 ViewBag 传入，View 里 .SummaryData(ViewBag.Grid1SummaryData)
// Controller: ViewBag.Grid1SummaryData = GetSummaryData(GetDataTable2());
```

**当前页合计 vs 全部合计**：
- **全部合计**：对全量数据算一次（不随翻页变）。
- **当前页合计**：在翻页处理器里**按当前页数据重算** `SummaryData`。

```csharp
// Core-MVC —— 当前页合计：翻页时重算
public IActionResult Grid1_PageIndexChanged(string[] Grid1_fields, int Grid1_pageIndex) {
    var grid1 = UIHelper.Grid("Grid1");
    grid1.RecordCount(DataSourceUtil.GetTotalCount());
    var dataSource = DataSourceUtil.GetPagedDataTable(Grid1_pageIndex, 5);
    grid1.DataSource(dataSource, Grid1_fields);
    grid1.SummaryData(GetSummaryData(dataSource));   // 用当前页数据重算
    return UIHelper.Result();
}
```

---

## 3. 多行合计 + 自定义渲染

`SummaryRowCount="3"` 出 3 行合计；每列用 `SummaryRendererFunction` 指向 JS 函数，签名 `(summaryRowIndex, cellValue, params)`，值用 `grid.calcSummaryValue(field, 'min'|'max'|'avg'|'sum', 格式)` 算。

```javascript
// F.js —— 3 行分别显示 最小/最大/平均
{ type: 'Grid', summary: true, summaryPosition: 'bottom', summaryRowCount: 3,
  columns: [{ text: '学费', field: 'Xuefei', fieldType: 'int', summaryRenderer: xuefeiSummaryRenderer }] }

function xuefeiSummaryRenderer(summaryRowIndex) {
    var grid1 = this;
    if (summaryRowIndex == 0) return grid1.calcSummaryValue('Xuefei', 'min');
    if (summaryRowIndex == 1) return grid1.calcSummaryValue('Xuefei', 'max');
    return grid1.calcSummaryValue('Xuefei', 'avg', 'N2');
}
```
```csharp
// Core-MVC —— 多行 + 服务端多行数据
@(F.Grid().EnableSummary(true).SummaryPosition(SummaryPosition.Bottom).SummaryRowCount(2)
    .Columns(F.RenderField().HeaderText("学费").DataField("Fee").SummaryRendererFunction("feeSummaryRenderer"))
    .SummaryDataArray(ViewBag.Grid1SummaryDataArray))
```
```csharp
// Pro —— 多行服务端数据（JArray，每行一个 JObject）
JArray summaryArray = new JArray();
summaryArray.Add(CalcSummaryRow(currentPageTable, "当前页合计："));
summaryArray.Add(CalcSummaryRow(DataSourceUtil.GetDataTable2(), "全部合计："));
Grid1.SummaryDataArray = summaryArray;
```

> 合计行的多行样式（不同行不同底色）靠 CSS 行选择器 `.f-grid-row-summary:first-child` / `:nth-child(2)` 与单元格 `params.cellCls`。

---

## 关键约束

1. **客户端 vs 服务端**：设了列 `SummaryType` 且不给合计数据 = 客户端算；给 `SummaryData`/`SummaryDataArray` = 服务端算。
2. **声明式只有 Sum/Avg**：`min`/`max`/`count` 走 `summaryRenderer` + `calcSummaryValue`。
3. **当前页合计要在翻页处理器重算** `SummaryData`；全部合计算一次即可。
4. **浮动位置** `SummaryPosition`：`Top`/`Bottom` 浮动固定，`Flow` 随行滚动。
5. **整表合计 ≠ 分组小计**：分组内小计是 `RowGroupSummary*`（[row-group.md](row-group.md)）。

## See also

- [data-loading.md](data-loading.md)：分页（当前页合计要配合翻页事件）
- [row-group.md](row-group.md)：分组内小计
