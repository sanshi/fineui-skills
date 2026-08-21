# 表单字段（Fields）

各字段类型的写法与读值。F.js 用 camelCase（`fieldLabel`/`required`），C# 用 PascalCase（`Label`/`Required`），**FineUIJava 用 kebab-case（`label`/`required`），属性「值」仍保持 PascalCase**（`text-mode="Password"`、`display-type="Switch"`）。

## 共同属性

| 概念 | F.js | C#（Pro / Core） | FineUIJava（Thymeleaf 方言） |
|------|------|------------------|------------------------------|
| 字段标签 | `fieldLabel` | `Label` | `label` |
| 隐藏标签 | `hideLabel: true` | `ShowLabel="false"` | `show-label="false"` |
| 必填 | `required: true` | `Required="true"` | `required="true"` |
| 红星 | `redStar: true` | `ShowRedStar="true"` | `show-red-star="true"` |
| 初始值/文本 | `value` | `Text`（输入类）/ `SelectedValue`（列表类）/ `SelectedDate`（日期） | `text` / `value` / `selected-value` |
| 占位提示 | `emptyText` | `EmptyText` | `empty-text` |
| 宽度 | `width` | `Width` | `width` |

> **FineUIJava 页面类**：`@FineUIPage("form/xxx")` + `extends FineUIPageBase`；控件字段手动声明（类名与控件同名时用全限定名 `com.fineui.java.core.controls.TextBox tbx;` 消歧）；事件处理器 `public void xxx_Click(Object sender, EventArgs e)`（返回 `void`，无需 return）。客户端 F.js API（`F.ui.xxx.getValue()` 等）四栈完全相同。

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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:text-box id="tbxName" label="用户名" required="true" show-red-star="true" empty-text="请输入"></f:text-box>
<f:text-box id="tbxPwd" label="密码" text-mode="Password" required="true" show-red-star="true"></f:text-box>
<f:text-area id="taDesc" label="描述" auto-grow-height="true" auto-grow-height-min="100" auto-grow-height-max="600"></f:text-area>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：千分位是 enable-commas（不是 commas）；评分模式 display-type="Rate" -->
<f:number-box label="0-9 整数" max-value="9" min-value="0" no-decimal="true" no-negative="true" required="true"></f:number-box>
<f:number-box label="金额" value="3000000" enable-commas="true" number-prefix="￥" decimal-precision="2"></f:number-box>
<f:number-box id="NumberBox2" display-type="Rate" value="3.5" rate-allow-half="true" rate-count="5"></f:number-box>
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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:date-picker id="dp1" label="开始日期" required="true" show-red-star="true" date-format-string="yyyy/MM/dd" empty-text="请选择"></f:date-picker>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：列表项 <f:list-item>，可直接为子元素，也可包在 <f:items> 里 -->
<f:drop-down-list id="ddl1" label="审批人" required="true" show-red-star="true" auto-select-first-item="false">
    <f:list-item text="老大甲" value="0"></f:list-item>
    <f:list-item text="不可选" value="2" enable-select="false"></f:list-item>
</f:drop-down-list>
```
```java
// FineUIJava 页面类：读值 getSelectedValue() / getText()，回填 setSelectedValue(...)
ddl1.setSelectedValue("0");
String val = ddl1.getSelectedValue();   // 选中值；ddl1.getText() = 选中文本
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
```html
<!-- FineUIJava（Thymeleaf 方言）：开关样式 display-type="Switch"；选中改变事件 on-checked-changed -->
<f:check-box id="cb1" show-label="false" text="复选框" checked="true"></f:check-box>
<f:check-box id="cb2" show-label="false" text="开关" display-type="Switch" on-checked-changed="cb2_CheckedChanged"></f:check-box>
```
```java
// FineUIJava 页面类：读 isChecked()，写 setChecked(...)
cb1.setChecked(!cb1.isChecked());
boolean on = cb1.isChecked();
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
```html
<!-- FineUIJava（Thymeleaf 方言）：竖排 column-vertical="true"；选中改变事件 on-selected-index-changed -->
<f:radio-button-list id="rbl1" label="性别" column-number="3">
    <f:radio-item text="男" value="1"></f:radio-item>
    <f:radio-item text="女" value="0"></f:radio-item>
</f:radio-button-list>
```
```java
// FineUIJava 页面类：读 getSelectedValue()，写 setSelectedValue(...)；运行时加项 addRadioItem(value, text, ...)
rbl1.setSelectedValue("1");
String sex = rbl1.getSelectedValue();
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
```html
<!-- FineUIJava（Thymeleaf 方言）：项用 <f:check-item>，预选 selected="true" -->
<f:check-box-list id="cbl1" label="兴趣" column-number="3">
    <f:check-item text="音乐" value="music" selected="true"></f:check-item>
    <f:check-item text="运动" value="sport"></f:check-item>
</f:check-box-list>
```
```csharp
// 读取选中值（Pro / RazorForms）
string[] selected = cbl1.SelectedValueArray;   // 如 ["music", "read"]
```
```java
// FineUIJava 页面类：读/写选中值用 List<String>（不是数组）
java.util.List<String> selected = cbl1.getSelectedValues();      // 如 ["music", "read"]
cbl1.setSelectedValues(java.util.List.of("music", "read"));
cbl1.addCheckItem("read", "阅读", true, false);                   // 运行时加项（值, 文本, ...）
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
```html
<!-- FineUIJava（Thymeleaf 方言）：范围属性是 min-time-text/max-time-text（不是 min-value/max-value）-->
<f:time-picker id="tp1" label="预约时间" increment="30" min-time-text="8:30" max-time-text="20:30" enable-edit="false"></f:time-picker>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：只读结果标签常配 encode-text="false" show-label="false"；HTML 原样输出用 text-raw-html -->
<f:label id="labResult" label="结果" text="初始文本"></f:label>
<f:label id="labHtml" text-raw-html="<span style='color:red'>红字</span>"></f:label>
```
```java
// FineUIJava 页面类：服务端更新 setText(...)
labResult.setText("新文本");
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
```html
<!-- FineUIJava（Thymeleaf 方言）：控件名 hidden-field -->
<f:hidden-field id="hfUserId"></f:hidden-field>
```
```java
// FineUIJava 页面类：服务端读写用 getText()/setText()；客户端仍是 F.ui.hfUserId.getValue()/setValue()
hfUserId.setText("67890");
String v = hfUserId.getText();
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

### FineUIJava —— 控件字段的 getter/setter（同 Core-RazorForms 思路，方法化）

```java
// FineUIJava 页面类：直接用手动声明的控件字段读写
String name = tbxName.getValue();                    // TextBox/TextArea
String v    = ddl1.getSelectedValue();               // DropDownList 值；ddl1.getText() = 选中文本
boolean on  = cb1.isChecked();                       // CheckBox
String sex  = rbl1.getSelectedValue();               // RadioButtonList
java.util.List<String> cs = cbl1.getSelectedValues();// CheckBoxList（List，非数组）
ddl1.setSelectedValue("1");                          // 回填
labResult.setText("用户名：" + name);                 // 更新只读标签
```

> DatePicker 的选中日期 getter 请以 `datepicker/DatePicker.java` 等示例为准（读值以 `getValue()`/`getText()` 系为主）。

## See also

- [validation.md](validation.md)：字段/整表校验、服务端 MarkInvalid
- [form-layout.md](form-layout.md)：容器与多列布局
