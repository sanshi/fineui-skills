# Grid 行选择（Selection）

整行复选框选择：勾选行、读取/设置选中行。各写法并列，最小代码。

> 选择涉及**数据初始化与事件**，因此 **RazorForms 与 RazorPages 在此有真实差异**（不像列定义那样一致）：RazorForms 走后台 `Page_Load`，RazorPages 走标签内联 + `OnPost`。下文分别给出。

## 两个易混概念（务必区分）

| | 作用 | 属性 |
|--|------|------|
| **整行复选框选择** | 在最左侧加一列复选框用于**选中整行** | F.js `checkboxSelect: true` / C# `EnableCheckBoxSelect="true"` |
| **布尔展示列** | 展示某个 bool **数据字段**（如"是否在校"） | F.js `columnType:'checkboxfield'` / C# `<f:RenderCheckField>`（见 [columns.md](columns.md)） |

本文只讲前者。

## 选择模式

- **多选**（默认）：开启复选框选择即多选。
- **单选**：C# 端 `EnableCheckBoxSelect(true)` + `EnableMultiSelect(false)`；**Java `enable-check-box-select="true"` + `enable-multi-select="false"`**。

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

### Pro（WebForms，aspx）

```aspx
<f:Grid ID="Grid1" runat="server" IsFluid="true" Title="表格"
        EnableCheckBoxSelect="true" DataIDField="Id" DataKeyNames="Id,Name">
    <Columns> <%-- ... --%> </Columns>
</f:Grid>
```

### Core-MVC（Fluent API）

```csharp
@(F.Grid().IsFluid(true).Title("表格").ID("Grid1").DataIDField("Id").DataTextField("Name")
    .EnableCheckBoxSelect(true)
    .Columns( /* ... */ )
    .DataSource(ViewBag.Grid1DataSource))
```

### Core-TagHelper（RazorForms / RazorPages）

```html
<%-- 标签相同；RazorForms 用 _DataKeyNames 声明服务端读取所需主键 --%>
<f:Grid ID="Grid1" IsFluid="true" Title="表格" EnableCheckBoxSelect="true"
        DataIDField="Id" DataTextField="Name" _DataKeyNames="Id,Name,Gender,Major">
    <Columns> <!-- ... --> </Columns>
</f:Grid>
```

### FineUIJava（Thymeleaf 方言）

> **注意**：服务端读取所需主键，Java 用 **`data-key-names`**（kebab-case，**不是** RazorForms 那个带下划线的 `_DataKeyNames`）。

```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:grid id="Grid1" is-fluid="true" title="表格" enable-check-box-select="true"
        data-id-field="Id" data-text-field="Name" data-key-names="Id,Name,Gender,Major">
    <f:columns> <!-- ... --> </f:columns>
</f:grid>
```

---

## 2. 默认选中行

所有栈都用稳定行 ID 初始化选择（下面假定第 5、10 行的 `Id` 分别为 `105`、`110`）。
Pro/Core 的 `SelectedRowIndex` / `SelectedRowIndexArray` 已废弃；Java 已直接删除同名 getter/setter 与 `selected-row-index-array` 模板属性。行索引会随分页、排序和行移动变化，不用于新代码。**注意各模式初始化位置不同**。

```javascript
// F.js —— 数据加载后按行 ID 选中
listeners: {
    dataload: function () { this.selectRows(['105', '110']); }
}
```
```csharp
// Pro / RazorForms —— 后台 Page_Load（!IsPostBack）内，DataBind 之后
Grid1.SelectedRowIDArray = new string[] { "105", "110" };
```
```csharp
// Core-MVC（Fluent API）—— View 内链式
.DataSource(ViewBag.Grid1DataSource).SelectedRowIDArray("105", "110")
```
```html
<!-- Core-RazorPages —— 标签内联 -->
<f:Grid ... SelectedRowIDArray="@(new string[] { "105", "110" })">
```
```java
// FineUIJava 页面类 —— Page_Load（!isPostBack()）内，dataBind() 之后
Grid1.setSelectedRowIdArray(new String[] { "105", "110" });
```

---

## 3. 读取选中行数据

C# 端按模式选用。Pro、RazorForms 和 Java 已把稳定 ID 关联封装进 Grid 实例方法；MVC、RazorPages 仍走客户端收集。

### 方式 A：Grid 实例方法（Pro / RazorForms）

前提：设置 `DataIDField`，并让 `DataKeyNames`（Pro）/ `_DataKeyNames`（RazorForms）包含该字段；数据在服务端 `DataBind()`。

```csharp
// Pro / RazorForms 后台代码
protected void Button1_Click(object sender, EventArgs e)
{
    foreach (object[] dataKeys in Grid1.GetSelectedDataKeys())
    {
        object id   = dataKeys[0];   // 对应 DataKeyNames 第 1 个字段 Id
        object name = dataKeys[1];   // 第 2 个字段 Name
        // ... 用 id / name 处理业务
    }
}
```

配套：点击时若无选中则不回发。用页面具名函数检查客户端选择状态：

```aspx
<f:Button ID="Button1" runat="server" Text="处理选中行"
    ClickHandler="onProcessSelectedClick" OnClick="Button1_Click" />
<script>
    var Grid1ClientID = '<%= Grid1.ClientID %>';
    function onProcessSelectedClick(event) {
        if (!F(Grid1ClientID).hasSelection()) {
            F.alert({ message: '没有选中项！', target: '_top' });
            return false;
        }
    }
</script>
```

### 方式 A（Java）：Grid 实例方法（同 RazorForms 范式）

前提：设置 `data-id-field`，并让 `data-key-names` 包含该字段；数据在服务端 `dataBind()`。

```java
// FineUIJava 页面类
public void Button1_Click(Object sender, EventArgs e) {
    List<Object[]> selectedDataKeys = Grid1.getSelectedDataKeys();
    if (selectedDataKeys.isEmpty()) { showNotify("没有选中项！"); return; }
    for (Object[] keys : selectedDataKeys) {
        Object id = keys[0];   // 对应 data-key-names 第 1 个字段
        Object name = keys[1]; // 第 2 个字段
        // ... 用 id / name 处理业务
    }
}
```

`GetSelectedDataKeys()` / `getSelectedDataKeys()` 的返回顺序与选中行 ID 顺序一致，每行字段顺序与数据键声明一致。内部按 `DataIDField` 对应的数据键关联，不使用行索引，所以内存分页、排序和行移动无需计算偏移。配置缺失、ID 重复或选中 ID 找不到数据键时会抛错，不会按索引猜测。

返回值来自客户端回传，只适合界面交互与展示。数据库分页跨页选择，或删除、审批、金额、权限等需要权威数据的业务，应读取 `SelectedRowIDArray` / `getSelectedRowIdArray()` 后查询数据库。

> 客户端读取（如 `notifySelectedRows('Grid1')`、`F.ui.Grid1.getSelectedRows(true)`、`getCheckedRows(true)`）与 F.js 完全相同；需回发服务端时用 `F.customEvent('事件名', 数据)` 触发页面类的 `Page_CustomEvent`。

### 方式 B：客户端收集 JSON 回发（Core-MVC / RazorPages）

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
// Core-MVC（Fluent API）：按钮把 getGridSelectedRows() 作为参数 selected 回发
@(F.Button().Text("选中了哪些行").ID("Button1")
    .OnClick(Url.Action("Button1_Click"), new Parameter("selected", "getGridSelectedRows()")))
```
```csharp
// Core-MVC Controller
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Button1_Click(JArray selected)
{
    foreach (JArray item in selected) { var id = item[0]; var text = item[1]; /* ... */ }
    return UIHelper.Result();
}
```

```html
<!-- Core-RazorPages：按钮 + 参数 -->
<f:Button Text="选中了哪些行" ID="Button1" OnClick="@Url.Handler(&quot;Button1_Click&quot;)"
          OnClickParameter1="@(new Parameter(&quot;selected&quot;, &quot;getGridSelectedRows()&quot;))"></f:Button>
```
```csharp
// Core-RazorPages 后台（OnPost 处理器）
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
Grid1.SelectedRowIDArray = new string[] { "102", "106", "108" };
```
```csharp
// Core-MVC / RazorPages（回发处理器内，用 UIHelper）
UIHelper.Grid("Grid1").SelectedRowIDArray("102", "106", "108");
return UIHelper.Result();
```
```java
// FineUIJava 页面类（处理器内直接用控件字段，void）
Grid1.setSelectedRowIdArray(new String[] { "102", "106", "108" });
```
```javascript
// F.js
F.ui.Grid1.selectRows(['R2', 'R6', 'R8']);
```

---

## 5. 复选框单选（Core-MVC 示例）

`EnableCheckBoxSelect(true)` + `EnableMultiSelect(false)` = 复选框单选；配 `rowselect` / `rowdeselect` 事件回发单行。

```csharp
// View（Fluent API）
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
- SKILL.md 约束 5、6：Core 三模式数据初始化差异、RazorForms 与 RazorPages 的后台模型差异
