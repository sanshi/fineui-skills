---
name: fineui-grid
description: >
  帮助开发者使用 FineUI 的 Grid（表格）组件，覆盖 F.js（JavaScript）、Pro（WebForms），
  以及 FineUICore 的三种开发模式 MVC（Fluent API）/ RazorForms（TagHelper）/ RazorPages（TagHelper）。
  用于表格配置、列定义与渲染、数据加载与绑定、单元格编辑、行选择（复选框多选/单选）、
  分页、排序、分组、合计行、工具栏等场景。
  Trigger phrases（触发词）: "FineUI 表格", "F.Grid", "Grid 列", "grid columns",
  "RenderField", "BoundField", "复选框多选", "EnableCheckBoxSelect", "checkboxSelect",
  "选中行", "SelectedRowIndexArray", "RendererFunction", "RendererArgument",
  "列渲染", "服务端分页", "loadData", "DataKeyNames", "Fluent API", "TagHelper".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI Grid（表格）技能

## 术语：3 部署栈 + Core 3 种开发模式

同一个 Grid，不同部署栈/模式写法不同。**先确定目标，再动手，不要混用。**

| 部署栈 | 开发模式 | 前台 UI 写法 | 后台/数据模型 |
|--------|---------|-------------|--------------|
| **F.js** | —（纯 JavaScript） | JS 配置对象 `F.create({...})` | 纯前端 |
| **Pro** | WebForms | ASPX 服务器控件 `<f:Grid runat="server">` | `Page_Load`/`IsPostBack` + designer |
| **Core** | **MVC**（经典 MVC） | **Fluent API** `Html.F().Grid()...` | Controller + `ViewBag` |
| **Core** | **RazorForms**（Core 推荐） | **TagHelper** `<f:Grid>` | 三件套 + `Page_Load`/`IsPostBack` |
| **Core** | **RazorPages** | **TagHelper** `<f:Grid>` | 两件套（无 designer）+ `OnGet`/`OnPost` |

> **RazorForms 与 RazorPages 共用 TagHelper 标签语法**（`<f:Grid>`/`<f:RenderField>` 等标签名、列属性相同），但**不是完全相同**——**数据初始化与事件挂接的推荐写法不同**：
> - **RazorForms**：数据在后台 `Page_Load` 里 `Grid1.DataSource=...; Grid1.DataBind();`（`<f:Grid>` 标签上**不写** `DataSource`）；按钮 `OnClick="方法名"`。
> - **RazorPages**：数据在标签上**内联** `DataSource="@Model.GetXxx()"`；按钮 `OnClick="@Url.Handler(\"方法名\")"`。

## 何时使用（When to Use）

- 展示表格数据，配置列（文本/数字/日期/格式化/自定义渲染）
- 行选择：复选框多选、单选、读取选中行
- 数据加载：本地内联数据 / 服务端数据 / 服务端分页
- 单元格编辑、排序、筛选、分组、合计行、工具栏

## 开始前（Before You Start）—— 先判定"哪个模式"

> 判定线索：`.aspx` → **Pro（WebForms）**；`Controllers/` + `Views/*/Index.cshtml` 且用 `Html.F()`（Fluent API）→ **Core-MVC**；`Pages/*.cshtml` 用 `<f:Grid>` 标签 → **Core-RazorForms**（有 `.designer.cs`）或 **Core-RazorPages**（无 `.designer.cs`）；纯前端 `.html` → **F.js**。

再确认：**数据从哪来？**（本地内联 / 服务端）**要不要编辑？要不要选择？**

## 各写法速览（同一个基础表格）

```javascript
// ① F.js
F.create({
    type: 'Grid', isFluid: true, id: 'Grid1', renderTo: '#wrap', title: '表格',
    idField: 'Id', textField: 'Name',
    columns: [
        { columnType: 'rownumberfield' },
        { text: '姓名', field: 'Name' },
        { text: '入学年份', field: 'EntranceYear', fieldType: 'int' },
        { text: '所学专业', field: 'Major', flex: 1, minWidth: 150 }
    ],
    data: [{ Id: 1, Name: '张三', EntranceYear: 2020, Major: '软件工程' }]
});
```

```aspx
<%-- ② Pro（WebForms，.aspx）--%>
<f:Grid ID="Grid1" runat="server" IsFluid="true" ShowBorder="true" ShowHeader="true" Title="表格" DataKeyNames="Id">
    <Columns>
        <f:RowNumberField />
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名" />
        <f:RenderField ColumnID="EntranceYear" DataField="EntranceYear" FieldType="Int" HeaderText="入学年份" />
        <f:RenderField ColumnID="Major" DataField="Major" ExpandUnusedSpace="true" MinWidth="150px" HeaderText="所学专业" />
    </Columns>
</f:Grid>
```
```csharp
// ②后台 Grid.aspx.cs
protected void Page_Load(object sender, EventArgs e) {
    if (!IsPostBack) { Grid1.DataSource = GetDataTable(); Grid1.DataBind(); }
}
```

```csharp
// ③ Core-MVC（Fluent API，Index.cshtml）
@{ var F = Html.F(); }
@(F.Grid().IsFluid(true).Title("表格").ID("Grid1").DataIDField("Id").DataTextField("Name")
    .Columns(
        F.RowNumberField(),
        F.RenderField().HeaderText("姓名").DataField("Name"),
        F.RenderField().HeaderText("入学年份").DataField("EntranceYear").FieldType(FieldType.Int),
        F.RenderField().HeaderText("所学专业").DataField("Major").ExpandUnusedSpace(true).MinWidth(150)
    )
    .DataSource(ViewBag.Grid1DataSource)   // Controller: ViewBag.Grid1DataSource = GetDataTable();
)
```

```html
<!-- ④ Core-RazorForms（TagHelper）：标签不写 DataSource，数据在后台 Page_Load 绑定 -->
<f:Grid ID="Grid1" IsFluid="true" ShowBorder="true" ShowHeader="true" Title="表格" DataIDField="Id" DataTextField="Name">
    <Columns>
        <f:RowNumberField />
        <f:RenderField HeaderText="姓名" DataField="Name" />
        <f:RenderField HeaderText="入学年份" DataField="EntranceYear" FieldType="Int" />
        <f:RenderField HeaderText="所学专业" DataField="Major" ExpandUnusedSpace="true" MinWidth="150" />
    </Columns>
</f:Grid>
```
```csharp
// ④后台 Grid.cshtml.cs（partial，配 designer.cs）
protected void Page_Load(object sender, EventArgs e) {
    if (!IsPostBack) { Grid1.DataSource = GetDataTable(); Grid1.DataBind(); }
}
```

```html
<!-- ⑤ Core-RazorPages（TagHelper）：列同④，但数据在标签上内联绑定 -->
<f:Grid ID="Grid1" IsFluid="true" ShowBorder="true" ShowHeader="true" Title="表格"
        DataIDField="Id" DataTextField="Name" DataSource="@Model.GetDataTable()">
    <Columns>
        <%-- 列标签与④完全一致 --%>
    </Columns>
</f:Grid>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/columns.md](references/columns.md) | 列定义、列类型、格式化（日期/数字）、自定义渲染、固定列 |
| [references/data-loading.md](references/data-loading.md) | 内存分页 / 数据库分页、翻页事件回发、RecordCount |
| [references/editing.md](references/editing.md) | 单元格编辑、列编辑器、读取编辑数据（各栈 API 差异） |
| [references/selection.md](references/selection.md) | 行选择：复选框多选/单选、默认选中、读取与设置选中行 |
| [references/paging-toolbar.md](references/paging-toolbar.md) | 分页工具栏、页大小选择器、窄屏简洁分页、全局配置 |

## 概念 → 各写法属性名对照（Key Options at a Glance）

> Core-RazorForms 与 Core-RazorPages 共用同一套 TagHelper 属性（下表 "Core-TagHelper" 列），差异在数据初始化/事件（见约束 5、6 与 selection.md）。

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent API) | Core-TagHelper |
|------|------|----------------|------------------------|----------------|
| 组件 | `type: 'Grid'` | `<f:Grid runat="server">` | `F.Grid()` | `<f:Grid>` |
| 自适应宽度 | `isFluid: true` | `IsFluid="true"` | `.IsFluid(true)` | `IsFluid="true"` |
| 行 ID 字段 | `idField` | `DataKeyNames` | `.DataIDField()` | `DataIDField` |
| 列集合 | `columns: [ ]` | `<Columns>` | `.Columns( )` | `<Columns>` |
| 行号列 | `columnType:'rownumberfield'` | `<f:RowNumberField>` | `F.RowNumberField()` | `<f:RowNumberField>` |
| 数据字段 | `field` | `DataField` | `.DataField()` | `DataField` |
| 列标题 | `text` | `HeaderText` | `.HeaderText()` | `HeaderText` |
| 字段类型 | `fieldType` | `FieldType` | `.FieldType(FieldType.X)` | `FieldType` |
| 内置渲染器 | （fieldType+fieldFormat） | `Renderer`+`RendererArgument` | `.Renderer(Renderer.X).RendererArgument()` | `Renderer`+`RendererArgument` |
| 自定义 JS 渲染 | `renderer: fn` | `RendererFunction="fn"` | `.RendererFunction("fn")` | `RendererFunction="fn"` |
| 复选框多选 | `checkboxSelect: true` | `EnableCheckBoxSelect="true"` | `.EnableCheckBoxSelect(true)` | `EnableCheckBoxSelect="true"` |

## 相关技能（Related Skills）

- `fineui-foundation`：`F.create` / PageManager / 部署栈与模式总览 / RawHtml 安全模型 / 命名约定（规划中）
- `fineui-form`：Grid 单元格编辑器用到表单字段（规划中）

## 约束与规则（Constraints & Rules）

1. **绝不编造 API**：不确定的属性去查官网 API（见下方 Official Resources）或 `F/doc/` 的 JSDoc，别猜。不同模式属性名不同，尤其别把某模式的名字用到另一模式。
2. **先定模式、不混用**：一个页面只用一种模式的写法。判定见「开始前」。F.js 用 `text`/`field`，C# 三模式用 `HeaderText`/`DataField`——不要串；Fluent API（MVC）与 TagHelper（RazorForms/RazorPages）不要混写。
3. **文本默认 HTML 编码（v15.2 安全模型）**：Grid 单元格/列头文本默认转义。要输出可信 HTML：
   - F.js：列 `renderer` 返回的字符串会作为 HTML 插入（渲染函数内自行保证可信）；普通文本属性用 `F.rawHtml(...)`。
   - C#：给控件文本赋 HTML 用 `xxx.TextRawHtml = new RawHtml("...")`；消息用 `ShowNotify(new RawHtml("..."))`。**绝不手写 `Text` + `TextRaw` 两个属性**。
4. **Pro 数据绑定固定套路**：`!IsPostBack` 内 `Grid1.DataSource = table; Grid1.DataBind();`。忘了 `!IsPostBack` 会在每次回发重复绑定、丢失状态。
5. **Core 三种模式的数据初始化各不相同**：
   - **MVC（Fluent API）**：Controller 设 `ViewBag.Grid1DataSource`，View 里 `.DataSource(ViewBag.Grid1DataSource)`。
   - **RazorForms**：后台 `Page_Load`（`!IsPostBack`）里 `Grid1.DataSource=...; Grid1.DataBind();`，`<f:Grid>` 标签**不写** `DataSource`。
   - **RazorPages**：标签上内联 `DataSource="@Model.GetXxx()"`。
6. **RazorForms vs RazorPages（同为 TagHelper，但后台模型不同）**：RazorForms 三件套（`.cshtml` + `.cshtml.cs` **partial** + `.designer.cs`）、`Page_Load`/`IsPostBack`、`OnClick="方法名"`；RazorPages 两件套（**非** partial、**无** designer）、`OnGet`/`OnPostXxx`、`OnClick="@Url.Handler(\"方法名\")"`。
7. **客户端渲染列优先用 `RenderField`**：Core 三模式与 F.js 一致。Pro 另有声明式 `BoundField`（服务端渲染，日期用 `DataFormatString="{0:yyyy/MM/dd}"`），但为保持一致，除非必要优先 `RenderField`。
8. **行选择读取**：C# 端服务端读选中行用 `Grid1.SelectedRowIndexArray`（0 基索引）+ `Grid1.DataKeys[rowIndex][n]` 取主键（需先声明 `DataKeyNames`）。详见 [references/selection.md](references/selection.md)。

## 官方资源（Official Resources）

- 在线 API：
  - JS：https://fineui.com/js/api/（`FineUI.Grid` / `FineUI.GridColumn`）
  - Pro：https://fineui.com/pro/api/
  - Core：https://fineui.com/core/api/
- 官网示例库与文档见 https://www.fineui.com/
