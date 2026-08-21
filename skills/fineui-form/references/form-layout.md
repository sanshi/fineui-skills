# 表单容器与布局（Form Layout）

## Form vs SimpleForm

| | 用途 | C# 写法 | FineUIJava 写法 | F.js 写法 |
|--|------|---------|-----------------|-----------|
| **SimpleForm** | 单列表单 | `<f:SimpleForm><Items>字段…</Items></f:SimpleForm>` | `<f:simple-form><f:items>字段…</f:items></f:simple-form>` | `type:'Form'` + `layout:'anchor'` |
| **Form** | 多列表单 | `<f:Form><Rows><f:FormRow><Items>…</Items></f:FormRow></Rows></f:Form>` | `<f:form><f:rows><f:form-row><f:items>…</f:items></f:form-row></f:rows></f:form>` | `type:'Form'` + 嵌套 `Panel layout:'column'` |

> F.js 没有单独的 `SimpleForm` 类型；单列用 `Form` + `layout:'anchor'`，多列用嵌套的列布局 Panel。C# 的 `SimpleForm` 渲染到前端也是一个 Form。
> **FineUIJava**：全 kebab-case；`<f:form>` 的行既可包在 `<f:rows>` 里（对齐 Core 的 `<Rows>`），也可直接放在 `<f:items>` 下——两种示例都存在。

## 容器属性

常用：`LabelWidth`（标签宽度）、`LabelAlign`（`Left`/`Right`/`Top`）、`RedStarPosition`（红星位置 `AfterText`/`BeforeText`）、`BodyPadding`、`IsFluid`（100% 宽）、`Title`。

```javascript
// F.js —— fieldDefaults 统一给所有字段设默认（如标签宽、错误提示定位）
F.create({
    type: 'Form', isFluid: true, layout: 'anchor', bodyPadding: 10, title: '表单',
    fieldDefaults: { labelWidth: 100, msgTarget: 'qtip' },
    items: [ /* 字段 */ ]
});
```
```aspx
<%-- Pro --%>
<f:Form ID="Form1" runat="server" IsFluid="true" BodyPadding="10px" LabelWidth="100px"
        LabelAlign="Left" RedStarPosition="AfterText" Title="表单">
    <Rows> ... </Rows>
</f:Form>
```
```csharp
// Core-MVC（Fluent）
@(F.Form().IsFluid(true).BodyPadding(10).LabelWidth(100).RedStarPosition(RedStarPosition.AfterText).Title("表单")
    .Rows( /* ... */ ))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages）-->
<f:Form ID="Form1" IsFluid="true" BodyPadding="10" LabelWidth="100" RedStarPosition="AfterText" Title="表单">
    <Rows> ... </Rows>
</f:Form>
```
```html
<!-- FineUIJava（Thymeleaf 方言）：属性值 Left/Right/Top、AfterText 保持 PascalCase -->
<f:form id="Form1" is-fluid="true" body-padding="10" label-width="100" label-align="Left" red-star-position="AfterText" title="表单">
    <f:rows> ... </f:rows>
</f:form>
```

## 多列布局（Form + FormRow）

每个 `<f:FormRow>` = 一行；行内 `<Items>` 放几个字段就是几列。`ColumnWidths` 显式指定列宽（可混用 px 与 %）。

```aspx
<%-- Pro --%>
<f:Form ID="Form1" runat="server" IsFluid="true" LabelWidth="100px" Title="表单">
    <Rows>
        <f:FormRow>                          <%-- 两列，默认平分 --%>
            <Items>
                <f:Label ID="Label1" runat="server" Label="标签" Text="值" />
                <f:CheckBox ID="CheckBox1" runat="server" Label="复选框" Text="复选框" />
            </Items>
        </f:FormRow>
        <f:FormRow ColumnWidths="50% 50%">    <%-- 显式列宽 --%>
            <Items>
                <f:DropDownList ID="ddl1" runat="server" Label="下拉列表" Required="true">
                    <f:ListItem Text="A" Value="0" />
                </f:DropDownList>
                <f:TextBox ID="TextBox1" runat="server" Label="文本框" Required="true" />
            </Items>
        </f:FormRow>
        <f:FormRow ColumnWidths="20px 100%"> <%-- 20px + 占满剩余 --%>
            <Items>
                <f:Label runat="server" Text="1." />
                <f:TextBox ID="TextBox2" runat="server" Label="备注" />
            </Items>
        </f:FormRow>
    </Rows>
</f:Form>
```
```csharp
// Core-MVC（Fluent）—— FormRow().Items(...) 每行放多个字段
@(F.Form().IsFluid(true).LabelWidth(100).Title("表单")
    .Rows(
        F.FormRow().Items(
            F.Label().ID("Label1").Label("标签").Text("值"),
            F.CheckBox().ID("CheckBox1").Label("复选框").Text("复选框")
        ),
        F.FormRow().ColumnWidths("50% 50%").Items(
            F.DropDownList().ID("ddl1").Label("下拉列表").Required(true).Items(F.ListItem().Text("A").Value("0")),
            F.TextBox().ID("TextBox1").Label("文本框").Required(true)
        )
    ))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages，写法同 Pro 的 Rows/FormRow）-->
<f:Form ID="Form1" IsFluid="true" LabelWidth="100" Title="表单">
    <Rows>
        <f:FormRow ColumnWidths="50% 50%">
            <Items>
                <f:DropDownList ID="ddl1" Label="下拉列表" Required="true"><f:ListItem Text="A" Value="0" /></f:DropDownList>
                <f:TextBox ID="TextBox1" Label="文本框" Required="true"></f:TextBox>
            </Items>
        </f:FormRow>
    </Rows>
</f:Form>
```

```html
<!-- FineUIJava（Thymeleaf 方言）：每个 <f:form-row> = 一行，行内 <f:items> 放几个字段就是几列；列宽 column-widths -->
<f:form id="Form1" is-fluid="true" label-width="100" title="表单">
    <f:rows>
        <f:form-row column-widths="50% 50%">
            <f:items>
                <f:drop-down-list id="ddl1" label="下拉列表" required="true" show-red-star="true">
                    <f:list-item text="A" value="0"></f:list-item>
                </f:drop-down-list>
                <f:text-box id="TextBox1" label="文本框" required="true" show-red-star="true"></f:text-box>
            </f:items>
        </f:form-row>
        <f:form-row column-widths="20px 100%">   <!-- 20px + 占满剩余 -->
            <f:items>
                <f:label text="1."></f:label>
                <f:text-box id="TextBox2" label="备注"></f:text-box>
            </f:items>
        </f:form-row>
    </f:rows>
</f:form>
```

F.js 多列用嵌套列布局：

```javascript
items: [{
    type: 'Panel', layout: 'column', border: false, header: false,
    items: [
        { type: 'DatePicker', columnWidth: 0.5, hideLabel: true, fieldLabel: '开始', required: true },
        { type: 'DatePicker', columnWidth: 0.5, hideLabel: true, fieldLabel: '结束', required: true }
    ]
}]
```

## 全局标签配置

标签分隔符、对齐等可全局设：Pro 在 `Web.config` 的 `<FineUIPro>` 段（`FormLabelSeparator="："`、`FormLabelAlign="Left"`）；Core 在 `appsettings.json` 的 `FineUI` 段；**FineUIJava 在 `application.properties` 的 `fineui.*` 键（kebab-case，如 `fineui.form-label-width=100`、`fineui.form-label-align=Left`）**。单表单可用容器属性覆盖。

## 表格样式表单

`EnableTableStyle="true"`（Java：`enable-table-style="true"`）让表单以表格线样式呈现（**用它时要去掉 `BodyPadding`/`body-padding`**）。

## See also

- [fields.md](fields.md)：各字段类型
- [validation.md](validation.md)：校验
