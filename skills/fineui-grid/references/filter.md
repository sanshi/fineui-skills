# Grid 表头过滤（Header Filter）

给列加过滤器：表头菜单里弹出过滤框（文本/数字/日期/下拉/复选），或用**行内过滤行**（inline）。过滤后**重新查询数据并重绑**——服务端过滤是重点，各栈读取过滤条件的 API 不同。

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent) | Core-TagHelper (RazorForms/RazorPages) |
|------|------|----------------|-------------------|-----------------------------------------|
| 全局开启过滤（Grid） | `filters: true` | `AllowFilters="true"` | `.AllowFilters(true)` | `AllowFilters="true"` |
| 某列开启过滤 | 列 `filter: true`（或 `filter:{…}`） | 列 `EnableFilter="true"` | `.EnableFilter(true)` | `EnableFilter="true"` |
| 行内过滤行 | `inlineFilters: true` | `InlineFilters="true"` | `.InlineFilters(true)` | `InlineFilters="true"` |
| 多条件 | `filter:{ multi:true, matcherDefault:'all' }` | `<Filter EnableMultiConditions="true">` | `.Filter(F.GridFilter().EnableMultiConditions(true))` | `<Filter EnableMultiConditions="true">` |
| 过滤变更事件 | listener `filterchange` | `OnFilterChanged`（旧名 `OnFilterChange` 仍兼容） | `.OnFilterChanged(Url.Action(...),"Grid1")` | `OnFilterChanged` |
| 读取过滤条件 | 回调参数 `filteredData` | `Grid1.FilteredData` | 回发参数 `JArray Grid1_filteredData` | RazorForms `Grid1.FilteredData` / RazorPages `JArray Grid1_filteredData` |

> **事件命名（v15.2 起已统一）**：服务端一律用过去式带 "d" 的 **`OnFilterChanged`**（Pro 与 Core 三模式相同，与 `OnPageIndexChanged` / `OnPageSizeChanged` 命名一致）。Pro 旧名 `OnFilterChange`（无 d）已标记 `[Obsolete]`、仍可编译但不推荐。**F.js 客户端 listener 仍是小写 `filterchange`**（客户端事件名，天然不带 d，不受此次统一影响）。

---

## 1. 开启过滤 + 列过滤字段类型

Grid 上开 `AllowFilters`，列上开 `EnableFilter`，用 `<Filter>` 的 `<Field>` 指定过滤输入控件（不指定默认文本框），`<Operator>` 指定运算符下拉。

### F.js —— 列 `filter` 对象

```javascript
columns: [
    { text: '姓名', field: 'Name', filter: true },                          // 文本过滤（默认）
    { text: '入学年份', field: 'EntranceYear', fieldType: 'int',
      filter: { multi: true, matcherDefault: 'all',
        operator: { type: 'DropDownList', value: 'greater',
          data: [['greater','大于'],['less','小于'],['equal','等于']] },
        field: { type: 'NumberBox', noDecimal: true, noNegative: true } } },
    { text: '所学专业', field: 'Major',
      filter: { field: { type: 'DropDownList', multiSelect: true, multiSelectMode: 'tags',
        fields: ['value','text','display'], data: [['化学系','化学系'],['物理系','物理系']] } } }
]
// Grid 级：filters: true
```

### Pro（WebForms，aspx）

```aspx
<f:Grid ID="Grid1" runat="server" AllowFilters="true" OnFilterChanged="Grid1_FilterChanged" DataKeyNames="Id">
    <Columns>
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名" EnableFilter="true" />
        <f:RenderField ColumnID="EntranceYear" DataField="EntranceYear" FieldType="Int" HeaderText="入学年份" EnableFilter="true">
            <Filter EnableMultiConditions="true">
                <Operator>
                    <f:DropDownList runat="server">
                        <f:ListItem Text="大于" Value="greater" Selected="true" />
                        <f:ListItem Text="小于" Value="less" /><f:ListItem Text="等于" Value="equal" />
                    </f:DropDownList>
                </Operator>
                <Field><f:NumberBox runat="server" NoDecimal="true" NoNegative="true" /></Field>
            </Filter>
        </f:RenderField>
    </Columns>
</f:Grid>
```

### Core-MVC（Fluent API）

```csharp
@(F.Grid().ID("Grid1").AllowFilters(true).OnFilterChanged(Url.Action("Grid1_FilterChanged"), "Grid1")
    .Columns(
        F.RenderField().HeaderText("姓名").DataField("Name").EnableFilter(true),
        F.RenderField().HeaderText("所学专业").DataField("Major").EnableFilter(true)
            .Filter(F.GridFilter().Field(
                F.DropDownList().EnableEdit(false).AutoSelectFirstItem(false)
                    .EnableMultiSelect(true).MultiSelectMode(MultiSelectMode.Tags)
                    .Items(F.ListItem().Text("化学系").Value("化学系"), F.ListItem().Text("物理系").Value("物理系"))))
    ).DataSource(ViewBag.Grid1DataSource))
```

### Core-TagHelper（RazorForms / RazorPages，列标签相同）

```html
<f:Grid ID="Grid1" AllowFilters="true" OnFilterChanged="Grid1_FilterChanged">   <%-- RazorPages 见 §5 的 Url.Handler 写法 --%>
    <Columns>
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名" EnableFilter="true" />
        <f:RenderField ColumnID="Major" DataField="Major" HeaderText="所学专业" EnableFilter="true">
            <Filter>
                <Field>
                    <f:DropDownList EnableEdit="false" AutoSelectFirstItem="false" EnableMultiSelect="true" MultiSelectMode="Tags">
                        <f:ListItem Text="化学系" Value="化学系" /><f:ListItem Text="物理系" Value="物理系" />
                    </f:DropDownList>
                </Field>
            </Filter>
        </f:RenderField>
    </Columns>
</f:Grid>
```

**可选过滤字段控件**：文本 `TextBox`（默认）、数字 `NumberBox`、日期 `DatePicker`、下拉 `DropDownList`（可多选 tags）、复选框列表 `CheckBoxList`、单选 `RadioButtonList`。

---

## 2. 行内过滤（Inline Filter，独立一行过滤输入）

Grid 上 `InlineFilters="true"`（F.js `inlineFilters: true`），在表头下方直接显示一行过滤输入框，无需点开菜单。

```javascript
// F.js
F.create({ type: 'Grid', inlineFilters: true, filters: true, columns: [ { text:'姓名', field:'Name', filter:true }, ... ] });
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... AllowFilters="true" InlineFilters="true"> <Columns> ... </Columns> </f:Grid>
```

> **行内过滤仅支持**：文本框、数字框、日期选择器、下拉列表——**不支持多条件过滤**（多条件请用表头菜单过滤）。

---

## 3. 过滤初始值

```javascript
// F.js —— Grid 级 filteredData（运行时可 F.ui.grid1.filteredData = newData）
filteredData: [{ column: 'Name', field: 'Name', multi: false, items: [{ value: '张' }] }]
```
```csharp
// Pro / RazorForms —— 后台按列设 ColumnFilteredData
Grid1.FindColumn("Name").ColumnFilteredData = new GridColumnFilteredData() {
    Items = { new GridColumnFilteredItem() { Value = "张" } }
};
// 清空全部过滤：Grid1.FilteredData = null;
```

---

## 4. 服务端过滤处理（重点，各栈读取 API 不同）

过滤变更时回发，**读取当前过滤条件 → 按条件查询 → 重绑**。仓库示例把"按条件筛选 DataTable"的逻辑抽到了 `Code/` 下的辅助类里（Pro/RF 用 `NewFilteredTable`，MVC/RP 用 `FilteredTable`）。

### 读取条件的差异

| 栈 | 读过滤条件 | 每条条件的字段 |
|----|-----------|---------------|
| **Pro / RazorForms** | 遍历 `Grid1.Columns` 的 `column.ColumnFilteredData` | `.ColumnID` `.Matcher` `.Items[]`（每项 `.Value` `.Operator`） |
| **Core-MVC / RazorPages** | 回发参数 `JArray Grid1_filteredData` | JSON `"column"` `"multi"` `"matcher"` `"items"`（每项 `"operator"` `"value"` `"text"`） |
| **F.js** | listener 回调第二参 `filteredData` | 前端自行按条件过滤数组 |

### Pro（WebForms）—— 服务端事件 `OnFilterChanged`

```csharp
protected void Grid1_FilterChanged(object sender, EventArgs e) {
    BindGrid();                                   // 内部用 Grid1.FilteredData / NewFilteredTable 过滤后 DataBind
    labResult.Text = "过滤数据：" + EncodeJson(Grid1.FilteredData);
}
```

### Core-MVC —— Controller action，回发参数 `JArray Grid1_filteredData`

```csharp
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_FilterChanged(string[] Grid1_fields, JArray Grid1_filteredData) {
    FilteredTable filteredTable = new FilteredTable { FilterDataRowItem = FilterDataRowItemImplement };
    DataTable table = filteredTable.GetFilteredTable(Grid1_filteredData);
    UIHelper.Grid("Grid1").DataSource(table, Grid1_fields);
    return UIHelper.Result();
}
```

### Core-RazorForms —— 后台服务端事件 `Grid1_FilterChanged`（有 d），读 `Grid1.FilteredData`

```csharp
protected void Grid1_FilterChanged(object sender, EventArgs e) {
    NewFilteredTable ft = new NewFilteredTable { FilterDataRowItem = FilterDataRowItemImplement };
    Grid1.DataSource = ft.GetFilteredTable(Grid1);   // 内部读 column.ColumnFilteredData
    Grid1.DataBind();
}
```

### Core-RazorPages —— `OnPost` 处理器，回发参数同 MVC

```csharp
public IActionResult OnPostGrid1_FilterChanged(string[] Grid1_fields, JArray Grid1_filteredData) {
    FilteredTable ft = new FilteredTable { FilterDataRowItem = FilterDataRowItemImplement };
    UIHelper.Grid("Grid1").DataSource(ft.GetFilteredTable(Grid1_filteredData), Grid1_fields);
    return UIHelper.Result();
}
```

### F.js —— 纯客户端过滤

```javascript
listeners: {
    filterchange: function (event, filteredData) {
        F.ui.grid1.loadData(getFilteredDataSource(filteredData, filterer));   // 自己按条件过滤数组后重载
        F.ui.grid1.clearSelection();
    }
}
```

> **绑定处理器**：MVC/RazorPages 的按钮式回发要带 `Grid1_fields`（`OnFilterChangedFields="Grid1"` / `Url.Action(..., "Grid1")`）以保持列；Pro/RazorForms 是服务端事件，直接 `DataBind()`。

---

## 关键约束

1. **事件名已统一（v15.2）**：服务端全部用 `OnFilterChanged`（带 d，Pro 与 Core 一致）；Pro 旧名 `OnFilterChange` 为兼容别名（`[Obsolete]`，仍可用）。F.js 客户端 listener 仍是 `filterchange`。同理分页事件用 `OnPageIndexChanged` / `OnPageSizeChanged`。
2. **过滤 = 重新查询**：过滤本身只收集条件，真正筛选/翻页要在回发里按条件重查数据库再 `DataSource`/`DataBind`。
3. **读条件两套 API**：Pro/RazorForms 读 `Grid1.FilteredData` / `column.ColumnFilteredData`（服务端对象）；MVC/RazorPages 读回发参数 `JArray Grid1_filteredData`（JSON）。别混。
4. **行内过滤限制**：`InlineFilters` 不支持多条件；多条件用表头菜单 `<Filter EnableMultiConditions="true">`。

## See also

- [data-loading.md](data-loading.md)：过滤后重新查询与分页配合
- [sorting.md](sorting.md)：服务端排序（同为"回发→重查→重绑"范式）
- `fineui-form`：过滤字段用到的 NumberBox / DropDownList / CheckBoxList 等控件
