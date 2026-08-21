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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:text-box label="用户名" required="true" show-red-star="true"></f:text-box>
```

### 正则（邮箱等）

C# 用 `RegexPattern` + `RegexMessage`（内置 `EMAIL` 等，或自定义正则）；FineUIJava 用 `regex-pattern` + `regex-message`。

```aspx
<%-- Pro / Core-TagHelper --%>
<f:TextBox runat="server" Label="邮箱" RegexPattern="EMAIL" RegexMessage="请输入有效的邮箱地址" />
```
```csharp
// Core-MVC（Fluent）
F.TextBox().Label("邮箱").RegexPattern(RegexPattern.EMAIL).RegexMessage("请输入有效的邮箱地址")
```
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:text-box label="邮箱" regex-pattern="EMAIL" regex-message="请输入有效的邮箱地址！"></f:text-box>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：compare-operator 值保持 PascalCase -->
<f:date-picker id="dp2" label="结束日期" compare-control="dp1"
    compare-operator="GreaterThan" compare-message="结束日期应该大于开始日期"></f:date-picker>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：validator-function 指向页面脚本槽里的 JS 函数（函数体与上文 passwordValidator 完全相同，不重复贴）-->
<f:text-box id="tbxPwd" label="密码" text-mode="Password" validator-function="passwordValidator"></f:text-box>
<!-- 函数放页面脚本槽（须在 f:scripts 之后）：
<th:block layout:fragment="script"><script> /* function passwordValidator() { ... } —— 同 F.js */ </script></th:block> -->
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
```html
<!-- FineUIJava（Thymeleaf 方言）：validate-forms 直接写、逗号分隔多表单——不需要 Core 的下划线便捷形式 -->
<f:button text="提交" validate-forms="Form1" validate-target="Top" on-click="btnSubmit_Click"></f:button>
<f:button text="提交两个表单" validate-forms="Form1,Form2" on-click="btnSubmitAll_Click"></f:button>
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
```java
// FineUIJava 页面类 —— 控件字段的 markInvalid（void 处理器，无需 return）
public void btnRegister_Click(Object sender, EventArgs e) {
    if ("admin".equals(tbxUserName.getValue())) {
        tbxUserName.markInvalid(tbxUserName.getValue() + " 是保留字，请另外选择！");
    } else {
        showNotify("用户名：" + tbxUserName.getValue());
    }
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
```html
<!-- FineUIJava —— 直接在重置按钮的 on-client-click 里调 F.js reset（客户端，无需回发）-->
<f:button id="btnReset" text="重置" on-client-click="F.ui.SimpleForm1.reset();"></f:button>
<f:button id="btnResetAll" text="重置两个表单" on-client-click="F.ui.Form1.reset();F.ui.Form2.reset();"></f:button>
```

## See also

- [fields.md](fields.md)：字段类型与读值
- [form-layout.md](form-layout.md)：容器与布局
