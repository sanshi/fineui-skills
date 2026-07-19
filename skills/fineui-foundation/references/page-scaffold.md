# 页面骨架（Page Scaffold）—— 从零搭一个 FineUI 页面

一个最小可运行页面在五种写法里的结构。**先看关键差异，再看代码。**

## 关键点（务必先知道）

1. **PageManager 是必备控件，但放置位置不同**：
   - **Pro（WebForms）**：`<f:PageManager>` 写在 **aspx 页面内**（`<form runat="server">` 里）。
   - **Core 三模式**：PageManager **不写在业务页**，而是统一放在**共享布局 `_Layout.cshtml`** 里的 `@F.PageManager`。**Core 没有 `<f:PageManager>` TagHelper 写法**——业务页里不要写它。
2. **Core 的 CSS/JS 引入**：在 `_Layout.cshtml` 里用 `@F.RenderCss()` / `@F.RenderScript()`（Pro 由 PageManager 自动注入）。
3. **Core 启用 TagHelper**：`_ViewImports.cshtml` 里 `@addTagHelper *, FineUICore` + `@using FineUICore`。

---

## 1) F.js —— 纯前端

引 `FineUI.css` + `FineUI.js`，占位 `<div>`，`F.ready` 内 `F.create`。

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <link href="FineUI.css" rel="stylesheet" />
</head>
<body>
    <div id="wrap1"></div>

    <script src="FineUI.js"></script>
    <script>
        F.init({ theme: 'pure_black', locale: 'zh_CN' });   // 可选：主题/语言
        F.ready(function () {
            F.create({
                type: 'Button', text: '点击弹出对话框', renderTo: '#wrap1',
                handler: function () { F.alert({ message: '你好 FineUI' }); }
            });
        });
    </script>
</body>
</html>
```

`F.create` 工厂：`type`（组件类型）+ `renderTo`（挂载点）+ `id` + `items`（子组件）。全屏 Region 布局：

```javascript
F.create({
    type: 'Panel', renderTo: document.body, isViewPort: true, layout: 'region',
    items: [
        { type: 'Panel', region: 'top',    title: 'Top',    split: true },
        { type: 'Panel', region: 'left',   title: 'Left',   width: 200, split: true },
        { type: 'Panel', region: 'center', title: 'Center' }
    ]
});
```

---

## 2) Pro —— WebForms（三件套：aspx + aspx.cs + aspx.designer.cs）

```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Hello.aspx.cs" Inherits="Xxx.Hello" %>
<!DOCTYPE html>
<html>
<head runat="server"><title></title></head>
<body>
    <form id="form1" runat="server">
        <f:PageManager ID="PageManager1" runat="server" />   <%-- ← 必备，页面内 --%>
        <f:Button ID="btnHello" runat="server" Text="点击弹出对话框" OnClick="btnHello_Click" />
    </form>
</body>
</html>
```
```csharp
// Hello.aspx.cs
public partial class Hello : PageBase {
    protected void Page_Load(object sender, EventArgs e) { }
    protected void btnHello_Click(object sender, EventArgs e) {
        Alert.Show("你好 FineUIPro！", MessageBoxIcon.Warning);
    }
}
// Hello.aspx.designer.cs（自动生成）：protected global::FineUIPro.PageManager PageManager1; protected global::FineUIPro.Button btnHello;
```
全屏布局：`<f:PageManager AutoSizePanelID="Panel1" />` + `<f:Panel Layout="Region"><Items><f:Panel RegionPosition="Top" .../></Items></f:Panel>`。

---

## 3) Core-MVC —— Fluent API（Index.cshtml + Controller）

业务页（**无 PageManager**）：

```cshtml
@{ ViewBag.Title = "Hello"; var F = Html.F(); }

@section body {
    @(F.Button().ID("btnHello").Text("点击弹出对话框").OnClick(Url.Action("btnHello_Click")))
}
```
```csharp
// HelloController.cs
[Area("Basic")]
public class HelloController : BaseController {
    public IActionResult Index() => View();

    [HttpPost, ValidateAntiForgeryToken]
    public IActionResult btnHello_Click() {
        Alert.Show("你好 FineUI！", MessageBoxIcon.Warning);
        return UIHelper.Result();
    }
}
```

**共享布局**（PageManager 在此，三模式通用结构）`Views/Shared/_Layout.cshtml`：

```cshtml
@{ var F = Html.F(); }
<!DOCTYPE html>
<html>
<head>
    <title>@ViewBag.Title</title>
    @F.RenderCss()              @* 引 FineUI 样式 *@
</head>
<body>
    @Html.AntiForgeryToken()
    @F.PageManager             @* ← 必备控件，统一在布局里 *@
    @RenderSection("body", true)
    @F.RenderScript()          @* 引 FineUI 脚本 *@
</body>
</html>
```

---

## 4) Core-RazorForms —— TagHelper（三件套：cshtml + cshtml.cs(partial) + designer）

```cshtml
@page
@model Xxx.HelloModel
@{ ViewBag.Title = "Hello"; }

@section body {
    <f:Button ID="btnHello" Text="点击弹出对话框" OnClick="btnHello_Click"></f:Button>
}
```
```csharp
// Hello.cshtml.cs（partial，WebForms 风格）
public partial class HelloModel : BaseModel {
    protected void Page_Load(object sender, EventArgs e) { }
    protected void btnHello_Click(object sender, EventArgs e) {
        Alert.Show("你好 FineUI！", MessageBoxIcon.Warning);
    }
}
// Hello.cshtml.designer.cs（自动生成）：protected FineUICore.Button btnHello;
```
PageManager 同样在 `Pages/Shared/_Layout.cshtml`（`@F.PageManager` + `@F.RenderCss()`/`@F.RenderScript()`，结构与 MVC 版一致）。

---

## 5) Core-RazorPages —— TagHelper（两件套：cshtml + cshtml.cs，无 designer）

```cshtml
@page
@model Xxx.HelloModel
@{ ViewBag.Title = "Hello"; }

@section body {
    <f:Button ID="btnHello" Text="点击弹出对话框" OnClick="@Url.Handler(&quot;btnHello_Click&quot;)"></f:Button>
}
```
```csharp
// Hello.cshtml.cs（普通 PageModel，无 designer）
public class HelloModel : BaseModel {
    public void OnGet() { }
    public IActionResult OnPostBtnHello_Click() {
        Alert.Show("你好 FineUI！", MessageBoxIcon.Warning);
        return UIHelper.Result();
    }
}
```
PageManager 同样在 `Pages/Shared/_Layout.cshtml`。

---

## 骨架层面对比

| 维度 | Pro (WebForms) | Core-MVC | Core-RazorForms | Core-RazorPages |
|------|----------------|----------|-----------------|-----------------|
| 页面语法 | `.aspx` `<f:...>` | `Index.cshtml` Fluent | `.cshtml` TagHelper | `.cshtml` TagHelper |
| 文件件数 | 3（aspx/cs/designer） | 2（cshtml + Controller） | 3（cshtml/cs/designer） | 2（cshtml/cs） |
| 后置基类·方法 | `PageBase`·`Page_Load` | Controller·`Index()`+`[HttpPost]` | `BaseModel`(partial)·`Page_Load` | `BaseModel`·`OnGet`+`OnPostXxx` |
| 事件绑定 | `OnClick="方法名"` | `Url.Action("...")` | `OnClick="方法名"` | `@Url.Handler("...")` |
| **PageManager 位置** | **页面内 `<f:PageManager>`** | 共享 `_Layout` `@F.PageManager` | 共享 `_Layout` `@F.PageManager` | 共享 `_Layout` `@F.PageManager` |

> Region 布局各端一致：顶层 `Layout=Region` 的 Panel（Pro 用 `AutoSizePanelID` 撑满、Core 用 `IsViewPort=true`），子 Panel 用 `RegionPosition = Top/Left/Center/Right/Bottom` 定位。

## See also

- [stacks.md](stacks.md)：五写法与命名约定
- [rawhtml.md](rawhtml.md)：可信 HTML
- `fineui-grid` 技能：在骨架里放一个数据表格
