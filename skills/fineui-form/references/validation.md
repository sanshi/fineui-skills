# 表单校验（Validation）

分三层：**字段级校验**（在字段上）、**整表校验**（在提交按钮上）、**服务端标记无效**（后台）。

## 一、字段级校验

### 必填

```javascript
{ type: 'TextBox', fieldLabel: '用户名', required: true, redStar: true }   // F.js
```
```aspx
<f:TextBox runat="server" Label="用户名" Required="true" ShowRedStar="true" />   <%-- Pro / Core-TagHelper 同 --%>
```
```csharp
F.TextBox().Label("用户名").Required(true).ShowRedStar(true)                      // Core-MVC
```

### 正则（邮箱等）

C# 用 `RegexPattern` + `RegexMessage`（内置 `EMAIL` 等，或自定义正则）。

```aspx
<%-- Pro / Core-TagHelper --%>
<f:TextBox runat="server" Label="邮箱" RegexPattern="EMAIL" RegexMessage="请输入有效的邮箱地址" />
```
```csharp
// Core-MVC（Fluent）
F.TextBox().Label("邮箱").RegexPattern(RegexPattern.EMAIL).RegexMessage("请输入有效的邮箱地址")
```

### 比较校验（与另一字段比较）

`CompareControl`（对比目标）+ `CompareOperator`（`GreaterThan`/`Equal`/…）+ `CompareMessage`。

```javascript
// F.js
{ type: 'DatePicker', id: 'dp2', fieldLabel: '结束日期', required: true,
  compareControl: 'dp1', compareOperator: '>', compareMessage: '结束日期应大于开始日期！' }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:DatePicker runat="server" ID="dp2" Label="结束日期" CompareControl="dp1"
    CompareOperator="GreaterThan" CompareMessage="结束日期应大于开始日期" />
```
```csharp
// Core-MVC（Fluent）
F.DatePicker().ID("dp2").Label("结束日期").CompareControl("dp1")
    .CompareOperator(Operator.GreaterThan).CompareMessage("结束日期应大于开始日期")
```

### 自定义校验（JS 函数，返回 true 或错误字符串）

```javascript
// F.js —— 字段的 validator 函数
{ type: 'TextBox', id: 'textbox2', fieldLabel: '文本框 2（等于文本框 1）', required: true,
  validator: function () {
      return this.getValue() == F.ui.textbox1.getValue() ? true : '文本框 2 应等于文本框 1！';
  } }
```
```aspx
<%-- Pro / Core-TagHelper —— ValidatorFunction 指向页面内 JS 函数 --%>
<f:TextBox runat="server" ID="tbxPwd" Label="密码" ValidatorFunction="passwordValidator" />
```
```javascript
// 页面内 JS（C# 三模式通用），this 指向字段，返回 true 或错误字符串
function passwordValidator() {
    return $.trim(this.getValue()).length === 6 ? true : '密码必须为 6 个字符！';
}
```
```csharp
// Core-MVC（Fluent）
F.TextBox().ID("tbxPwd").Label("密码").ValidatorFunction("passwordValidator")
```

## 二、整表校验（提交按钮触发，客户端）

在**提交按钮**上声明要校验的表单；校验通过才继续。多表单逗号分隔。

```javascript
// F.js —— validateForm
{ type: 'Button', text: '提交', validateForm: 'form1', handler: function () { showNotify('通过'); } }
```
```aspx
<%-- Pro --%>
<f:Button runat="server" Text="提交" ValidateForms="Form1" ValidateTarget="Top" OnClick="btnSubmit_Click" />
<f:Button runat="server" Text="提交两个表单" ValidateForms="Form1,Form2" OnClick="btnSubmitAll_Click" />
```
```csharp
// Core-MVC（Fluent）
F.Button().Text("提交").ValidateForms("Form1").OnClick(Url.Action("btnSubmit_Click"), "Form1")
```
```html
<!-- Core-RazorForms（TagHelper）：_ValidateForms 下划线便捷形式（ValidateForms 是 string[]）-->
<f:Button Text="提交" _ValidateForms="Form1" OnClick="btnSubmit_Click"></f:Button>
<!-- Core-RazorPages -->
<f:Button Text="提交" _ValidateForms="Form1" OnClick="@Url.Handler(&quot;btnSubmit_Click&quot;)" OnClickFields="Form1"></f:Button>
```

## 三、服务端标记字段无效（后台业务校验）

客户端校验通过后，后台还可根据业务把某字段标记为无效（如“用户名已被占用”）。

```csharp
// Pro / Core-RazorForms —— 控件字段
protected void btnRegister_Click(object sender, EventArgs e) {
    if (tbxUserName.Text == "admin")
        tbxUserName.MarkInvalid(String.Format("{0} 是保留字，请另选！", tbxUserName.Text));
}
```
```csharp
// Core-MVC / RazorPages —— UIHelper
public IActionResult btnRegister_Click(string userName) {           // MVC；RazorPages 为 OnPostBtnRegister_Click
    if (userName == "admin")
        UIHelper.TextBox("tbxUserName").MarkInvalid(userName + " 是保留字，请另选！");
    return UIHelper.Result();
}
```
```javascript
// F.js —— 客户端 markInvalid
F.ui.tbxUserName.markInvalid('用户名已被占用！');
```

> Pro 还可弹框并聚焦：`Alert.Show(msg, "", tbxUserName.GetFocusReference(true, 200));`

## 四、重置表单

```javascript
F.ui.form1.reset();                          // F.js
```
```csharp
// Pro —— 客户端重置（Page_Load 内绑定）
btnReset.OnClientClick = SimpleForm1.GetResetReference();
```

## See also

- [fields.md](fields.md)：字段类型与读值
- [form-layout.md](form-layout.md)：容器与布局
