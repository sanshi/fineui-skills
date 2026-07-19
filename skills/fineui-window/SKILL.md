---
name: fineui-window
description: >
  帮助开发者使用 FineUI 的窗口与消息框：Window（内联内容 / iframe 弹窗）、打开/关闭窗口、
  iframe 子页回传数据给父页（closeArgument）、以及 MessageBox（Alert 对话框 / Confirm 确认框 / Notify 通知框）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper）。
  Trigger phrases（触发词）: "FineUI 窗口", "F.Window", "弹窗", "对话框", "MessageBox",
  "Alert.Show", "F.alert", "F.confirm", "确认框", "通知框", "Notify", "ShowNotify",
  "iframe 窗口", "关闭窗口", "回传数据", "closeArgument", "GetShowReference", "OnClose".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 窗口与消息框技能（Window & MessageBox）

## 何时使用（When to Use）

- 弹出窗口：内联内容窗口，或加载另一个页面的 iframe 弹窗（常见的编辑弹窗）
- 打开/关闭窗口、iframe 子页关闭后回传数据给父页
- 消息提示：Alert 对话框、Confirm 确认框、Notify 通知框

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages（判定见 `fineui-foundation`）。
2. **窗口内容是内联还是 iframe？**
   - **内联**：内容写在窗口里（F.js `contentEl` / C# `<Content>` 或 `ContentEl`）。
   - **iframe**：加载另一个页面（`EnableIFrame=true` + iframe url），用于独立编辑页。**编辑弹窗基本都用 iframe。**
3. **只是提示消息**（不是弹页面）→ 直接用 MessageBox（[references/messagebox.md](references/messagebox.md)）。

## 各写法速览（一个内联窗口 + 显示按钮）

```javascript
// ① F.js
F.create({
    type: 'Window', id: 'Window1', title: '窗体', width: 650, height: 300, modal: false,
    resizable: true, maximizable: true, bodyPadding: 10, contentEl: '#content1', hidden: true
});
F.create({ type: 'Button', renderTo: '#wrap', text: '显示窗体',
    handler: function () { F.ui.Window1.show(); } });
```
```aspx
<%-- ② Pro（WebForms）--%>
<f:Window ID="Window1" runat="server" Title="窗体" Width="650px" Height="300px" IsModal="false"
    EnableResize="true" EnableMaximize="true" BodyPadding="10px" CloseAction="HidePostBack" OnClose="Window1_Close">
    <Content><p>窗口内联内容</p></Content>
</f:Window>
<f:Button ID="btnShow" runat="server" Text="显示窗体" />
<%-- 后台 Page_Load：btnShow.OnClientClick = Window1.GetShowReference(); --%>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Window().ID("Window1").Title("窗体").Width(650).Height(300).IsModal(false)
    .EnableResize(true).EnableMaximize(true).BodyPadding(10)
    .CloseAction(CloseAction.HidePostBack).OnClose(Url.Action("Window1_Close")).ContentEl("#content1").Hidden(true))
@(F.Button().Text("显示窗体").Listener("click", "F.ui.Window1.show();"))
```
```html
<!-- ④ Core-RazorForms（TagHelper）：OnClose="方法名" -->
<f:Window ID="Window1" Title="窗体" Width="650" Height="300" IsModal="false"
    EnableResize="true" EnableMaximize="true" BodyPadding="10" CloseAction="HidePostBack" OnClose="Window1_Close">
    <Content><p>窗口内联内容</p></Content>
</f:Window>
<f:Button ID="btnShow" Text="显示窗体" OnClientClick="F.ui.Window1.show();"></f:Button>
```
```html
<!-- ⑤ Core-RazorPages（TagHelper）：OnClose="@Url.Handler(...)" -->
<f:Window ID="Window1" Title="窗体" Width="650" Height="300" CloseAction="HidePostBack"
    OnClose="@Url.Handler(&quot;Window1_Close&quot;)"> <Content><p>...</p></Content> </f:Window>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/window.md](references/window.md) | Window 容器（内联/iframe）、打开/关闭、iframe 回传数据（closeArgument） |
| [references/messagebox.md](references/messagebox.md) | Alert 对话框、Confirm 确认框、Notify 通知框（含图标、回调） |

## 相关技能（Related Skills）

- `fineui-foundation`：写法判定、RawHtml（消息内容含 HTML 时）
- `fineui-form`：iframe 编辑窗口里通常放一个表单
- `fineui-grid`：双击行/行操作按钮打开编辑窗口

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js camelCase（`modal`/`maximizable`）；C# PascalCase（`IsModal`/`EnableMaximize`）。
2. **`CloseAction="HidePostBack"` + `OnClose`**：要在窗口关闭时触发服务端事件（如刷新父表格），窗口需设 `CloseAction="HidePostBack"` 并绑定 `OnClose`。
3. **OnClose 事件签名各写法不同**：Pro / RazorForms `方法名(object sender, WindowCloseEventArgs e)`（读 `e.CloseArgument`）；MVC `[HttpPost] 方法名()`；RazorPages `OnPost方法名()`。
4. **iframe 窗口回传数据**：子页用 `ActiveWindow.GetHidePostBackReference(参数)`（Pro/带参关闭）或 `F.doPostBack` 回发；**回发参数名 = 窗口ID + "_closeArgument"**（如 `Window1_closeArgument`）。详见 [references/window.md](references/window.md)。
5. **iframe 内弹消息用 `Alert.ShowInTop` / `target:'_top'`**：iframe 里直接 `Alert.Show` 会显示在小框里，跨到顶层用 `ShowInTop`（C#）或 `target: '_top'`（JS）。
6. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
