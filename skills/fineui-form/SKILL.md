---
name: fineui-form
description: >
  帮助开发者使用 FineUI 的表单：Form / SimpleForm 容器、表单字段（TextBox / TextArea / NumberBox /
  DatePicker / DropDownList / CheckBox / RadioButtonList 等）、字段与整表校验、读取字段值。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper），
  以及 FineUIJava（Spring Boot + Thymeleaf 方言标签，kebab-case）。
  Trigger phrases（触发词）: "FineUI 表单", "F.Form", "SimpleForm", "FormRow", "表单校验",
  "ValidateForms", "Required", "TextBox", "NumberBox", "DatePicker", "DropDownList",
  "RadioButtonList", "CheckBox", "CheckBoxList", "TimePicker", "Label", "Hidden",
  "FileUpload", "TriggerBox", "DropDownBox", "HtmlEditor", "MarkInvalid", "字段标签",
  "LabelWidth", "读取表单值", "下拉树", "文件上传", "富文本编辑器",
  "FineUIJava", "Spring Boot", "Thymeleaf", "@FineUIPage", "text-box", "simple-form", "form-row".
metadata:
  author: FineUI
  version: "16.0"
  compatibility: FineUI v16.0（事件驱动变化回发与 RawHtml 安全模型）
---

# FineUI 表单技能（Form）

## 何时使用（When to Use）

- 搭建录入/编辑表单，配置字段、标签、必填红星、多列布局
- 字段校验（必填、正则、比较、自定义）与整表提交校验
- 读取/回填字段值

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / **Java（Spring Boot）**（判定见 `fineui-foundation`）。
2. **单列还是多列？**
   - 单列：**SimpleForm**（C#，字段直接放 `<Items>`）；F.js 用 `Form` + `layout: 'anchor'`。
   - 多列：**Form**（C#，`<Rows><f:FormRow ColumnWidths="...">`）；F.js 用嵌套 `Panel layout:'column'` + `columnWidth`。

## 各写法速览（一个必填字段 + 提交校验）

```javascript
// ① F.js
F.create({
    type: 'Form', isFluid: true, id: 'form1', renderTo: '#wrap', title: '登录', layout: 'anchor', bodyPadding: 10,
    fieldDefaults: { labelWidth: 100 },
    items: [
        { type: 'TextBox', id: 'tbxName', fieldLabel: '用户名', required: true, emptyText: '请输入用户名' },
        { type: 'Button', text: '提交', validateForm: 'form1', handler: function () { showNotify('通过'); } }
    ]
});
```
```aspx
<%-- ② Pro（WebForms）--%>
<f:SimpleForm ID="SimpleForm1" runat="server" IsFluid="true" BodyPadding="10px" Title="登录">
    <Items>
        <f:TextBox runat="server" ID="tbxName" Label="用户名" Required="true" ShowRedStar="true" EmptyText="请输入用户名" />
        <f:Button ID="btnSubmit" runat="server" Text="提交" ValidateForms="SimpleForm1" OnClick="btnSubmit_Click" />
    </Items>
</f:SimpleForm>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.SimpleForm().IsFluid(true).BodyPadding(10).Title("登录").ID("SimpleForm1")
    .Items(
        F.TextBox().ID("tbxName").Label("用户名").Required(true).ShowRedStar(true).EmptyText("请输入用户名"),
        F.Button().Text("提交").ValidateForms("SimpleForm1").OnClick(Url.Action("btnSubmit_Click"), "SimpleForm1")
    ))
```
```html
<!-- ④ Core-RazorForms（TagHelper）：OnClick="方法名" -->
<f:SimpleForm ID="SimpleForm1" IsFluid="true" BodyPadding="10" Title="登录">
    <Items>
        <f:TextBox ID="tbxName" Label="用户名" Required="true" ShowRedStar="true" EmptyText="请输入用户名"></f:TextBox>
        <f:Button ID="btnSubmit" Text="提交" _ValidateForms="SimpleForm1" OnClick="btnSubmit_Click"></f:Button>
    </Items>
</f:SimpleForm>
```
```html
<!-- ⑤ Core-RazorPages（TagHelper）：OnClick="@Url.Handler(...)" + OnClickFields -->
<f:Button ID="btnSubmit" Text="提交" _ValidateForms="SimpleForm1"
          OnClick="@Url.Handler(&quot;btnSubmit_Click&quot;)" OnClickFields="SimpleForm1"></f:Button>
```
```html
<!-- ⑥ FineUIJava（Thymeleaf 方言）：全 kebab-case；validate-forms 直接写、逗号分隔（无下划线前缀）-->
<f:simple-form id="SimpleForm1" is-fluid="true" body-padding="10" title="登录">
    <f:items>
        <f:text-box id="tbxName" label="用户名" required="true" show-red-star="true" empty-text="请输入用户名"></f:text-box>
        <f:button id="btnSubmit" text="提交" validate-forms="SimpleForm1" on-click="btnSubmit_Click"></f:button>
    </f:items>
</f:simple-form>
```
```java
// FineUIJava 页面类：@FineUIPage 路由 + extends FineUIPageBase；void 处理器无需 return
@FineUIPage("form/login")
public class Login extends FineUIPageBase {
    public void btnSubmit_Click(Object sender, EventArgs e) { showNotify("通过"); }
}
```

> **注意 TagHelper 里 `_ValidateForms`**（下划线便捷形式）——`ValidateForms` 是 `string[]`，用下划线字符串写法传。多表单逗号分隔：`_ValidateForms="Form1,Form2"`。**FineUIJava 则直接写 `validate-forms="Form1,Form2"`（无下划线前缀）。**

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/form-layout.md](references/form-layout.md) | Form vs SimpleForm、LabelWidth/LabelAlign、多列 FormRow/ColumnWidths |
| [references/fields.md](references/fields.md) | 基础字段（TextBox/TextArea/NumberBox/DatePicker/DropDownList/CheckBox/RadioButtonList/CheckBoxList/TimePicker/Label/Hidden）+ 读值 |
| [references/advanced-fields.md](references/advanced-fields.md) | 高级字段：FileUpload（文件上传）、TriggerBox（触发器）、DropDownBox（下拉树/下拉表格/多选下拉）、HtmlEditor（富文本） |
| [references/validation.md](references/validation.md) | 必填/正则/比较/自定义校验、整表校验、服务端 MarkInvalid |

## 相关技能（Related Skills）

- `fineui-foundation`：写法判定、页面骨架、命名约定、RawHtml
- `fineui-grid`：Grid 单元格编辑用同一套字段控件作 `editor`

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js 用 `fieldLabel`/`required`（camelCase）；C# 用 `Label`/`Required`（PascalCase）；**FineUIJava 用 `label`/`required`（kebab-case），属性「值」仍 PascalCase（`text-mode="Password"`、`display-type="Switch"`、`compare-operator="GreaterThan"`）**。别把 F.js 的 `fieldLabel` 用到 C#（C# 是 `Label`）。
2. **单列用 SimpleForm、多列用 Form**：Form 的字段必须包在 `<Rows><f:FormRow><Items>` 里；SimpleForm 字段直接在 `<Items>`。
3. **必填红星**：`Required="true"` 需配 `ShowRedStar="true"` 才显示红星。
4. **整表校验在提交按钮上**：`ValidateForms="表单ID"`（Core-TagHelper 用 `_ValidateForms`；**FineUIJava 直接写 `validate-forms="表单ID"`，逗号分隔多表单，无下划线前缀**）；不是设在 Form 容器上。
5. **读值方式随写法不同**：Pro / Core-RazorForms 用控件字段（`tbxName.Text`）；Core-MVC / RazorPages 用回发参数或 `UIHelper.TextBox("id")`；**FineUIJava 用控件字段的 getter/setter（`tbxName.getValue()`、`ddl.getSelectedValue()`、`cbx.isChecked()`、`cbl.getSelectedValues()`）**。详见 [references/fields.md](references/fields.md)。
6. **Pro 变化事件采用推荐模式**：项目全局设置 `EnableImplicitChangeEvents="false"` 后，声明 `OnTextChanged`、`OnSelectedIndexChanged`、`OnCheckedChanged` 等服务端事件即可自动回发，无需再写 `AutoPostBack="true"`；显式 `AutoPostBack` 始终优先。其他控件回发只同步字段值，不会连带触发该字段的变化事件。
7. **绝不编造 API**：字段属性不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- **FineUIJava**：控件属性语义与 Core 一致（属性名 kebab-case、枚举值同 Core），客户端 F.js API 与 JS 端完全相同；查属性时参考 Core API 再按命名约定转 kebab-case。
