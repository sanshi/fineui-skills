# Button 按钮

## 语义颜色

值：`Primary`（主）/ `Success`（成功·绿）/ `Danger`（危险·红）/ `Warning`（警告·橙）/ `Info`（信息·蓝）。不设为默认灰。

```javascript
// F.js —— color 小写
{ type: 'Button', text: '主按钮', color: 'primary' }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Button runat="server" Text="成功按钮" ButtonColor="Success" />
```
```csharp
// Core-MVC（Fluent）
F.Button().Text("危险按钮").ButtonColor(ButtonColor.Danger)
```

## 图标

`Icon`（内置枚举，如 `Email`/`Star`/`Delete`）、`IconFont`（字体图标，如 `_Home`/`_Car`）、`IconUrl`（自定义图片）；`IconAlign="Right"` 图标在右。仅图标按钮不设 `Text`。

```javascript
// F.js
{ type: 'Button', text: '首页', iconFont: 'home' }
{ type: 'Button', text: '右图标', icon: '../res/x.png', iconAlign: 'right' }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Button runat="server" Text="邮件" Icon="Email" />
<f:Button runat="server" Text="首页" IconFont="_Home" />
<f:Button runat="server" IconUrl="~/res/images/16/1.png" />   <%-- 仅图标 --%>
```
```csharp
// Core-MVC（Fluent）
F.Button().Text("首页").IconFont(IconFont._Home)
F.Button().Text("右图标").Icon(Icon.Star).IconAlign(IconAlign.Right)
```

## 尺寸与徽标

尺寸：F.js `scale: 'medium'/'large'`；C# `Size="Small/Medium/Large"`。徽标：`Badge="true" BadgeText="10" BadgeType="Success"`（BadgeType 用语义色）。

```aspx
<f:Button runat="server" Text="大按钮" Size="Large" />
<f:Button runat="server" Text="消息" Badge="true" BadgeText="10" BadgeType="Warning" />
```

## 服务端点击 vs 客户端点击

### 服务端点击（回发）

```javascript
// F.js 无"服务端"概念，handler 即客户端回调
```
```aspx
<%-- Pro / Core-RazorForms：OnClick="方法名" --%>
<f:Button runat="server" Text="服务端" OnClick="btnServer_Click" />
```
```csharp
// Core-MVC：Url.Action（可带回发参数）
@(F.Button().Text("服务端").OnClick(Url.Action("btnServer_Click"), new Parameter("v", "F.ui.tbx1.getValue()")))
// Core-RazorPages：@Url.Handler + OnClickParameter1
// <f:Button Text="服务端" OnClick="@Url.Handler("btnServer_Click")" OnClickParameter1="@(new Parameter("v","..."))" />
```
```csharp
// 后台：MVC/RazorPages 用 action/OnPost；Pro/RazorForms 用事件方法
public IActionResult btnServer_Click() { ShowNotify("服务端事件"); return UIHelper.Result(); }  // MVC
// RazorPages: public IActionResult OnPostBtnServer_Click() {...}
// Pro/RazorForms: protected void btnServer_Click(object sender, EventArgs e) { ShowNotify("..."); }
```

### 客户端点击（不回发）

```javascript
// F.js —— handler
{ type: 'Button', text: '客户端', handler: function () { F.alert('客户端事件'); } }
```
```aspx
<%-- Pro / Core-TagHelper：OnClientClick + EnablePostBack="false"，或 Listener --%>
<f:Button runat="server" Text="客户端" OnClientClick="alert('客户端事件');" EnablePostBack="false" />
<f:Button runat="server" Text="客户端2" EnablePostBack="false">
    <Listeners><f:Listener Event="click" Handler="onBtnClick" /></Listeners>
</f:Button>
```
```csharp
// Core-MVC（Fluent）
F.Button().Text("客户端").OnClientClick("alert('客户端事件');")
F.Button().Text("客户端2").Listener("click", "onBtnClick")
```

## 确认按钮（点击先弹确认）

声明式 `ConfirmText` + `ConfirmTarget`，确认后才回发。

```aspx
<%-- Pro / Core-TagHelper --%>
<f:Button runat="server" Text="删除" Icon="Delete" ConfirmText="确定删除？" ConfirmTarget="Top" OnClick="btnDelete_Click" />
```
```csharp
// Core-MVC（Fluent）
F.Button().Text("删除").ConfirmText("确定删除？").ConfirmTarget(Target.Top).OnClick(Url.Action("btnDelete_Click"))
```
```javascript
// F.js / 纯客户端：用 F.confirm 回调
{ type: 'Button', text: '删除', handler: function () {
    F.confirm({ message: '确定删除？', messageIcon: 'question', ok: function () { doDelete(); } });
} }
```
> `ConfirmTitle`（标题）、`ConfirmIcon`（图标，默认 Warning）也是控件属性。

## Type：提交/重置按钮（表单）

C# `Type` 是 `ButtonType{Button,Submit,Reset}`（**不是颜色**）。表单里：

```csharp
// Core-MVC —— 提交按钮 + 校验表单
F.Button().Type(ButtonType.Submit).ValidateForms("SimpleForm1").OnClick(Url.Action("btnLogin_Click"), "SimpleForm1").Text("登录")
F.Button().Type(ButtonType.Reset).Text("重置")
```

## LinkButton 超链接按钮

样式像超链接的按钮，同样有 `OnClick`（服务端）/ `OnClientClick`+`EnablePostBack="false"`（客户端）/ `Enabled`。

```aspx
<%-- Pro / Core-TagHelper --%>
<f:LinkButton runat="server" ID="LinkButton3" Text="服务端事件" OnClick="LinkButton3_Click" />
<f:LinkButton runat="server" ID="LinkButton1" Text="客户端事件" OnClientClick="clickIt();" EnablePostBack="false" />
```
```csharp
// Core-MVC（Fluent）
F.LinkButton().ID("LinkButton3").Text("服务端事件").OnClick(Url.Action("LinkButton3_Click"))
```

## 其他常用（F.js 方法）

```javascript
F.ui.btn.enable(); F.ui.btn.disable(); F.ui.btn.isDisabled();
F.ui.btn.setText('新文本'); F.ui.btn.setTooltip('提示');
// 切换按下状态：{ type:'Button', enableToggle:true, pressed:true } → F.ui.btn.toggle()
```

## See also

- [menu.md](menu.md)：下拉菜单
- `fineui-layout`：Toolbar 工具栏
- `fineui-window`：Confirm 确认框、消息框
