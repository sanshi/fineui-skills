---
name: fineui-grid
description: >
  帮助开发者使用 FineUI 的 Grid（表格）组件，覆盖 F.js 前端、FineUIPro(WebForms/aspx)、
  FineUICore(MVC 流式 API) 与 FineUICore(RazorForms/RazorPages TagHelper) 四端写法。
  用于表格配置、列定义与渲染、数据加载与绑定、单元格编辑、行选择（复选框多选/单选）、
  分页、排序、分组、合计行、工具栏等场景。
  Trigger phrases（触发词）: "FineUI 表格", "F.Grid", "Grid 列", "grid columns",
  "RenderField", "BoundField", "复选框多选", "EnableCheckBoxSelect", "checkboxSelect",
  "选中行", "SelectedRowIndexArray", "RendererFunction", "RendererArgument",
  "列渲染", "服务端分页", "loadData", "DataKeyNames".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI Grid（表格）技能

用于在 **FineUI 四端**中构建和配置 Grid（表格）组件。四端 API 名称不同，但概念一致。

## 何时使用（When to Use）

- 展示表格数据，配置列（文本/数字/日期/格式化/自定义渲染）
- 行选择：复选框多选、单选、读取选中行
- 数据加载：本地内联数据 / 服务端数据 / 服务端分页
- 单元格编辑、排序、筛选、分组、合计行、工具栏

## 开始前（Before You Start）—— 先确定"哪一端"

FineUI 有四种写法，**API 不同，先确认再动手**，不要混用：

1. **F.js（纯 JavaScript）** → `F.create({ type: 'Grid', ... })`，列是 JSON 对象
2. **FineUIPro（.NET Framework / WebForms）** → `.aspx` 里 `<f:Grid runat="server">` + `.aspx.cs` 后置代码，`!IsPostBack` 内 `DataBind()`
3. **FineUICore（MVC，流式 API）** → `.cshtml` 里 `Html.F().Grid()...`，数据经 `ViewBag` 传入 `.DataSource(...)`
4. **FineUICore（RazorForms / RazorPages，TagHelper）** → `.cshtml` 里 `<f:Grid>` 标签，`DataSource="@Model.GetXxx()"`

> **判定线索**：项目里是 `.aspx` → Pro；是 `Controllers/` + `Views/*/Index.cshtml` 用 `Html.F()` → Core MVC 流式；是 `Pages/*.cshtml` 用 `<f:Grid>` 标签 → Core TagHelper（有 `.designer.cs` 是 RazorForms，没有是 RazorPages）；纯前端 `.html` → F.js。

再确认：**数据从哪来？**（本地内联 / 服务端）**要不要编辑？要不要选择？**

## 四端速览（同一个基础表格）

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
<%-- ② FineUIPro（.aspx）--%>
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
// ②后置代码 Grid.aspx.cs
protected void Page_Load(object sender, EventArgs e) {
    if (!IsPostBack) { Grid1.DataSource = GetDataTable(); Grid1.DataBind(); }
}
```

```csharp
// ③ FineUICore MVC（流式 API，Index.cshtml）
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
<!-- ④ FineUICore TagHelper（RazorForms/RazorPages，.cshtml）-->
<f:Grid ID="Grid1" IsFluid="true" ShowBorder="true" ShowHeader="true" Title="表格"
        DataIDField="Id" DataTextField="Name" DataSource="@Model.GetDataTable()">
    <Columns>
        <f:RowNumberField />
        <f:RenderField HeaderText="姓名" DataField="Name" />
        <f:RenderField HeaderText="入学年份" DataField="EntranceYear" FieldType="Int" />
        <f:RenderField HeaderText="所学专业" DataField="Major" ExpandUnusedSpace="true" MinWidth="150" />
    </Columns>
</f:Grid>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/columns.md](references/columns.md) | 列定义、列类型、格式化（日期/数字）、自定义渲染、固定列 |
| [references/selection.md](references/selection.md) | 行选择：复选框多选/单选、默认选中、读取与设置选中行 |
| _（规划中）_ data-loading.md | 服务端数据、服务端分页、loadData |
| _（规划中）_ editing.md | 单元格/行编辑、读取编辑后数据 |
| _（规划中）_ paging-toolbar.md | 分页工具栏、PageManager 全局配置、窄屏简洁分页 |

## 概念 → 四端属性名对照（Key Options at a Glance）

| 概念 | F.js | Pro (aspx) | Core 流式 | Core TagHelper |
|------|------|-----------|-----------|----------------|
| 组件 | `type: 'Grid'` | `<f:Grid>` | `F.Grid()` | `<f:Grid>` |
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

- `fineui-foundation`：`F.create` / PageManager / 四端总览 / RawHtml 安全模型 / 命名约定（规划中）
- `fineui-form`：Grid 单元格编辑器用到表单字段（规划中）

## 约束与规则（Constraints & Rules）

1. **绝不编造 API**：不确定的属性去查官网 API（见下方 Official Resources）或 `F/doc/` 的 JSDoc，别猜。四端属性名不同，尤其别把某端的名字用到另一端。
2. **先定端、不混用**：一个页面只用一种端的写法。判定见「开始前」。F.js 用 `text`，C# 三端用 `HeaderText`；F.js 用 `field`，C# 三端用 `DataField`——不要串。
3. **文本默认 HTML 编码（v15.2 安全模型）**：Grid 单元格/列头文本默认转义。要输出可信 HTML：
   - F.js：列 `renderer` 返回的字符串会作为 HTML 插入（渲染函数内自行保证可信）；普通文本属性用 `F.rawHtml(...)`。
   - C#：给控件文本赋 HTML 用 `xxx.TextRawHtml = new RawHtml("...")`；消息用 `ShowNotify(new RawHtml("..."))`。**绝不手写 `Text` + `TextRaw` 两个属性**。
4. **Pro 数据绑定固定套路**：`!IsPostBack` 内 `Grid1.DataSource = table; Grid1.DataBind();`。忘了 `!IsPostBack` 会在每次回发重复绑定、丢失状态。
5. **Core 两种数据绑定**：MVC 流式经 `ViewBag`（Controller 设 `ViewBag.Grid1DataSource`，View `.DataSource(ViewBag.Grid1DataSource)`）；TagHelper 多为标签内 `DataSource="@Model.GetXxx()"`（RazorPages/RazorForms）或后置 `Grid1.DataSource=...; DataBind()`（RazorForms）。
6. **RazorForms vs RazorPages**：RazorForms 是三件套（`.cshtml` + `.cshtml.cs` **partial** + `.designer.cs`），用 `Page_Load`/`IsPostBack`、`OnClick="方法名"`；RazorPages 是两件套（**非** partial、**无** designer），用 `OnGet`/`OnPostXxx`、`OnClick="@Url.Handler(\"方法名\")"`。
7. **客户端渲染列优先用 `RenderField`**：四端一致、与 F.js 对齐。Pro 另有声明式 `BoundField`（服务端渲染，日期用 `DataFormatString="{0:yyyy/MM/dd}"`），但为保持四端一致，除非必要优先 `RenderField`。
8. **行选择读取**：C# 端服务端读选中行用 `Grid1.SelectedRowIndexArray`（0 基索引）+ `Grid1.DataKeys[rowIndex][n]` 取主键（需先声明 `DataKeyNames`）。详见 [references/selection.md](references/selection.md)。

## 官方资源（Official Resources）

- 在线 API：
  - JS：https://fineui.com/js/api/（`FineUI.Grid` / `FineUI.GridColumn`）
  - Pro：https://fineui.com/pro/api/
  - Core：https://fineui.com/core/api/
- 官网示例库（按端）与文档见 https://www.fineui.com/
