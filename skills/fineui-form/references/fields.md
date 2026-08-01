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
千分位 `Commas`；前后缀 `NumberPrefix`/`NumberSuffix`；显示模式 `DisplayType`（`default`/`progress`/`rate`）。

```javascript
// F.js
{ type: 'NumberBox', fieldLabel: '0-9 整数', maxValue: 9, minValue: 0, noDecimal: true, noNegative: true, required: true }
// 千分位 + 前缀：
{ type: 'NumberBox', fieldLabel: '金额', value: 12345.67, commas: true, numberPrefix: '￥', decimalPrecision: 2 }
// 评分模式：
{ type: 'NumberBox', fieldLabel: '评分', value: 3.5, displayType: 'rate', rateAllowHalf: true, rateCount: 5 }
```
```aspx
<%-- Pro --%>
<f:NumberBox runat="server" Label="0-9 整数" MaxValue="9" MinValue="0" NoDecimal="true" NoNegative="true" Required="true" />
<f:NumberBox runat="server" Label="两位小数" MaxValue="1" MinValue="0" DecimalPrecision="2" Increment="0.01" />
<f:NumberBox runat="server" Label="金额" Commas="true" NumberPrefix="￥" DecimalPrecision="2" />
```
```csharp
// Core-MVC（Fluent）
F.NumberBox().Label("0-9 整数").MaxValue(9).MinValue(0).NoDecimal(true).NoNegative(true).Required(true)
F.NumberBox().Label("金额").Commas(true).NumberPrefix("￥").DecimalPrecision(2)
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

## CheckBoxList（复选框列表，多选）

多个互不互斥的复选框；`ColumnNumber` 控制列数；`DisplayType="Switch"` 开关样式；可 `DataSource` 绑定。读值：`SelectedValueArray`（字符串数组）。

```javascript
// F.js —— data 二维数组 [[值,文本], ...]；value 为预选值数组
{ type: 'CheckBoxList', id: 'cbl1', fieldLabel: '兴趣', columnNumber: 3,
  data: [['music', '音乐'], ['sport', '运动'], ['read', '阅读']], value: ['music', 'read'] }
```
```aspx
<%-- Pro --%>
<f:CheckBoxList runat="server" ID="cbl1" Label="兴趣" ColumnNumber="3" Required="true">
    <f:CheckItem Text="音乐" Value="music" />
    <f:CheckItem Text="运动" Value="sport" />
    <f:CheckItem Text="阅读" Value="read" />
</f:CheckBoxList>
```
```csharp
// Core-MVC（Fluent）
F.CheckBoxList().ID("cbl1").Label("兴趣").ColumnNumber(3)
    .Items(F.CheckItem().Text("音乐").Value("music"), F.CheckItem().Text("运动").Value("sport"))
```
```html
<!-- Core-TagHelper -->
<f:CheckBoxList ID="cbl1" Label="兴趣" ColumnNumber="3">
    <f:CheckItem Text="音乐" Value="music" />
    <f:CheckItem Text="运动" Value="sport" />
</f:CheckBoxList>
```
```csharp
// 读取选中值（Pro / RazorForms）
string[] selected = cbl1.SelectedValueArray;   // 如 ["music", "read"]
```

## TimePicker（时间选择）

固定步长的时间下拉列表（区别于 DatePicker 的 time 模式）。`Increment` 间隔分钟数；`MinValue`/`MaxValue` 限制范围（格式 `HH:mm`）。

```javascript
// F.js
{ type: 'TimePicker', id: 'tp1', fieldLabel: '预约时间', minValue: '09:00', maxValue: '18:00', increment: 60 }
```
```aspx
<%-- Pro --%>
<f:TimePicker runat="server" ID="tp1" Label="预约时间" MinValue="09:00" MaxValue="18:00" Increment="60" />
```
```csharp
// Core-MVC（Fluent）
F.TimePicker().ID("tp1").Label("预约时间").MinValue("09:00").MaxValue("18:00").Increment(60)
```
```html
<!-- Core-TagHelper -->
<f:TimePicker ID="tp1" Label="预约时间" MinValue="09:00" MaxValue="18:00" Increment="60"></f:TimePicker>
```

## Label（只读文本标签）

展示只读文本，常用于显示计算结果或回显。`setValue()` 动态更新；可信 HTML 用 `F.rawHtml(...)` / `new RawHtml(...)`。

```javascript
// F.js
{ type: 'Label', id: 'labResult', fieldLabel: '结果', value: '初始文本' }
// 动态更新：F.ui.labResult.setValue('新文本');
```
```aspx
<%-- Pro --%>
<f:Label runat="server" ID="labResult" Label="结果" Text="初始文本" />
```
```csharp
// Core-MVC（Fluent）
F.Label().ID("labResult").Label("结果").Text("初始文本")
// 服务端更新：UIHelper.Label("labResult").Text("新文本");
```
```html
<!-- Core-TagHelper -->
<f:Label ID="labResult" Label="结果" Text="初始文本"></f:Label>
```
```csharp
// Pro / RazorForms 服务端更新
labResult.Text = "新文本";
```

## Hidden（隐藏字段）

不可见的表单字段，用于在客户端记录状态、传递参数。`setValue()`/`getValue()` 读写。

```javascript
// F.js
{ type: 'Hidden', id: 'hfUserId', value: '12345' }
// 读写：F.ui.hfUserId.getValue(); F.ui.hfUserId.setValue('67890');
```
```aspx
<%-- Pro --%>
<f:Hidden runat="server" ID="hfUserId" Value="12345" />
```
```csharp
// Core-MVC（Fluent）
F.Hidden().ID("hfUserId").Value("12345")
// 服务端读写（Pro/RazorForms）：hfUserId.Value = "67890"; string v = hfUserId.Value;
```
```html
<!-- Core-TagHelper -->
<f:Hidden ID="hfUserId" Value="12345"></f:Hidden>
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
