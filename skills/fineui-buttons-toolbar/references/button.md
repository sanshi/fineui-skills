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
```html
<!-- FineUIJava（Thymeleaf 方言）：属性名 kebab-case，值仍 PascalCase -->
<f:button id="btnSuccess" text="成功按钮" button-color="Success"></f:button>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：icon / icon-font / icon-url / icon-align，值 PascalCase -->
<f:button text="邮件" icon="Email"></f:button>
<f:button text="首页" icon-font="_Home"></f:button>
<f:button text="右图标" icon="Star" icon-align="Right"></f:button>
<f:button icon-url="~/res/images/16/1.png"></f:button>   <!-- 仅图标 -->
```

## 尺寸与徽标

尺寸：F.js `scale: 'medium'/'large'`；C# `Size="Small/Medium/Large"`。徽标：`Badge="true" BadgeText="10" BadgeType="Success"`（BadgeType 用语义色）；徽标动画 `BadgeAnimationType="Processing/Fade/Move/Shake"`。

```aspx
<f:Button runat="server" Text="大按钮" Size="Large" />
<f:Button runat="server" Text="消息" Badge="true" BadgeText="10" BadgeType="Warning" />
```
```html
<!-- FineUIJava（Thymeleaf 方言）：size / badge / badge-text / badge-type / badge-animation-type -->
<f:button text="大按钮" size="Large"></f:button>
<f:button text="消息" badge="true" badge-text="10" badge-type="Warning" badge-animation-type="Processing"></f:button>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：on-click="方法名"（结构同 Core-RazorForms） -->
<f:button id="btnServer" text="服务端" on-click="btnServer_Click"></f:button>
```
```java
// FineUIJava 页面类：处理器 (Object sender, EventArgs e)，返回 void（无需 return UIHelper.Result()）
public void btnServer_Click(Object sender, EventArgs e) { showNotify("服务端事件"); }
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
```html
<!-- FineUIJava（Thymeleaf 方言）：on-client-click 内联脚本，或 <f:listeners> 挂 click -->
<f:button text="客户端" on-client-click="alert('客户端事件');"></f:button>
<f:button text="客户端2">
    <f:listeners><f:listener event="click" handler="onBtnClick" /></f:listeners>
</f:button>
```
> 服务端改按钮的客户端事件：`btn.setOnClientClick(Alert.getShowInTopReference("..."))`（`Alert.getShowInTopReference` 生成 iframe 内跨顶层弹框的脚本字符串）。

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
```html
<!-- FineUIJava（Thymeleaf 方言）：confirm-text + confirm-target，确认后回发 on-click 处理器 -->
<f:button text="删除" icon="Delete" confirm-text="确定删除？" confirm-target="Top" on-click="btnDelete_Click"></f:button>
```
```javascript
// F.js / 纯客户端：用 F.confirm 回调（Java 客户端同 F.js）
{ type: 'Button', text: '删除', handler: function () {
    F.confirm({ message: '确定删除？', messageIcon: 'question', ok: function () { doDelete(); } });
} }
```
> `ConfirmTitle`（标题）、`ConfirmIcon`（图标，默认 Warning）也是控件属性（Java：`confirm-title` / `confirm-icon`）。

## Type：提交/重置按钮（表单）

C# `Type` 是 `ButtonType{Button,Submit,Reset}`（**不是颜色**）。表单里：

```csharp
// Core-MVC —— 提交按钮 + 校验表单
F.Button().Type(ButtonType.Submit).ValidateForms("SimpleForm1").OnClick(Url.Action("btnLogin_Click"), "SimpleForm1").Text("登录")
F.Button().Type(ButtonType.Reset).Text("重置")
```
```html
<!-- FineUIJava（Thymeleaf 方言）：type 值保持 PascalCase（Submit/Reset）-->
<f:button type="Submit" validate-forms="SimpleForm1" on-click="btnLogin_Click" text="登录"></f:button>
<f:button type="Reset" text="重置"></f:button>
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
```html
<!-- FineUIJava（Thymeleaf 方言）：<f:link-button>，on-click / on-client-click / enabled / confirm-text -->
<f:link-button id="LinkButton3" text="服务端事件" on-click="LinkButton3_Click"></f:link-button>
<f:link-button id="LinkButton1" text="客户端事件" on-client-click="clickIt();"></f:link-button>
```
```java
// FineUIJava 页面类：控件字段用全限定名或 import；启用/禁用同 setEnabled
com.fineui.java.core.controls.LinkButton LinkButton1;
public void btnChangeEnable_Click(Object sender, EventArgs e) { LinkButton1.setEnabled(!LinkButton1.isEnabled()); }
```

## 其他常用（F.js 方法）

```javascript
F.ui.btn.enable(); F.ui.btn.disable(); F.ui.btn.isDisabled();
F.ui.btn.setText('新文本'); F.ui.btn.setTooltip('提示');
// 切换按下状态：{ type:'Button', enablePress:true, pressed:true } → F.ui.btn.toggle()
```

## See also

- [menu.md](menu.md)：下拉菜单
- `fineui-panel`：Toolbar 工具栏（放在容器 `<Toolbars>`）
- `fineui-window`：Confirm 确认框、消息框

---

## ButtonGroup 按钮分组

多个按钮拼接显示（无间距），支持横向/纵向、互斥按下、多按下等模式。

关键属性（F.js camelCase / C# PascalCase / Java kebab-case）：
- 纵向显示（默认横向）：`vertical` / `Vertical` / `vertical`
- 启用按下状态分组（互斥单选）：`pressGroup` / **`EnablePressGroup`** / `enable-press-group`（**C# 是 `EnablePressGroup`，不是 `PressGroup`**）
- 允许多个按钮同时按下：`allowMultiPress` / `AllowMultiPress` / `allow-multi-press`
- 允许分组中没有按钮处于按下状态：`allowNonePress` / `AllowNonePress` / `allow-none-press`
- 按钮子项需设 `enablePress` / `EnablePress` / `enable-press`（**已废弃别名 `enableToggle` 不要用**）才能参与按下状态；`pressed` / `Pressed` / `pressed` 初始按下
- 按下状态改变事件：`presschange`（F.js）/ **`OnPressChanged`**（C#）/ `on-press-changed`（Java）

```javascript
// F.js —— 基础分组（无间距拼接）
F.create({ type: 'ButtonGroup', renderTo: '#wrap', items: [
    { type: 'Button', text: '左对齐', iconFont: 'align-left' },
    { type: 'Button', text: '居中', iconFont: 'align-center' },
    { type: 'Button', text: '右对齐', iconFont: 'align-right' }
] });

// F.js —— 互斥按下（单选，pressGroup: true）
F.create({ type: 'ButtonGroup', renderTo: '#wrap', pressGroup: true, items: [
    { type: 'Button', text: '日', enablePress: true, pressed: true },
    { type: 'Button', text: '周', enablePress: true },
    { type: 'Button', text: '月', enablePress: true }
], listeners: { presschange: function (event, item, pressed) {
    if (pressed) { showNotify('选中：' + item.text); }
} } });

// F.js —— 多按下（allowMultiPress: true，工具栏开关组合）
F.create({ type: 'ButtonGroup', renderTo: '#wrap', allowMultiPress: true, items: [
    { type: 'Button', iconFont: 'bold', enablePress: true },
    { type: 'Button', iconFont: 'italic', enablePress: true },
    { type: 'Button', iconFont: 'underline', enablePress: true }
] });

// F.js —— 纵向显示
F.create({ type: 'ButtonGroup', renderTo: '#wrap', vertical: true, items: [
    { type: 'Button', text: '上移', iconFont: 'arrow-up' },
    { type: 'Button', text: '下移', iconFont: 'arrow-down' }
] });
```
```aspx
<%-- Pro / Core-TagHelper：属性是 EnablePressGroup（不是 PressGroup） --%>
<f:ButtonGroup runat="server" ID="bg1" EnablePressGroup="true">
    <Items>
        <f:Button Text="日" EnablePress="true" Pressed="true" runat="server" />
        <f:Button Text="周" EnablePress="true" runat="server" />
        <f:Button Text="月" EnablePress="true" runat="server" />
    </Items>
</f:ButtonGroup>
```
```csharp
// Core-MVC（Fluent）
@(F.ButtonGroup().ID("bg1").EnablePressGroup(true)
    .Items(
        F.Button().Text("日").EnablePress(true).Pressed(true),
        F.Button().Text("周").EnablePress(true),
        F.Button().Text("月").EnablePress(true)
    ))
```
```html
<!-- FineUIJava（Thymeleaf 方言）：enable-press-group；on-press-changed 绑按下改变事件 -->
<f:button-group id="bg1" enable-press-group="true" on-press-changed="bg1_PressChanged">
    <f:button text="日" enable-press="true" pressed="true"></f:button>
    <f:button text="周" enable-press="true"></f:button>
    <f:button text="月" enable-press="true"></f:button>
</f:button-group>
```
```java
// FineUIJava 页面类：处理器读分组子项的按下状态
ButtonGroup bg1;
public void bg1_PressChanged(Object sender, EventArgs e) {
    for (Button btn : bg1.getItems()) { if (btn.isPressed()) { /* btn.getText() */ } }
}
```
