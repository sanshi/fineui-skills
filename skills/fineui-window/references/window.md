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
```html
<!-- FineUIJava（Thymeleaf 方言）：内联内容直接写在 content 属性里；客户端事件用 <f:listeners> -->
<f:window id="Window1" width="650" height="300" icon="TagBlue" title="窗体" is-modal="false"
    enable-maximize="true" enable-collapse="true" enable-resize="true" auto-scroll="true" body-padding="10"
    close-action="HidePostBack" on-close="Window1_Close" content="<p>窗口内联内容</p>">
    <f:listeners><f:listener event="resize" handler="onWindowResize" /></f:listeners>
</f:window>
```

## 二、iframe 窗口（编辑弹窗最常用）

窗口内容是另一个页面。`EnableIFrame=true` + iframe url（可在打开时动态指定）。

```javascript
// F.js —— iframe: true 必填，否则 show(url) 不会创建 iframe 元素
F.create({ type: 'Window', id: 'Window1', title: '编辑', width: 850, height: 500, modal: true,
    iframe: true, maximizable: true, resizable: true, hidden: true,
    closeAction: 'hidepostback', listeners: { close: function (event, closeArgument) {
        // closeArgument 为子页回传参数（等同 C# OnClose 的 e.CloseArgument）
        if (closeArgument) { F.ui.Grid1.getStore().reload(); }
    } } });
// 打开：F.ui.Window1.show('edit.html?id=1', '编辑 - 张三');
// show 还可覆盖尺寸：F.ui.Window1.show(url, title, 900, 600);
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
```html
<!-- FineUIJava（Thymeleaf 方言）：enable-iframe + hidden；url 在打开时给（target 常用 Parent/Top） -->
<f:window id="Window1" title="编辑" enable-iframe="true" hidden="true" is-modal="true" target="Parent"
    width="850" height="500" enable-maximize="true" enable-resize="true"
    close-action="HidePostBack" on-close="Window1_Close"></f:window>
```
```java
// FineUIJava 页面类：服务端打开 iframe 窗体（url + 标题）；回写目标控件先登记
Window Window1;
public void Button1_Click(Object sender, EventArgs e) {
    Window1.saveStateControlIds("tbxProvince");          // 登记「回写目标」控件（供子页 writeBackValue）
    Window1.show("/iframe/pass-value/iframe-window?selected=xx", "编辑");
}
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
```java
// FineUIJava —— Bean setter（对齐 Core-RazorForms 的控件属性写法）
Window1.setHidden(false);   // 显示
Window1.setHidden(true);    // 隐藏
// 打开 iframe：Window1.show(url, "标题");
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
```java
// FineUIJava —— ActiveWindow 静态方法直接派发命令（无需 RegisterStartupScript，纯 JSON 命令、无 eval）
ActiveWindow.hide();                                  // 仅关闭
ActiveWindow.hidePostBack("arg");                     // 关闭 + 带参回发父页（触发 on-close，参数 → e.getArgument()）
ActiveWindow.hideRefresh();                           // 关闭 + 刷新父页
ActiveWindow.writeBackValue(ddlSheng.getSelectedValue()); // 把值写回父页登记控件（配 ActiveWindow.hide()）
ActiveWindow.hideCallParentFunction("removeActiveTab");   // 关闭 + 按名调父页全局函数（替代「执行 JS」，无 eval）
```
```javascript
// 子页客户端（F.activeWindow 方法集）
F.activeWindow.hide();                 // 仅关闭
F.activeWindow.hidePostBack('arg');    // 关闭 + 带参回发父页（触发 OnClose/close 事件，e.CloseArgument='arg'）
F.activeWindow.hideRefresh();          // 关闭 + 刷新父页
F.activeWindow.hideExecuteScript('parent.F.ui.Grid1.getStore().reload();'); // 关闭 + 执行父页 JS
F.activeWindow.close();                // 关闭（触发 close 事件，但不传 closeArgument；服务端 e.CloseArgument 为空串）
```

> `closeAction: 'close'` 与 `'hidepostback'` 在客户端行为完全一致（同一代码分支），均触发 `close` 事件。`'close'` 是 F.js 独有别名，Pro/Core 服务端枚举只有 `Hide`/`HideRefresh`/`HidePostBack`。

### 父页接收 closeArgument

```javascript
// F.js —— 父页创建窗口时注册 close 事件（closeAction: 'hidepostback' 或 'close' 时触发）
F.create({ type: 'Window', id: 'Window1', iframe: true, hidden: true, closeAction: 'hidepostback',
    listeners: { close: function (event, closeArgument) {
        if (closeArgument) { F.ui.Grid1.getStore().reload(); }   // 收到回传参数，刷新表格
    } } });
```
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
```java
// FineUIJava —— on-close 处理器读 e.getArgument()（不是 WindowCloseEventArgs.CloseArgument）
public void Window1_Close(Object sender, EventArgs e) {
    String arg = String.valueOf(e.getArgument());
    if (arg != null && arg.startsWith("SelectProvince$")) { /* 用 arg 更新父页 */ }
}
```

> 另一种回传：**直接写回发起控件**——子页 `ActiveWindow.GetWriteBackValueReference(值) + ActiveWindow.GetHideReference()`（C#）/ **`ActiveWindow.writeBackValue(值) + ActiveWindow.hide()`（Java，父页先 `Window1.saveStateControlIds("控件id")` 登记目标）**，把值写回父页打开窗口的那个控件（如 TriggerBox），无需 OnClose。

## 五、窗口内嵌表单

窗口里放一个表单 + 底部工具栏（提交/关闭按钮）：C# 用 `Layout="Fit"` + `<Items><f:Form></Items>` + `<Toolbars>`；提交按钮 `ValidateForms` 校验表单。详见 `fineui-form`。

## See also

- [messagebox.md](messagebox.md)：Alert / Confirm / Notify
- `fineui-form`：iframe 编辑窗口里的表单
