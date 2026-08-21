# 部署栈与开发模式总览（Stacks & Modes）

FineUI 有 **4 部署栈**（F.js / Pro / Core / Java），其中 **Core 含 3 种开发模式**。同一个组件，写法各不相同——**先确定目标，不要混用**。

## 全景表

| 部署栈 | 开发模式 | 技术栈 | 前台写法 | 后台/数据模型 |
|--------|---------|--------|---------|--------------|
| **F.js** | —（纯 JS） | JavaScript + jQuery | `F.create({ type:'Xxx', ... })` | 纯前端 |
| **Pro** | WebForms | .NET Framework | `<f:Xxx runat="server">` | `Page_Load`/`IsPostBack` + designer |
| **Core** | **MVC**（经典 MVC） | ASP.NET Core | **Fluent API** `Html.F().Xxx()...` | Controller + `ViewBag` |
| **Core** | **RazorForms**（Core 推荐） | ASP.NET Core | **TagHelper** `<f:Xxx>`（PascalCase） | 三件套 + `Page_Load`/`IsPostBack` |
| **Core** | **RazorPages** | ASP.NET Core | **TagHelper** `<f:Xxx>`（PascalCase） | 两件套（无 designer）+ `OnGet`/`OnPost` |
| **Java** | Spring Boot | Java + Spring Boot + Thymeleaf | **Thymeleaf 方言** `<f:xxx>`（kebab-case） | 两件套（`.html` + 页面类 `.java`，无 designer）+ `Page_Load`/`isPostBack()` |

> **术语纪律**：`Fluent API`、`TagHelper`、`Thymeleaf 方言` 是**前台写法**，不是模式名；Core 模式永远说 MVC / RazorForms / RazorPages。**RazorForms 与 RazorPages 共用 TagHelper 标签**，但数据初始化与事件不同（见下）。
>
> **Java 与 Core-RazorForms 是「孪生栈」**：两者都是「标签式有状态服务端组件 + `Page_Load`/`IsPostBack` + 控件字段 + `xxx_Click` 事件」。差异**只在后台栈**：Thymeleaf vs Razor 模板、kebab-case vs PascalCase、Java Bean setter/getter vs C# 属性。**客户端 F.js 运行时四栈完全相同**——`F.ui.Grid1.xxx()`、渲染函数、监听器 JS 一字不差，Java 侧一行不改地复用。

## 判定线索（拿到项目怎么认）

- 纯前端 `.html` + `F.create` → **F.js**
- `.aspx` + `<f:Xxx runat="server">` → **Pro（WebForms）**
- `Controllers/` + `Views/*/Index.cshtml` 且用 `Html.F()` → **Core-MVC**
- `Pages/*.cshtml` + `<f:Xxx>` 标签：有 `.cshtml.designer.cs` → **Core-RazorForms**；无 designer → **Core-RazorPages**
- `.html` 里 `<html xmlns:f="http://fineui.com/java">` + `<f:xxx>` 小写标签、`layout:decorate`；`pom.xml`/`mvnw`；页面类 `.java` 带 `@FineUIPage(...)` + `extends FineUIPageBase` → **Java（Spring Boot）**

## 前台写法差异（同一个组件，各写法）

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
```html
<!-- FineUIJava（Thymeleaf 方言：标签名 + 属性名全 kebab-case）-->
<f:panel id="Panel1" title="面板" body-padding="10"> ... </f:panel>
```

## 后台/数据与事件模型差异（关键分歧点）

| 维度 | Pro (WebForms) | Core-MVC | Core-RazorForms | Core-RazorPages | Java (Spring Boot) |
|------|----------------|----------|-----------------|-----------------|--------------------|
| 数据绑定 | `x.DataSource=...; x.DataBind();`（`!IsPostBack`） | Controller 设 `ViewBag`，View `.DataSource(ViewBag.x)` | 后台 `Page_Load` 里 `DataBind()` | 标签内联 `DataSource="@Model.GetX()"` | `x.setDataSource(...); x.dataBind();`（`!isPostBack()`） |
| 生命周期 | `Page_Load` + `IsPostBack` | Controller action | `Page_Load` + `IsPostBack` | `OnGet` / `OnPost` | `Page_Load(Object,EventArgs)` + `isPostBack()` |
| 控件字段 | designer 声明 | 无（View 内局部） | designer 声明 `protected FineUICore.Xxx x;` | 无 | **手动在页面类声明** `Xxx x;`（同名 = 标签 id） |
| 按钮事件 | `OnClick="方法名"` | `.OnClick(Url.Action("方法名"), "参数控件")` | `OnClick="方法名"` | `OnClick="@Url.Handler(\"方法名\")"` | `on-click="方法名"` |
| 服务端回发 | `方法名(object s, EventArgs e)` | `[HttpPost] IActionResult 方法名(...)` | `方法名(object s, EventArgs e)` | `IActionResult OnPost方法名(...)` | `void 方法名(Object s, EventArgs e)`（**返回 void**） |
| 服务端操作控件 | 直接用控件字段 `x.Xxx=...` | `UIHelper.Xxx("id").Yyy(...)` | 直接用控件字段 | `UIHelper.Xxx("id").Yyy(...)` | 直接用控件字段 `x.setYyy(...)`（Bean setter） |
| 页面路由 | 物理 `.aspx` 路径 | Controller 路由 | Pages 目录结构 | Pages 目录结构 | `@FineUIPage("area/page")` 注解 |

> Java 回发处理器**返回 `void`**：框架自动 diff 脏属性并回传 JSON 增量，不像 Core 需 `return UIHelper.Result();`。

## 命名约定

- **属性大小写**：JS 用 camelCase（`bodyPadding`、`showHeader`）；C# 三模式用 PascalCase（`BodyPadding`、`ShowHeader`）；**Java 标签属性用 kebab-case（`body-padding`、`show-header`）**。
- **属性「值」在 Java 里保持原样**：枚举/图标/类型等值仍是 PascalCase，如 `text-mode="Password"`、`icon="TagBlue"`、`field-type="Int"`、`renderer="Date"`——只有属性「名」转 kebab-case。
- **少数名称不是简单大小写映射**（按组件而定），如 Grid 列：F.js `text`/`field` ↔ C# `HeaderText`/`DataField` ↔ Java `header-text`/`data-field`。不确定就查该组件的 API/JSDoc，别硬套。
- **可信 HTML**：`Xxx` → `XxxRawHtml`（C#）/ `xxx-raw-html`（Java）（见 [rawhtml.md](rawhtml.md)）。
- **“全局可设、单控件可覆盖”的配置项（如 Grid 级）多段命名**：
  - JS 全局：`grid` + 帕斯卡名（`gridPagerAutoSimpleMode`，`F.init({...})` 设）
  - JS/C# 实例属性：`pagerAutoSimpleMode` / `PagerAutoSimpleMode`；Java 标签属性 `pager-auto-simple-mode`
  - C# 全局：`Grid` 前缀（`GridPagerAutoSimpleMode`，PageManager 或 `Web.config`/`appsettings.json` 设）
  - Java 全局：`application.properties` 的 `fineui.grid-pager-auto-simple-mode`（`fineui.` + kebab-case）
- **全局配置入口**：Pro = `Web.config` 的 `<FineUIPro>` 段或页面 `<f:PageManager>`；Core = `appsettings.json` 的 `FineUI` 段或页面 `F.PageManager.GridXxx(...)`；**Java = `application.properties` 的 `fineui.*` 键（全站默认）+ `FineUIPageManagerInitializer` bean（页面级/按用户，`pm.theme(...)`/`pm.language(...)`/`pm.displayMode(...)`）**。

## See also

- [page-scaffold.md](page-scaffold.md)：一个最小页面的完整骨架（含 PageManager）
- [rawhtml.md](rawhtml.md)：可信 HTML 声明式写法
- `fineui-grid` 技能：以 Grid 为例的各写法完整对照（含 Java）
