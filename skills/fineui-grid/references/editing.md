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

---

## 关键约束

1. **读编辑数据的 API 四套不同，别混**：
   - F.js：`grid.getModifiedData()`（方法）；
   - **Pro（WebForms）**：`Grid1.GetModifiedData()` / `GetModifiedDict()` / `GetMergedData()`（**方法**）；
   - **Core-MVC / RazorPages**：回发参数 `JArray Grid1_mergedData`（**没有** `GetMergedData()` 方法）；
   - **Core-RazorForms**：控件属性 `Grid1.MergedData`（**没有** `GetMergedData()` 方法）。
2. **合并数据结构**：C# 端 `JArray`，逐行 `row["values"]`（`JObject`），再 `values.Value<T>("列名")`。
3. **新增/删除行**需 `IncludeMergedData="true"`；否则只拿到修改、拿不到新增删除。
4. **点击进入编辑**：F.js `cellEditingClicks`；C# 三模式 `ClicksToEdit`（1 单击 / 2 双击）。
5. **列级只读**：F.js 不设 `editable` 或用 `beforeedit` 返回 `false`；C# 用 `EnableColumnEdit="false"`。
6. **编辑器控件**：TextBox / DropDownList / NumberBox / DatePicker 等表单字段（详见 `fineui-form` 技能）。

## See also

- [columns.md](columns.md)：列渲染、`RendererFunction`
- [selection.md](selection.md)：`DataKeyNames` 与选中行（编辑常与选中配合）
