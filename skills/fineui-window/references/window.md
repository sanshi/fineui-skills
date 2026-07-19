# Window 窗口（容器 / 打开关闭 / 回传数据）

## 一、内联内容窗口

内容写在窗口内部。

```javascript
// F.js —— contentEl 指向页面内隐藏的 div
F.create({ type: 'Window', id: 'Window1', title: '窗体', width: 650, height: 300,
    modal: false, resizable: true, maximizable: true, collapsible: true, bodyPadding: 10,
    autoScroll: true, icon: '../res/icon/tag_blue.png', contentEl: '#content1', hidden: true });
```
```aspx
<%-- Pro / Core-TagHelper —— <Content> 内联 --%>
<f:Window ID="Window1" runat="server" Title="窗体" Width="650px" Height="300px" Icon="TagBlue"
    IsModal="false" EnableResize="true" EnableMaximize="true" EnableCollapse="true" AutoScroll="true"
    BodyPadding="10px" CloseAction="HidePostBack" OnClose="Window1_Close">
    <Content><p>窗口内联内容</p></Content>
</f:Window>
```
```csharp
// Core-MVC（Fluent）
@(F.Window().ID("Window1").Title("窗体").Width(650).Height(300).Icon(Icon.TagBlue).IsModal(false)
    .EnableResize(true).EnableMaximize(true).EnableCollapse(true).AutoScroll(true).BodyPadding(10)
    .CloseAction(CloseAction.HidePostBack).OnClose(Url.Action("Window1_Close")).ContentEl("#content1"))
```

## 二、iframe 窗口（编辑弹窗最常用）

窗口内容是另一个页面。`EnableIFrame=true` + iframe url（可在打开时动态指定）。

```javascript
// F.js —— show 时传 url
F.create({ type: 'Window', id: 'Window1', title: '编辑', width: 850, height: 500, modal: true,
    maximizable: true, resizable: true, hidden: true });
// 打开：F.ui.Window1.show('edit.html?id=1', '编辑 - 张三');
```
```aspx
<%-- Pro —— EnableIFrame + Hidden；url 在打开时给 --%>
<f:Window ID="Window1" runat="server" Title="编辑" EnableIFrame="true" Hidden="true" IsModal="true"
    Target="Top" Width="850px" Height="500px" EnableMaximize="true" EnableResize="true"
    CloseAction="HidePostBack" OnClose="Window1_Close"></f:Window>
```
```csharp
// Core-MVC（Fluent）—— 固定 url 用 .IFrameUrl；动态 url 在打开时给
@(F.Window().ID("Window1").Title("编辑").EnableIFrame(true).Target(Target.Top)
    .IFrameUrl(Url.Content("~/IFrame/Window/IFrameWindow")).Width(850).Height(500)
    .EnableMaximize(true).EnableResize(true).CloseAction(CloseAction.HidePostBack)
    .OnClose(Url.Action("Window1_Close")).Hidden(true))
```
```html
<!-- Core-TagHelper（RazorForms/RazorPages）：RazorForms OnClose="方法名"，RazorPages OnClose="@Url.Handler(...)" -->
<f:Window ID="Window1" Title="编辑" EnableIFrame="true" Hidden="true" IsModal="true" Target="Top"
    Width="850" Height="500" CloseAction="HidePostBack" OnClose="Window1_Close"></f:Window>
```

## 三、打开 / 关闭窗口

### 客户端（各写法一致）

```javascript
F.ui.Window1.show();                       // 显示
F.ui.Window1.show('edit.html?id=1', '编辑'); // 显示并加载 iframe url + 标题
F.ui.Window1.hide();                       // 隐藏（纯客户端）
F.ui.Window1.hidePostBack();               // 隐藏并回发（触发 OnClose）
```

### 服务端触发

```csharp
// Pro —— 注册脚本（客户端绑定：btn.OnClientClick = Window1.GetShowReference(url, title);）
PageContext.RegisterStartupScript(Window1.GetShowReference("edit.aspx?id=1", "编辑 - 张三"));
```
```csharp
// Core-MVC / RazorPages —— UIHelper
UIHelper.Window("Window1").Show();
```
```csharp
// Core-RazorForms —— 控件属性
Window1.Hidden = false;
```

## 四、iframe 子页关闭并回传数据给父页（closeArgument）

**子页（iframe 内的编辑页）** 关闭自身并把数据回传给父页；**父页** 在 `OnClose` 里接收。

### 子页关闭自身

```csharp
// Pro / Core —— 服务端注册脚本，ActiveWindow = 承载本 iframe 的父页窗口
PageContext.RegisterStartupScript(ActiveWindow.GetHideReference());            // 仅关闭
PageContext.RegisterStartupScript(ActiveWindow.GetHidePostBackReference("arg"));// 关闭 + 带参回发父页(触发 OnClose)
PageContext.RegisterStartupScript(ActiveWindow.GetHideRefreshReference());     // 关闭 + 刷新父页
PageContext.RegisterStartupScript(ActiveWindow.GetHideExecuteScriptReference("parent.removeActiveTab();")); // 关闭 + 执行 JS
```
```javascript
// 子页客户端
F.activeWindow.hide();                 // 仅关闭
F.activeWindow.hidePostBack('arg');    // 关闭 + 带参回发
```

### 父页接收 closeArgument

```csharp
// Pro / Core-RazorForms —— WindowCloseEventArgs.CloseArgument
protected void Window1_Close(object sender, WindowCloseEventArgs e) {
    if (e.CloseArgument.StartsWith("SelectProvince$"))
        ddlSheng.SelectedValue = e.CloseArgument.Substring("SelectProvince$".Length);
}
```
```csharp
// Core-MVC —— 回发参数名 = 窗口ID + "_closeArgument"
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Window1_Close(string Window1_closeArgument) {
    // 用 Window1_closeArgument 更新父页
    return UIHelper.Result();
}
// Core-RazorPages —— OnPostWindow1_Close(string Window1_closeArgument)
```

> 另一种回传：**直接写回发起控件**——子页 `ActiveWindow.GetWriteBackValueReference(值) + ActiveWindow.GetHideReference()`，把值写回父页打开窗口的那个控件（如 TriggerBox），无需 OnClose。

## 五、窗口内嵌表单

窗口里放一个表单 + 底部工具栏（提交/关闭按钮）：C# 用 `Layout="Fit"` + `<Items><f:Form></Items>` + `<Toolbars>`；提交按钮 `ValidateForms` 校验表单。详见 `fineui-form`。

## See also

- [messagebox.md](messagebox.md)：Alert / Confirm / Notify
- `fineui-form`：iframe 编辑窗口里的表单
