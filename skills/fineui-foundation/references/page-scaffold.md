# 页面骨架（Page Scaffold）—— 从零搭一个 FineUI 页面

一个最小可运行页面在各写法里的结构。**先看关键差异，再看代码。**

## 关键点（务必先知道）

1. **PageManager 是必备控件，但放置位置不同**：
   - **Pro（WebForms）**：`<f:PageManager>` 写在 **aspx 页面内**（`<form runat="server">` 里）。
   - **Core 三模式**：PageManager **不写在业务页**，而是统一放在**共享布局 `_Layout.cshtml`** 里的 `@F.PageManager`。**Core 没有 `<f:PageManager>` TagHelper 写法**——业务页里不要写它。
   - **Java（Spring Boot）**：不写 PageManager 控件；共享母版 `shared/layout.html` 用 `<f:styles>`/`<f:scripts>` 输出 CSS/JS，页面级/按用户配置由实现 `FineUIPageManagerInitializer` bean 完成（渲染前回调）。
2. **CSS/JS 引入**：Core 在 `_Layout.cshtml` 里用 `@F.RenderCss()` / `@F.RenderScript()`（Pro 由 PageManager 自动注入）；**Java 由母版里的 `<f:styles>`（head）/ `<f:scripts>`（body 末）输出**。
3. **启用标签**：Core 需 `_ViewImports.cshtml` 里 `@addTagHelper *, FineUICore` + `@using FineUICore`；**Java 在页面根标签声明方言命名空间 `<html xmlns:f="http://fineui.com/java">`，并用 `layout:decorate="~{shared/layout}"` 装饰母版**。

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

## 6) FineUIJava —— Spring Boot + Thymeleaf 方言（两件套：`.html` + 页面类 `.java`，无 designer）

业务页 `templates/basic/hello.html`（**无 PageManager**，标签/属性全 kebab-case）：

```html
<!DOCTYPE html>
<html xmlns:f="http://fineui.com/java" layout:decorate="~{shared/layout}">
<head><title>FineUIJava · Hello</title></head>
<body>
    <th:block layout:fragment="body">
        <f:button id="btnHello" text="点击弹出对话框" on-click="btnHello_Click"></f:button>
    </th:block>
    <!-- 页面专属脚本放这里（须在 f:scripts 之后） -->
    <th:block layout:fragment="script"></th:block>
</body>
</html>
```
```java
// HelloPage.java —— @FineUIPage 路由 + extends FineUIPageBase；控件字段手动声明（同名 = 标签 id）
package com.fineui.java.examples.basic;

import com.fineui.java.core.*;
import com.fineui.java.core.controls.Button;

@FineUIPage("basic/hello")
public class HelloPage extends FineUIPageBase {

    Button btnHello;   // 与标签 id="btnHello" 同名，框架自动绑定

    public void Page_Load(Object sender, EventArgs e) { }

    public void btnHello_Click(Object sender, EventArgs e) {   // 返回 void，无需 UIHelper.Result()
        showAlert("你好 FineUIJava！", MessageBoxIcon.Warning);
    }
}
```

**共享母版**（PageManager 相当物在此）`templates/shared/layout.html`：唯一写 `html/head/body` 骨架的地方，`<f:styles>` 在 head 输出 CSS、`<f:scripts>` 在 body 末输出 `FineUI.js` + 语言包 + `F.render`：

```html
<!DOCTYPE html>
<html f:lang="true" xmlns:f="http://fineui.com/java">
<head>
    <meta charset="UTF-8" />
    <title layout:title-pattern="$CONTENT_TITLE">FineUIJava</title>
    <f:styles></f:styles>                          <!-- CSS 输出到 head（先行、无 FOUC） -->
</head>
<body>
    <th:block layout:fragment="body"></th:block>   <!-- 各页 body 片段注入这里 -->
    <f:scripts></f:scripts>                         <!-- JS + 语言包 + F.render 输出到 body 末 -->
    <th:block layout:fragment="script"></th:block> <!-- 页面专属脚本（在 f:scripts 之后） -->
</body>
</html>
```

页面级/按用户配置（主题/语言/显示模式）实现一个 `FineUIPageManagerInitializer` bean，全站默认走 `application.properties` 的 `fineui.*` 键（见 [stacks.md](stacks.md) 命名约定）。

---

## 骨架层面对比

| 维度 | Pro (WebForms) | Core-MVC | Core-RazorForms | Core-RazorPages | Java (Spring Boot) |
|------|----------------|----------|-----------------|-----------------|--------------------|
| 页面语法 | `.aspx` `<f:...>` | `Index.cshtml` Fluent | `.cshtml` TagHelper | `.cshtml` TagHelper | `.html` Thymeleaf `<f:xxx>` |
| 文件件数 | 3（aspx/cs/designer） | 2（cshtml + Controller） | 3（cshtml/cs/designer） | 2（cshtml/cs） | 2（html + 页面类 java） |
| 后置基类·方法 | `PageBase`·`Page_Load` | Controller·`Index()`+`[HttpPost]` | `BaseModel`(partial)·`Page_Load` | `BaseModel`·`OnGet`+`OnPostXxx` | `FineUIPageBase`·`Page_Load` |
| 事件绑定 | `OnClick="方法名"` | `Url.Action("...")` | `OnClick="方法名"` | `@Url.Handler("...")` | `on-click="方法名"` |
| 回发结尾 | 无返回（void） | `return UIHelper.Result()` | `return UIHelper.Result()` | `return UIHelper.Result()` | 无返回（void） |
| **PageManager 位置** | **页面内 `<f:PageManager>`** | 共享 `_Layout` `@F.PageManager` | 共享 `_Layout` `@F.PageManager` | 共享 `_Layout` `@F.PageManager` | 母版 `<f:styles>`/`<f:scripts>` + Initializer bean |

> Region 布局各端一致：顶层 `Layout=Region` 的 Panel（Pro 用 `AutoSizePanelID` 撑满、Core/Java 用 `IsViewPort=true` / `is-view-port="true"`），子 Panel 用 `RegionPosition = Top/Left/Center/Right/Bottom`（Java `region-position="Top"`）定位。

## See also

- [stacks.md](stacks.md)：各写法与命名约定
- [rawhtml.md](rawhtml.md)：可信 HTML
- `fineui-grid` 技能：在骨架里放一个数据表格
