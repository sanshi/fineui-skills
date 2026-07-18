# Grid 行选择（Selection）

整行复选框选择：勾选行、读取/设置选中行。四端并列，最小代码。

## 两个易混概念（务必区分）

| | 作用 | 属性 |
|--|------|------|
| **整行复选框选择** | 在最左侧加一列复选框用于**选中整行** | F.js `checkboxSelect: true` / C# `EnableCheckBoxSelect="true"` |
| **布尔展示列** | 展示某个 bool **数据字段**（如"是否在校"） | F.js `columnType:'checkboxfield'` / C# `<f:RenderCheckField>`（见 [columns.md](columns.md)） |

本文只讲前者。

## 选择模式

- **多选**（默认）：开启复选框选择即多选。
- **单选**：C# 端 `EnableCheckBoxSelect(true)` + `EnableMultiSelect(false)`。

---

## 1. 开启复选框多选

### F.js

```javascript
F.create({
    type: 'Grid', id: 'Grid1', isFluid: true, renderTo: '#wrap', title: '表格',
    checkboxSelect: true,               // 整行复选框多选
    idField: 'Id', textField: 'Name',
    columns: [ /* ... */ ],
    data: [ /* ... */ ]
});
```

### FineUIPro (aspx)

```aspx
<f:Grid ID="Grid1" runat="server" IsFluid="true" Title="表格"
        EnableCheckBoxSelect="true" DataKeyNames="Id,Name">
    <Columns> <%-- ... --%> </Columns>
</f:Grid>
```

### Core 流式（MVC）

```csharp
@(F.Grid().IsFluid(true).Title("表格").ID("Grid1").DataIDField("Id").DataTextField("Name")
    .EnableCheckBoxSelect(true)
    .Columns( /* ... */ )
    .DataSource(ViewBag.Grid1DataSource))
```

### Core TagHelper

```html
<%-- RazorForms：主键字段用 _DataKeyNames --%>
<f:Grid ID="Grid1" IsFluid="true" Title="表格" EnableCheckBoxSelect="true"
        DataIDField="Id" DataTextField="Name" _DataKeyNames="Id,Name,Gender,Major">
    <Columns> <!-- ... --> </Columns>
</f:Grid>
```

---

## 2. 默认选中行

`SelectedRowIndexArray` 用**0 基行索引**（`4, 9` = 第 5、10 行）。F.js 用行 ID。

```javascript
// F.js —— 数据加载后按行 ID 选中
listeners: {
    dataload: function () { this.selectRows(['R5', 'R10']); }
}
```
```csharp
// Pro（后置代码，!IsPostBack 内，DataBind 之后）
Grid1.SelectedRowIndexArray = new int[] { 4, 9 };
```
```csharp
// Core 流式（View 内）
.DataSource(ViewBag.Grid1DataSource).SelectedRowIndexArray(4, 9)
```
```html
<!-- Core TagHelper（RazorPages，标签内内联）-->
<f:Grid ... SelectedRowIndexArray="@(new int[] { 4, 9 })">
```

---

## 3. 读取选中行

C# 端有两种范式。

### 方式 A：服务端按索引读取（Pro / RazorForms）

前提：声明了 `DataKeyNames`（Pro）/ `_DataKeyNames`（RazorForms），且数据在服务端 `DataBind()`，这样 `DataKeys` 可用。

```csharp
// Pro / RazorForms 后置代码
protected void Button1_Click(object sender, EventArgs e)
{
    foreach (int rowIndex in Grid1.SelectedRowIndexArray)   // 0 基索引
    {
        object id   = Grid1.DataKeys[rowIndex][0];   // 对应 DataKeyNames 第 1 个字段 Id
        object name = Grid1.DataKeys[rowIndex][1];   // 第 2 个字段 Name
        // ... 用 id / name 处理业务
    }
}
```

配套：点击时若无选中则不回发——
```csharp
// Page_Load 内（!IsPostBack）
Button1.OnClientClick = Grid1.GetNoSelectionAlertInTopReference("没有选中项！");
```

### 方式 B：客户端收集 JSON 回发（Core MVC / RazorPages）

客户端把选中行收集成 JSON，作为参数回发；服务端用 `JArray` 接收（需 `using Newtonsoft.Json.Linq;`）。

```html
<!-- 页面内 JS：收集选中行 -->
<script>
    function getGridSelectedRows() {
        var result = [], grid = F.ui.Grid1;
        $.each(grid.getSelectedRows(true), function (i, item) {
            // item = { id, text, values: { 字段: 值, ... } }
            result.push([item.id, item.text, item.values.Gender, item.values.Major]);
        });
        return F.toJSON(result);
    }
</script>
```

```csharp
// Core MVC（流式）：按钮把 getGridSelectedRows() 作为参数 selected 回发
@(F.Button().Text("选中了哪些行").ID("Button1")
    .OnClick(Url.Action("Button1_Click"), new Parameter("selected", "getGridSelectedRows()")))
```
```csharp
// Core MVC Controller
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Button1_Click(JArray selected)
{
    foreach (JArray item in selected) { var id = item[0]; var text = item[1]; /* ... */ }
    return UIHelper.Result();
}
```

```html
<!-- Core RazorPages：按钮 + 参数 -->
<f:Button Text="选中了哪些行" ID="Button1" OnClick="@Url.Handler(&quot;Button1_Click&quot;)"
          OnClickParameter1="@(new Parameter(&quot;selected&quot;, &quot;getGridSelectedRows()&quot;))"></f:Button>
```
```csharp
// Core RazorPages 后置代码（OnPost 处理器）
public IActionResult OnPostButton1_Click(JArray selected)
{
    foreach (JArray item in selected) { /* ... */ }
    return UIHelper.Result();
}
```

### F.js：纯客户端读取

```javascript
if (!F.ui.Grid1.hasSelection()) { F.alert('没有选中项！'); return; }
var rows = F.ui.Grid1.getSelectedRows(true);   // [{ id, text, values: {...} }, ...]
rows.forEach(function (item) { console.log(item.id, item.values.Name); });
```

---

## 4. 编程设置选中行（服务端主动选中）

```csharp
// Pro / RazorForms
Grid1.SelectedRowIndexArray = new int[] { 1, 5, 7 };
```
```csharp
// Core 流式 / RazorPages（回发处理器内，用 UIHelper）
UIHelper.Grid("Grid1").SelectedRowIndexArray(1, 5, 7);
return UIHelper.Result();
```
```javascript
// F.js
F.ui.Grid1.selectRows(['R2', 'R6', 'R8']);
```

---

## 5. 复选框单选（Core 流式示例）

`EnableCheckBoxSelect(true)` + `EnableMultiSelect(false)` = 复选框单选；配 `rowselect` / `rowdeselect` 事件回发单行。

```csharp
// View
@(F.Grid().IsFluid(true).Title("表格（单选）").ID("Grid1").DataIDField("Id").DataTextField("Name")
    .EnableCheckBoxSelect(true).EnableMultiSelect(false)
    .Listener("rowselect", "onGrid1RowSelect").Listener("rowdeselect", "onGrid1RowDeselect")
    .Columns( /* ... */ ).DataSource(ViewBag.Grid1DataSource))
```
```csharp
// Controller：rowselect 回发的参数（框架约定）
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_RowSelect(string rowId, string rowText, int rowIndex, string columnText, bool isDeselect)
{
    // rowIndex 0 基；isDeselect 区分选中/取消
    return UIHelper.Result();
}
```

## See also

- [columns.md](columns.md)：布尔展示列（`checkboxfield` / `RenderCheckField`）与整行选择的区别
- SKILL.md 约束 6：RazorForms 与 RazorPages 的事件模型差异（`OnClick="方法名"` vs `OnClick="@Url.Handler(...)"`）
