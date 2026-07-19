# 部署栈与开发模式总览（Stacks & Modes）

FineUI 有 **3 部署栈**，其中 **Core 含 3 种开发模式**。同一个组件，写法各不相同——**先确定目标，不要混用**。

## 全景表

| 部署栈 | 开发模式 | 技术栈 | 前台写法 | 后台/数据模型 |
|--------|---------|--------|---------|--------------|
| **F.js** | —（纯 JS） | JavaScript + jQuery | `F.create({ type:'Xxx', ... })` | 纯前端 |
| **Pro** | WebForms | .NET Framework | `<f:Xxx runat="server">` | `Page_Load`/`IsPostBack` + designer |
| **Core** | **MVC**（经典 MVC） | ASP.NET Core | **Fluent API** `Html.F().Xxx()...` | Controller + `ViewBag` |
| **Core** | **RazorForms**（Core 推荐） | ASP.NET Core | **TagHelper** `<f:Xxx>` | 三件套 + `Page_Load`/`IsPostBack` |
| **Core** | **RazorPages** | ASP.NET Core | **TagHelper** `<f:Xxx>` | 两件套（无 designer）+ `OnGet`/`OnPost` |

> **术语纪律**：`Fluent API`、`TagHelper` 是**前台写法**，不是模式名；模式永远说 MVC / RazorForms / RazorPages。**RazorForms 与 RazorPages 共用 TagHelper 标签**，但数据初始化与事件不同（见下）。

## 判定线索（拿到项目怎么认）

- 纯前端 `.html` + `F.create` → **F.js**
- `.aspx` + `<f:Xxx runat="server">` → **Pro（WebForms）**
- `Controllers/` + `Views/*/Index.cshtml` 且用 `Html.F()` → **Core-MVC**
- `Pages/*.cshtml` + `<f:Xxx>` 标签：有 `.cshtml.designer.cs` → **Core-RazorForms**；无 designer → **Core-RazorPages**

## 前台写法差异（同一个组件，五写法）

以创建一个 Panel 为例：

```javascript
// F.js
F.create({ type: 'Panel', renderTo: '#wrap', id: 'Panel1', title: '面板', bodyPadding: 10 });
```
```aspx
<%-- Pro（WebForms）--%>
<f:Panel ID="Panel1" runat="server" Title="面板" BodyPadding="10px"> ... </f:Panel>
```
```csharp
// Core-MVC（Fluent API）
@(Html.F().Panel().ID("Panel1").Title("面板").BodyPadding(10))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages 相同）-->
<f:Panel ID="Panel1" Title="面板" BodyPadding="10"> ... </f:Panel>
```

## 后台/数据与事件模型差异（关键分歧点）

| 维度 | Pro (WebForms) | Core-MVC | Core-RazorForms | Core-RazorPages |
|------|----------------|----------|-----------------|-----------------|
| 数据绑定 | `x.DataSource=...; x.DataBind();`（`!IsPostBack`） | Controller 设 `ViewBag`，View `.DataSource(ViewBag.x)` | 后台 `Page_Load` 里 `DataBind()` | 标签内联 `DataSource="@Model.GetX()"` |
| 生命周期 | `Page_Load` + `IsPostBack` | Controller action | `Page_Load` + `IsPostBack` | `OnGet` / `OnPost` |
| 控件字段 | designer 声明 | 无（View 内局部） | designer 声明 `protected FineUICore.Xxx x;` | 无 |
| 按钮事件 | `OnClick="方法名"` | `.OnClick(Url.Action("方法名"), "参数控件")` | `OnClick="方法名"` | `OnClick="@Url.Handler(\"方法名\")"` |
| 服务端回发 | `方法名(object s, EventArgs e)` | `[HttpPost] IActionResult 方法名(...)` | `方法名(object s, EventArgs e)` | `IActionResult OnPost方法名(...)` |
| 服务端操作控件 | 直接用控件字段 `x.Xxx=...` | `UIHelper.Xxx("id").Yyy(...)` | 直接用控件字段 | `UIHelper.Xxx("id").Yyy(...)` |

## 命名约定

- **属性大小写**：JS 用 camelCase（`bodyPadding`、`showHeader`）；C# 三模式用 PascalCase（`BodyPadding`、`ShowHeader`）。
- **少数名称不是简单大小写映射**（按组件而定），如 Grid 列：F.js `text`/`field` ↔ C# `HeaderText`/`DataField`。不确定就查该组件的 API/JSDoc，别硬套。
- **可信 HTML**：`Xxx` → `XxxRawHtml`（见 [rawhtml.md](rawhtml.md)）。
- **“全局可设、单控件可覆盖”的配置项（如 Grid 级）四段命名**：
  - JS 全局：`grid` + 帕斯卡名（`gridPagerAutoSimpleMode`，`F.init({...})` 设）
  - JS/C# 实例属性：`pagerAutoSimpleMode` / `PagerAutoSimpleMode`
  - C# 全局：`Grid` 前缀（`GridPagerAutoSimpleMode`，PageManager 或 `Web.config`/`appsettings.json` 设）
- **全局配置入口**：Pro = `Web.config` 的 `<FineUIPro>` 段或页面 `<f:PageManager>`；Core = `appsettings.json` 的 `FineUI` 段或页面 `F.PageManager.GridXxx(...)`。

## See also

- [page-scaffold.md](page-scaffold.md)：一个最小页面的完整骨架（含 PageManager）
- [rawhtml.md](rawhtml.md)：可信 HTML 声明式写法
- `fineui-grid` 技能：以 Grid 为例的五写法完整对照
