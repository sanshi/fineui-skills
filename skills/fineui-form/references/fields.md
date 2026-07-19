# 表单字段（Fields）

各字段类型的写法与读值。F.js 用 camelCase（`fieldLabel`/`required`），C# 用 PascalCase（`Label`/`Required`）。

## 共同属性

| 概念 | F.js | C#（Pro / Core） |
|------|------|------------------|
| 字段标签 | `fieldLabel` | `Label` |
| 隐藏标签 | `hideLabel: true` | `ShowLabel="false"` |
| 必填 | `required: true` | `Required="true"` |
| 红星 | `redStar: true` | `ShowRedStar="true"` |
| 初始值/文本 | `value` | `Text`（输入类）/ `SelectedValue`（列表类）/ `SelectedDate`（日期） |
| 占位提示 | `emptyText` | `EmptyText` |
| 宽度 | `width` | `Width` |

---

## TextBox / TextArea

密码框：F.js `inputType: 'password'`；C# `TextMode="Password"`。TextArea 自动增高：`AutoGrowHeight`。

```javascript
// F.js
{ type: 'TextBox', id: 'tbxName', fieldLabel: '用户名', required: true, emptyText: '请输入' }
{ type: 'TextArea', id: 'taDesc', fieldLabel: '描述', autoGrowHeight: true }
```
```aspx
<%-- Pro --%>
<f:TextBox runat="server" ID="tbxName" Label="用户名" Required="true" ShowRedStar="true" EmptyText="请输入" />
<f:TextBox runat="server" ID="tbxPwd" Label="密码" TextMode="Password" Required="true" />
<f:TextArea runat="server" ID="taDesc" Label="描述" AutoGrowHeight="true" AutoGrowHeightMin="100" AutoGrowHeightMax="600" />
```
```csharp
// Core-MVC（Fluent）
F.TextBox().ID("tbxName").Label("用户名").Required(true).ShowRedStar(true).EmptyText("请输入"),
F.TextBox().ID("tbxPwd").Label("密码").TextMode(TextMode.Password).Required(true)
```
```html
<!-- Core-TagHelper -->
<f:TextBox ID="tbxName" Label="用户名" Required="true" ShowRedStar="true" EmptyText="请输入"></f:TextBox>
<f:TextBox ID="tbxPwd" Label="密码" TextMode="Password" Required="true"></f:TextBox>
```

## NumberBox

`MaxValue`/`MinValue`/`NoDecimal`/`NoNegative`/`DecimalPrecision`/`Increment`。

```javascript
// F.js
{ type: 'NumberBox', fieldLabel: '0-9 整数', maxValue: 9, minValue: 0, noDecimal: true, noNegative: true, required: true }
```
```aspx
<%-- Pro --%>
<f:NumberBox runat="server" Label="0-9 整数" MaxValue="9" MinValue="0" NoDecimal="true" NoNegative="true" Required="true" />
<f:NumberBox runat="server" Label="两位小数" MaxValue="1" MinValue="0" DecimalPrecision="2" Increment="0.01" />
```
```csharp
// Core-MVC（Fluent）
F.NumberBox().Label("0-9 整数").MaxValue(9).MinValue(0).NoDecimal(true).NoNegative(true).Required(true)
```
```html
<!-- Core-TagHelper -->
<f:NumberBox Label="0-9 整数" MaxValue="9" MinValue="0" NoDecimal="true" NoNegative="true" Required="true" />
```

## DatePicker

`DateFormatString`（显示格式）；比较校验见 [validation.md](validation.md)。

```javascript
// F.js
{ type: 'DatePicker', id: 'dp1', fieldLabel: '开始日期', required: true, value: '2015-06-02' }
```
```aspx
<%-- Pro --%>
<f:DatePicker runat="server" ID="dp1" Label="开始日期" Required="true" DateFormatString="yyyy/MM/dd" EmptyText="请选择" />
```
```csharp
// Core-MVC（Fluent）
F.DatePicker().ID("dp1").Label("开始日期").Required(true).DateFormatString("yyyy/MM/dd").SelectedDate(DateTime.Now)
```
```html
<!-- Core-TagHelper -->
<f:DatePicker ID="dp1" Label="开始日期" Required="true" DateFormatString="yyyy/MM/dd"></f:DatePicker>
```

## DropDownList

选项用 `ListItem`（Value 不能为空字符串）；`AutoSelectFirstItem`；也可 `DataSource` 绑定。

```javascript
// F.js —— data 二维数组 [[值,文本], ...]
{ type: 'DropDownList', id: 'ddl1', fieldLabel: '审批人', required: true, data: [['0', '老大甲'], ['1', '老大乙']] }
```
```aspx
<%-- Pro --%>
<f:DropDownList runat="server" ID="ddl1" Label="审批人" Required="true" EmptyText="请选择" AutoSelectFirstItem="false">
    <f:ListItem Text="老大甲" Value="0" />
    <f:ListItem Text="不可选" Value="2" EnableSelect="false" />
</f:DropDownList>
```
```csharp
// Core-MVC（Fluent）
F.DropDownList().ID("ddl1").Label("审批人").Required(true).AutoSelectFirstItem(false)
    .Items(F.ListItem().Text("老大甲").Value("0"), F.ListItem().Text("老大乙").Value("1"))
```
```html
<!-- Core-TagHelper -->
<f:DropDownList ID="ddl1" Label="审批人" Required="true">
    <f:ListItem Text="老大甲" Value="0" />
</f:DropDownList>
```

## CheckBox

`Text`=旁边文字，`Checked`=初始选中；开关样式 `DisplayType="Switch"`。

```javascript
// F.js
{ type: 'CheckBox', id: 'cb1', inputLabel: '复选框', checked: true }
```
```aspx
<%-- Pro --%>
<f:CheckBox runat="server" ID="cb1" Label="复选框" Text="复选框" Checked="true" />
<f:CheckBox runat="server" ID="cb2" Label="开关" Text="开关" DisplayType="Switch" />
```
```csharp
// Core-MVC（Fluent）
F.CheckBox().ID("cb1").ShowLabel(false).Text("复选框").Checked(true)
```
```html
<!-- Core-TagHelper -->
<f:CheckBox ID="cb1" ShowLabel="false" Text="复选框" Checked="true"></f:CheckBox>
```

## RadioButtonList

选项用 `RadioItem`；多列 `ColumnNumber`、竖排 `ColumnVertical`；可 `DataSource` 绑定。

```aspx
<%-- Pro --%>
<f:RadioButtonList runat="server" ID="rbl1" Label="性别" ColumnNumber="3" Required="true">
    <f:RadioItem Text="男" Value="1" />
    <f:RadioItem Text="女" Value="0" />
</f:RadioButtonList>
```
```csharp
// Core-MVC（Fluent）—— 静态项 或 数据源绑定
F.RadioButtonList().ID("rbl1").Label("性别").Items(F.RadioItem().Text("男").Value("1"), F.RadioItem().Text("女").Value("0")),
F.RadioButtonList().ID("rbl2").DataTextField("Name").DataValueField("Id").DataSource(ViewBag.Rbl2DataSource).SelectedValue(ViewBag.Rbl2Value)
```
```html
<!-- Core-TagHelper -->
<f:RadioButtonList ID="rbl1" Label="性别" ColumnNumber="3">
    <f:RadioItem Text="男" Value="1" />
    <f:RadioItem Text="女" Value="0" />
</f:RadioButtonList>
```

---

## 读取字段值（各写法不同）

### F.js（客户端）

```javascript
F.ui.tbxName.getValue();      // 输入类
F.ui.ddl1.getValue();         // 下拉列表选中值
F.ui.form1.reset();           // 重置整表
```

### Pro / Core-RazorForms —— 直接用控件字段

```csharp
string name = tbxName.Text;                 // TextBox/TextArea
string v    = ddl1.SelectedValue;           // DropDownList 值；ddl1.Text = 选中文本
DateTime? d = dp1.SelectedDate;             // DatePicker（可空）
string sex  = rbl1.SelectedValue;           // RadioButtonList
ddl1.SelectedValue = "1";                   // 回填
```

### Core-MVC / RazorPages —— 回发参数 或 UIHelper

```csharp
// 回发参数：按钮 .OnClick(url, new Parameter("name", "F.ui.tbxName.getValue()"))
public IActionResult btnSubmit_Click(string name) { ... }          // MVC
public IActionResult OnPostBtnSubmit_Click(string name) { ... }    // RazorPages

// 或用 UIHelper 读/写控件
var tbx = UIHelper.TextBox("tbxName");
UIHelper.Label("labResult").Text("用户名：" + name);
```

## See also

- [validation.md](validation.md)：字段/整表校验、服务端 MarkInvalid
- [form-layout.md](form-layout.md)：容器与多列布局
