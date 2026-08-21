# Grid 单元格编辑（Editing）

启用可编辑表格、配置列编辑器、读取编辑后的数据。**读取数据的 API 各栈差异很大，是本文重点。**

## 开启编辑 + 列编辑器

三步：① Grid 上开单元格编辑（`cellEditing` / `AllowCellEditing`）+ 点击次数；② 列上开 `editable` / 放 `<Editor>`；③ 编辑器控件（TextBox / DropDownList / NumberBox / DatePicker …）。

### F.js

```javascript
F.create({
    type: 'Grid', id: 'Grid1', isFluid: true, renderTo: '#wrap',
    showSelectedCell: true, cellEditing: true, cellEditingClicks: 2,   // 2=双击进入编辑，1=单击
    columns: [
        { columnType: 'rownumberfield' },
        { text: '姓名', field: 'Name', editable: true, editor: { type: 'TextBox', required: true } },
        { text: '性别', field: 'Gender', fieldType: 'int', editable: true,
          renderer: function (v) { return v == 1 ? '男' : '女'; },
          editor: { type: 'DropDownList', required: true, data: [['1', '男'], ['0', '女']] } },
        { text: '入学年份', field: 'EntranceYear', fieldType: 'int', editable: true,
          editor: { type: 'NumberBox', required: true, noDecimal: true, minValue: 2000, maxValue: 2025 } },
        { text: '入学日期', field: 'EntranceDate', fieldType: 'date', fieldFormat: 'yyyy/MM/dd', editable: true,
          editor: { type: 'DatePicker', required: true } },
        { text: '是否在校', field: 'AtSchool', columnType: 'checkboxfield', editable: true }
    ],
    idField: 'Id', dataUrl: '/api/students'
});
```

### Pro（WebForms，aspx）

```aspx
<f:Grid ID="Grid1" runat="server" AllowCellEditing="true" ClicksToEdit="2" DataKeyNames="Id"> <%-- 1=单击 2=双击 --%>
    <Columns>
        <f:RowNumberField />
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名">
            <Editor><f:TextBox ID="tbxName" Required="true" runat="server" /></Editor>
        </f:RenderField>
        <f:RenderField ColumnID="Gender" DataField="Gender" FieldType="Int" RendererFunction="renderGender" HeaderText="性别">
            <Editor>
                <f:DropDownList ID="ddlGender" Required="true" runat="server">
                    <f:ListItem Text="男" Value="1" /><f:ListItem Text="女" Value="0" />
                </f:DropDownList>
            </Editor>
        </f:RenderField>
        <f:RenderField ColumnID="EntranceYear" DataField="EntranceYear" FieldType="Int" HeaderText="入学年份">
            <Editor><f:NumberBox NoDecimal="true" MinValue="2000" MaxValue="2025" runat="server" /></Editor>
        </f:RenderField>
        <f:RenderCheckField ColumnID="AtSchool" DataField="AtSchool" HeaderText="是否在校" />
        <f:RenderField ColumnID="Major" DataField="Major" HeaderText="所学专业" EnableColumnEdit="false" /> <%-- 列级只读 --%>
    </Columns>
</f:Grid>
```

### Core-MVC（Fluent API）

```csharp
@(F.Grid().ID("Grid1").AllowCellEditing(true).ClicksToEdit(2)
    .Columns(
        F.RowNumberField(),
        F.RenderField().ColumnID("Name").DataField("Name").HeaderText("姓名")
            .Editor(F.TextBox().ID("tbxName").Required(true)),
        F.RenderField().ColumnID("Gender").DataField("Gender").FieldType(FieldType.Int).RendererFunction("renderGender").HeaderText("性别")
            .Editor(F.DropDownList().ID("ddlGender").Required(true)
                .Items(F.ListItem().Text("男").Value("1"), F.ListItem().Text("女").Value("0"))),
        F.RenderField().ColumnID("EntranceYear").DataField("EntranceYear").FieldType(FieldType.Int).HeaderText("入学年份")
            .Editor(F.NumberBox().NoDecimal(true).MinValue(2000).MaxValue(2025))
    ).DataSource(ViewBag.Grid1DataSource))
```

### Core-TagHelper（RazorForms / RazorPages，`<Editor>` 子标签）

```html
<f:Grid ID="Grid1" AllowCellEditing="true" ClicksToEdit="2">
    <Columns>
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名">
            <Editor><f:TextBox ID="tbxName" Required="true"></f:TextBox></Editor>
        </f:RenderField>
        <f:RenderField ColumnID="Gender" DataField="Gender" FieldType="Int" RendererFunction="renderGender" HeaderText="性别">
            <Editor>
                <f:DropDownList ID="ddlGender" Required="true">
                    <f:ListItem Text="男" Value="1"></f:ListItem><f:ListItem Text="女" Value="0"></f:ListItem>
                </f:DropDownList>
            </Editor>
        </f:RenderField>
    </Columns>
</f:Grid>
```

### FineUIJava（Thymeleaf 方言，`<f:editor>` 子标签）

结构同 Core-TagHelper：`allow-cell-editing` + `clicks-to-edit`（1 单击 / 2 双击），列内嵌 `<f:editor>` 放表单字段；列级只读用 `enable-column-edit="false"`。

```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:grid id="Grid1" allow-cell-editing="true" clicks-to-edit="1">
    <f:columns>
        <f:row-number-field></f:row-number-field>
        <f:render-field header-text="姓名" column-id="Name" data-field="Name">
            <f:editor><f:text-box id="tbxName" required="true"></f:text-box></f:editor>
        </f:render-field>
        <f:render-field header-text="性别" column-id="Gender" data-field="Gender" field-type="Int" renderer-function="renderGender">
            <f:editor>
                <f:drop-down-list id="ddlGender" required="true">
                    <f:list-item text="男" value="1"></f:list-item>
                    <f:list-item text="女" value="0"></f:list-item>
                </f:drop-down-list>
            </f:editor>
        </f:render-field>
        <f:render-field header-text="入学年份" column-id="EntranceYear" data-field="EntranceYear" field-type="Int">
            <f:editor><f:number-box no-decimal="true" no-negative="true" min-value="2000" max-value="2025"></f:number-box></f:editor>
        </f:render-field>
        <f:render-field header-text="所学专业" column-id="Major" data-field="Major" enable-column-edit="false"></f:render-field>
    </f:columns>
</f:grid>
```

---

## 读取编辑后的数据（各栈 API 不同 —— 重点）

> **新增/删除行**场景需在 Grid 上加 `IncludeMergedData="true"`（Core）/ `IncludeMergedData="true"`（Pro），回发才带“合并数据”。C# 端“合并数据”是 `JArray`，每行是 `{ "values": { 列名: 值 } }`。

### F.js —— 方法 `getModifiedData()`

```javascript
var grid1 = F.ui.Grid1;
var modified = grid1.getModifiedData();   // 仅被改动的行
console.log(modified);
grid1.commitChanges();                    // 提交（清除“已改动”标记）
```

### Pro（WebForms）—— **方法** `GetModifiedData()` / `GetModifiedDict()` / `GetMergedData()`

```csharp
// 只取被改动的单元格：行索引 -> (列名 -> 值)
Dictionary<int, Dictionary<string, object>> dict = Grid1.GetModifiedDict();
foreach (int rowIndex in dict.Keys) {
    int id = Convert.ToInt32(Grid1.DataKeys[rowIndex][0]);
    UpdateRow(id, dict[rowIndex]);   // 仅更新含 key 的列
}

// 或取全部“合并后”数据（含新增/删除，需 IncludeMergedData="true"）
JArray merged = Grid1.GetMergedData();               // using Newtonsoft.Json.Linq;
foreach (JObject row in merged) {
    JObject values = row.Value<JObject>("values");
    string name = values.Value<string>("Name");
    int    year = values.Value<int>("EntranceYear");
}
```

### Core-MVC / RazorPages —— **回发参数** `JArray Grid1_mergedData`

```csharp
// Grid 上需 .IncludeMergedData(true)（Fluent）/ IncludeMergedData="true"（TagHelper）
// 按钮：MVC  .OnClick(Url.Action("btnSubmit_Click"), "Grid1")
//       RazorPages  OnClick="@Url.Handler("btnSubmit_Click")" OnClickFields="Grid1"
[HttpPost, ValidateAntiForgeryToken]                                  // MVC：Controller action
public IActionResult btnSubmit_Click(string[] Grid1_fields, JArray Grid1_mergedData) {
    foreach (JObject row in Grid1_mergedData) {
        JObject values = row.Value<JObject>("values");
        string name = values.Value<string>("Name");
    }
    return UIHelper.Result();
}
// RazorPages：public IActionResult OnPostBtnSubmit_Click(string[] Grid1_fields, JArray Grid1_mergedData) { ... }
```

### Core-RazorForms —— **控件属性** `Grid1.MergedData`

```csharp
protected void btnSubmit_Click(object sender, EventArgs e) {
    foreach (JObject row in Grid1.MergedData) {          // JArray 属性（需 IncludeMergedData="true"）
        JObject values = row.Value<JObject>("values");
        string name = values.Value<string>("Name");
    }
}
```

### FineUIJava —— **方法** `Grid1.getModifiedData()`（含 status）/ `Grid1.getMergedData()`

Java 走**方法**（不像 RazorForms 的属性），两者返回 `List<Map<String, Object>>`，每行有 `values`（`Map<String,Object>`）：

- **`getModifiedData()`**：只含被改动的行，每行带 `status`——`"modified"`（改现有行）/`"newadded"`（新增行）/`"deleted"`（删除行）+ `id`。**处理新增/删除首选它**（靠 status 分流），无需开 `include-merged-data`。
- **`getMergedData()`**：所有未删除行的当前值（含新增/已改/未改）——需先在 `<f:grid>` 上开 `include-merged-data="true"`，适合“整表重建”。

```java
// FineUIJava 页面类 —— 用 getModifiedData() 的 status 分流新增/改/删（推荐）
public void btnSubmit_Click(Object sender, EventArgs e) {
    for (Map<String, Object> row : Grid1.getModifiedData()) {
        String status = String.valueOf(row.get("status"));
        String rowId  = String.valueOf(row.get("id"));
        @SuppressWarnings("unchecked")
        Map<String, Object> values = (Map<String, Object>) row.get("values");   // 改动的列
        if ("modified".equals(status))      { /* 用 values 更新 rowId 对应行 */ }
        else if ("newadded".equals(status)) { /* 用 values 插入新行 */ }
        else if ("deleted".equals(status))  { /* 删除 rowId 对应行 */ }
    }
    Grid1.dataBind();
}
```
```java
// 或整表重建：需 <f:grid include-merged-data="true">
public void btnSubmit_Click(Object sender, EventArgs e) {
    for (Map<String, Object> mergedRow : Grid1.getMergedData()) {
        @SuppressWarnings("unchecked")
        Map<String, Object> values = (Map<String, Object>) mergedRow.get("values");
        String name = String.valueOf(values.get("Name"));
        // ... 用 values 各列重建整表
    }
}
```

> 编辑器控件、渲染函数、`beforeedit`/`aftercelledit` 等客户端监听 JS 与 F.js 完全相同，Java 侧直接复用。

---

## 关键约束

1. **读编辑数据的 API 五套不同，别混**：
   - F.js：`grid.getModifiedData()`（方法）；
   - **Pro（WebForms）**：`Grid1.GetModifiedData()` / `GetModifiedDict()` / `GetMergedData()`（**方法**）；
   - **Core-MVC / RazorPages**：回发参数 `JArray Grid1_mergedData`（**没有** `GetMergedData()` 方法）；
   - **Core-RazorForms**：控件属性 `Grid1.MergedData`（**没有** `GetMergedData()` 方法）；
   - **Java**：**方法** `Grid1.getModifiedData()`（带 `status` 分流新增/改/删）/ `Grid1.getMergedData()`（需 `include-merged-data="true"`），返回 `List<Map<String,Object>>`（**不是 JArray**）。
2. **合并数据结构**：C# 端 `JArray`，逐行 `row["values"]`（`JObject`），再 `values.Value<T>("列名")`；**Java 端是 `Map<String,Object>`，逐行 `row.get("values")`（也是 `Map`），再 `values.get("列名")`**。
3. **新增/删除行**：C# 需 `IncludeMergedData="true"`（Java `include-merged-data="true"`）才能用合并数据；**Java 也可不开该属性、改用 `getModifiedData()` 的 `status`（`modified`/`newadded`/`deleted`）分流**。
4. **点击进入编辑**：F.js `cellEditingClicks`；C# 三模式 `ClicksToEdit`；**Java `clicks-to-edit`**（1 单击 / 2 双击）。
5. **列级只读**：F.js 不设 `editable` 或用 `beforeedit` 返回 `false`；C# 用 `EnableColumnEdit="false"`；**Java `enable-column-edit="false"`**。
6. **编辑器控件**：TextBox / DropDownList / NumberBox / DatePicker 等表单字段（详见 `fineui-form` 技能）。

## See also

- [columns.md](columns.md)：列渲染、`RendererFunction`
- [selection.md](selection.md)：`DataKeyNames` 与选中行（编辑常与选中配合）
