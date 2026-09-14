---
name: fineui-grid
description: >
  帮助开发者使用 FineUI 的 Grid（表格）组件，覆盖 F.js（JavaScript）、Pro（WebForms）、
  FineUICore 的 MVC / RazorForms / RazorPages，
  以及 FineUIJava（Spring Boot + Thymeleaf 方言标签）。
  用于表格配置、列定义与渲染、数据加载与绑定、单元格编辑、行选择（复选框多选/单选）、
  分页、排序、合计行、表头过滤、多表头、行分组、树表格、列锁定、单元格合并、
  行扩展/行命令/行事件、拖拽排序、大数据表格与工具栏。
  Trigger phrases（触发词）: "FineUI 表格", "F.Grid", "Grid 列", "grid columns",
  "FineUIJava", "Spring Boot", "Thymeleaf", "@FineUIPage", "f:grid", "render-field", "data-field",
  "RenderField", "RenderCheckField", "复选框多选", "EnableCheckBoxSelect",
  "选中行", "SelectedRowIndexArray", "GetSelectedDataKeys", "getSelectedDataKeys", "RendererFunction", "RendererArgument",
  "列渲染", "服务端分页", "loadData", "DataKeyNames",
  "表格排序", "AllowSorting", "合计行", "EnableSummary", "SummaryType",
  "表头过滤", "AllowFilters", "EnableFilter", "多表头", "GroupField", "行分组", "EnableRowGroup",
  "树表格", "EnableTree", "TreeColumn", "列锁定", "AllowColumnLocking", "单元格合并", "mergeColumns",
  "行扩展列", "RowExpander", "行命令", "RowCommand", "行单击事件", "OnRowClick",
  "拖拽排序", "EnableColumnMove", "大数据表格", "EnableBigData".
metadata:
  author: FineUI
  version: "16.0"
  compatibility: FineUI v16.0（RenderField.Commands、统一事件与回发协议）
---

# FineUI Grid（表格）技能

## 术语：4 部署栈（Core 含 3 种开发模式）

同一个 Grid，不同部署栈/模式写法不同。**先确定目标，再动手，不要混用。**

| 部署栈 | 开发模式 | 前台 UI 写法 | 后台/数据模型 |
|--------|---------|-------------|--------------|
| **F.js** | —（纯 JavaScript） | JS 配置对象 `F.create({...})` | 纯前端 |
| **Pro** | WebForms | ASPX 服务器控件 `<f:Grid runat="server">` | `Page_Load`/`IsPostBack` + designer |
| **Core** | **MVC**（经典 MVC） | **Fluent API** `Html.F().Grid()...` | Controller + `ViewBag` |
| **Core** | **RazorForms**（Core 推荐） | **TagHelper** `<f:Grid>`（PascalCase） | 三件套 + `Page_Load`/`IsPostBack` |
| **Core** | **RazorPages** | **TagHelper** `<f:Grid>`（PascalCase） | 两件套（无 designer）+ `OnGet`/`OnPost` |
| **Java** | Spring Boot | **Thymeleaf 方言** `<f:grid>`（kebab-case） | 两件套（`.html` + 页面类 `.java`，无 designer）+ `Page_Load`/`isPostBack()` |

> **RazorForms 与 RazorPages 共用 TagHelper 标签语法**（`<f:Grid>`/`<f:RenderField>` 等标签名、列属性相同），但**不是完全相同**——**数据初始化与事件挂接的推荐写法不同**：
> - **RazorForms**：数据在后台 `Page_Load` 里 `Grid1.DataSource=...; Grid1.DataBind();`（`<f:Grid>` 标签上**不写** `DataSource`）；按钮 `OnClick="方法名"`。
> - **RazorPages**：数据在标签上**内联** `DataSource="@Model.GetXxx()"`；按钮 `OnClick="@Url.Handler(\"方法名\")"`。
>
> **FineUIJava 是 Core-RazorForms 的「孪生栈」**：同样「标签式有状态服务端组件 + `Page_Load`/`isPostBack()` + 控件字段 + `xxx_Click`/事件处理器」，差异**只在后台栈**——标签/属性全 **kebab-case**（`<f:grid>`/`<f:render-field data-field=…>`），页面类用 Java Bean setter/getter（`Grid1.setDataSource(...); Grid1.dataBind();`），处理器**返回 `void`**（纯 JSON 增量回发，无需 `UIHelper.Result()`）。**客户端 F.js 运行时四栈完全相同**——`F.ui.Grid1.xxx()`、渲染函数、`notifySelectedRows('Grid1')`、监听器 JS 一字不改。详见 `fineui-foundation`。

## 何时使用（When to Use）

- 展示表格数据，配置列（文本/数字/日期/格式化/自定义渲染）
- 行选择：复选框多选、单选、读取选中行
- 数据加载：本地内联数据 / 服务端数据 / 服务端分页
- 单元格编辑、排序、筛选、分组、合计行、工具栏

## 开始前（Before You Start）—— 先判定"哪个模式"

> 判定线索：`.aspx` → **Pro（WebForms）**；`Controllers/` + `Views/*/Index.cshtml` 且用 `Html.F()`（Fluent API）→ **Core-MVC**；`Pages/*.cshtml` 用 `<f:Grid>` 标签 → **Core-RazorForms**（有 `.designer.cs`）或 **Core-RazorPages**（无 `.designer.cs`）；纯前端 `.html` → **F.js**；`.html` 里 `<html xmlns:f="http://fineui.com/java">` + `<f:grid>` 小写标签、`pom.xml`/`mvnw`、页面类 `.java` 带 `@FineUIPage(...)` + `extends FineUIPageBase` → **Java（Spring Boot）**。

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
<f:Grid ID="Grid1" runat="server" IsFluid="true" ShowBorder="true" ShowHeader="true" Title="表格" DataIDField="Id" DataKeyNames="Id">
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

```html
<!-- ⑥ FineUIJava（Thymeleaf 方言）：标签/属性全 kebab-case；列同④结构，数据在页面类 Page_Load 绑定 -->
<!DOCTYPE html>
<html xmlns:f="http://fineui.com/java" layout:decorate="~{shared/layout}">
<head><title>FineUIJava · 基础表格</title></head>
<body>
  <th:block layout:fragment="body">
    <f:grid id="Grid1" is-fluid="true" show-border="true" show-header="true" title="表格" data-id-field="Id" data-text-field="Name">
      <f:columns>
        <f:row-number-field></f:row-number-field>
        <f:render-field header-text="姓名" data-field="Name"></f:render-field>
        <f:render-field header-text="入学年份" data-field="EntranceYear" field-type="Int"></f:render-field>
        <f:render-field header-text="所学专业" data-field="Major" expand-unused-space="true" min-width="150"></f:render-field>
      </f:columns>
    </f:grid>
  </th:block>
  <th:block layout:fragment="script"><script src="/res/js/grid.js"></script></th:block>
</body>
</html>
```
```java
// ⑥后台 Grid.java（页面类，无 designer；字段名 = 标签 id）
@FineUIPage("grid/grid")
public class Grid extends FineUIPageBase {
    com.fineui.java.core.controls.Grid Grid1;   // 全限定名避免与本页类名 Grid 冲突；或 import 后写 Grid Grid1;
    public void Page_Load(Object sender, EventArgs e) {
        if (!isPostBack()) { Grid1.setDataSource(getDataList()); Grid1.dataBind(); }
    }
}
```

## 参考文档（Documentation Reference Files）

> 表格是最复杂组件，按功能分类拆成 13 个参考文件。**按需求只读相关的那几篇**，不必全读。

**基础（数据 / 列 / 分页 / 选择 / 编辑）**

| 文件 | 何时读 |
|------|--------|
| [references/columns.md](references/columns.md) | 列定义、列类型、格式化（日期/数字）、自定义渲染、固定列 |
| [references/data-loading.md](references/data-loading.md) | 内存分页 / 数据库分页、翻页事件回发、RecordCount |
| [references/editing.md](references/editing.md) | 单元格编辑、列编辑器、读取编辑数据（各栈 API 差异） |
| [references/selection.md](references/selection.md) | 行选择：复选框多选/单选、默认选中、读取与设置选中行 |
| [references/paging-toolbar.md](references/paging-toolbar.md) | 分页工具栏、页大小选择器、窄屏简洁分页、全局配置 |

**数据处理（排序 / 合计 / 过滤 / 分组 / 树）**

| 文件 | 何时读 |
|------|--------|
| [references/sorting.md](references/sorting.md) | 客户端/服务端排序、多列排序、按别的字段排、自定义排序函数 |
| [references/summary.md](references/summary.md) | 合计行：客户端/服务端、当前页/全部、浮动、多行 |
| [references/filter.md](references/filter.md) | 表头过滤：文本/数字/日期/下拉/复选、多条件、行内过滤、服务端过滤处理 |
| [references/row-group.md](references/row-group.md) | 行分组：分组头渲染、展开折叠、组内小计 |
| [references/tree-grid.md](references/tree-grid.md) | 树表格：父子层级行、复选框级联、图标、分页 |

**布局与交互（表头 / 行功能 / 高级）**

| 文件 | 何时读 |
|------|--------|
| [references/header.md](references/header.md) | 表头选项（隐藏/菜单/提示/列自定义属性）、**多表头**、动态创建列 |
| [references/row-features.md](references/row-features.md) | 行扩展列、弹窗列、行命令按钮、行单击/双击/选中事件、行/单元格样式、行高行密度 |
| [references/advanced.md](references/advanced.md) | 列锁定、列/行拖拽排序、单元格合并、大数据表格 |

## 概念 → 各写法属性名对照（Key Options at a Glance）

> Core-RazorForms 与 Core-RazorPages 共用同一套 TagHelper 属性（下表 "Core-TagHelper" 列），差异在数据初始化/事件（见约束 5、6 与 selection.md）。**Java 列单列，属性名 = Core-TagHelper 名转 kebab-case**（属性「值」如 `Int`/`Date` 仍保持 PascalCase）。

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent API) | Core-TagHelper | Java (Thymeleaf 方言) |
|------|------|----------------|------------------------|----------------|------------------------|
| 组件 | `type: 'Grid'` | `<f:Grid runat="server">` | `F.Grid()` | `<f:Grid>` | `<f:grid>` |
| 自适应宽度 | `isFluid: true` | `IsFluid="true"` | `.IsFluid(true)` | `IsFluid="true"` | `is-fluid="true"` |
| 行 ID 字段 | `idField` | `DataIDField` | `.DataIDField()` | `DataIDField` | `data-id-field` |
| 列集合 | `columns: [ ]` | `<Columns>` | `.Columns( )` | `<Columns>` | `<f:columns>` |
| 行号列 | `columnType:'rownumberfield'` | `<f:RowNumberField>` | `F.RowNumberField()` | `<f:RowNumberField>` | `<f:row-number-field>` |
| 数据字段 | `field` | `DataField` | `.DataField()` | `DataField` | `data-field` |
| 列标题 | `text` | `HeaderText` | `.HeaderText()` | `HeaderText` | `header-text` |
| 字段类型 | `fieldType` | `FieldType` | `.FieldType(FieldType.X)` | `FieldType` | `field-type="Int"` |
| 内置渲染器 | （fieldType+fieldFormat） | `Renderer`+`RendererArgument` | `.Renderer(Renderer.X).RendererArgument()` | `Renderer`+`RendererArgument` | `renderer="Date"`+`renderer-argument` |
| 自定义 JS 渲染 | `renderer: fn` | `RendererFunction="fn"` | `.RendererFunction("fn")` | `RendererFunction="fn"` | `renderer-function="fn"` |
| 复选框多选 | `checkboxSelect: true` | `EnableCheckBoxSelect="true"` | `.EnableCheckBoxSelect(true)` | `EnableCheckBoxSelect="true"` | `enable-check-box-select="true"` |

## 相关技能（Related Skills）

- `fineui-foundation`：`F.create` / PageManager / 部署栈与模式总览 / RawHtml 安全模型 / 命名约定
- `fineui-form`：Grid 单元格编辑器、过滤字段用到的表单字段（TextBox / NumberBox / DropDownList 等）
- `fineui-window`：Grid 弹窗列用到的 Window 组件
- `fineui-tree`：独立的 Tree 控件（与 Grid 的"树表格"不同，见 tree-grid.md）

## 约束与规则（Constraints & Rules）

1. **绝不编造 API**：不确定的属性去查官网 API（见下方 Official Resources）或 `F/doc/` 的 JSDoc，别猜。不同模式属性名不同，尤其别把某模式的名字用到另一模式。
2. **先定模式、不混用**：一个页面只用一种模式的写法。判定见「开始前」。F.js 用 `text`/`field`，C# 三模式用 `HeaderText`/`DataField`，**Java 用 `header-text`/`data-field`（全 kebab-case）**——不要串；Fluent API（MVC）与 TagHelper（RazorForms/RazorPages）不要混写。
3. **文本默认 HTML 编码（RawHtml 安全模型）**：Grid 单元格/列头文本默认转义。要输出可信 HTML：
   - F.js：列 `renderer` 返回的字符串会作为 HTML 插入（渲染函数内自行保证可信）；普通文本属性用 `F.rawHtml(...)`。
   - C#：给控件文本赋 HTML 用 `xxx.TextRawHtml = new RawHtml("...")`；消息用 `ShowNotify(new RawHtml("..."))`。**绝不手写 `Text` + `TextRaw` 两个属性**。
   - **Java**：标签使用 `xxx-raw-html`（如 `header-text-raw-html` / `text-raw-html`），服务端使用 `new RawHtml(...)` 与对应 setter；消息用 `showNotifyRaw(new RawHtml(...))`。不要以 `encode-text="false"` 作为新代码的通用逃生口。渲染函数返回值与 F.js 一致（客户端插入）。
4. **Pro 数据绑定固定套路**：`!IsPostBack` 内 `Grid1.DataSource = table; Grid1.DataBind();`。忘了 `!IsPostBack` 会在每次回发重复绑定、丢失状态。
5. **Core 三种模式的数据初始化各不相同**：
   - **MVC（Fluent API）**：Controller 设 `ViewBag.Grid1DataSource`，View 里 `.DataSource(ViewBag.Grid1DataSource)`。
   - **RazorForms**：后台 `Page_Load`（`!IsPostBack`）里 `Grid1.DataSource=...; Grid1.DataBind();`，`<f:Grid>` 标签**不写** `DataSource`。
   - **RazorPages**：标签上内联 `DataSource="@Model.GetXxx()"`。
   - **Java（同 RazorForms 范式）**：页面类 `Page_Load`（`!isPostBack()`）里 `Grid1.setDataSource(list); Grid1.dataBind();`，`<f:grid>` 标签**不写** `data-source`；控件字段**手动在类里声明**（`com.fineui.java.core.controls.Grid Grid1;`，字段名 = 标签 `id`）；事件/命令处理器**返回 `void`**（如 `public void Grid1_PageIndexChanged(Object sender, GridPageEventArgs e)`）。
6. **RazorForms vs RazorPages（同为 TagHelper，但后台模型不同）**：RazorForms 三件套（`.cshtml` + `.cshtml.cs` **partial** + `.designer.cs`），后台继承项目现有的 RazorForms 页面基类（官方示例为 `BaseModel`，它本身可继承 `PageModel`），使用 `Page_Load`/`IsPostBack`、`OnClick="方法名"`；同步控件事件返回 `void`，按钮点击示例为 `protected void 方法名(object sender, EventArgs e)`，分页、排序、行命令等则使用各自的专用 `EventArgs`，**不写 `OnPost`、不返回 `IActionResult`、不调用 `UIHelper.Result()`**。RazorPages 两件套（**非** partial、**无** designer）、`OnGet`/`OnPostXxx`、`OnClick="@Url.Handler(\"方法名\")"`，`OnPostXxx` 返回 `IActionResult` 和 `UIHelper.Result()`；**Java 两件套（`.html` + `.java`，无 designer）、`Page_Load`/`isPostBack()`、`on-click="方法名"`、`@FineUIPage("area/page")` 定路由。**
7. **新代码只使用三栈共有列**：普通列用 `RenderField`，布尔展示列用 `RenderCheckField`，另有 `RowNumberField`、`GroupField` 及 `RenderField.Commands`。不要根据 FineUIPro 的历史示例生成仅 Pro 存在的服务端渲染列。
8. **行选择统一使用稳定行 ID**：初始化、读取与服务端主动选中都先设置 `DataIDField` / `data-id-field`，再使用 `SelectedRowID` / `SelectedRowIDArray` / `getSelectedRowIdArray()`。Pro/Core 的 `SelectedRowIndex` / `SelectedRowIndexArray` 已废弃，只用于识别和迁移旧代码；Java 已直接删除同名 getter/setter 与模板属性。读取选中行数据时，Pro / Core-RazorForms 调 `Grid1.GetSelectedDataKeys()`，Java 调 `Grid1.getSelectedDataKeys()`；必须让 `DataKeyNames` / `_DataKeyNames` / `data-key-names` 包含行 ID 字段。数据库分页跨页选择或敏感业务只取稳定行 ID，再查询数据库。详见 [references/selection.md](references/selection.md)。
9. **三个"分组/层级"概念别混**：**多表头**（列的分组，`GroupField`，[header.md](references/header.md)）≠ **行分组**（数据行按字段分组，`EnableRowGroup`，[row-group.md](references/row-group.md)）≠ **树表格**（行父子层级，`EnableTree`，[tree-grid.md](references/tree-grid.md)）。用户说"分组"时先确认是哪一种。
10. **F.js 嵌套配置 vs .NET 拍平属性**：多个高级功能 F.js 把选项收进一个对象（`tree:{...}`/`rowGroup:{...}`/`rowExpander:{...}`/`filter:{...}`），而 Pro/Core 拍平成一堆独立属性（`EnableTree`/`TreeColumn`/...）。转写时注意这种结构差异。
11. **行事件 EventArgs 类名按栈不同**：同一行事件，Pro 与 RazorForms 的参数类名不一样（如行命令 Pro `GridCommandEventArgs` / RazorForms `GridRowCommandEventArgs`）。**Java 的类名与 Core 又不完全一致**：行命令用 `GridCommandEventArgs`（同 Pro），行单击/双击用 `GridRowEventArgs`，行数据绑定用 `GridRowDataBoundEventArgs`，翻页 `GridPageEventArgs`、排序 `GridSortEventArgs`（getter 取值，如 `e.getRowIndex()`/`e.getCommandName()`）。详见 [row-features.md](references/row-features.md)，别照抄错。
12. **合并/拖拽无服务端属性**：单元格合并（`mergeCells`/`mergeColumns`）、行拖拽（`moveRowUp` 等）都是客户端方法；列/行顺序持久化使用 `F.customEvent` + `Page_CustomEvent`，结构化参数传 JSON 对象/数组。详见 [advanced.md](references/advanced.md)。
13. **行命令统一用 `RenderField.Commands`**：Pro、Core RazorForms 与 Java 都使用 `RenderField.Commands`；`Command` 只属于 `RenderField`。客户端附加逻辑监听 Grid 的 `rowcommand`，服务端声明 `OnRowCommand`。

## 官方资源（Official Resources）

- 在线 API：
  - JS：https://fineui.com/js/api/（`FineUI.Grid` / `FineUI.GridColumn`）
  - Pro：https://fineui.com/pro/api/
  - Core：https://fineui.com/core/api/
- **FineUIJava**：控件属性语义与 Core 一致（属性名 kebab-case、属性值同 Core），客户端 F.js API 与 JS 端完全相同；查属性时先看 Core API 再按命名约定转 kebab-case。
- 官网示例库与文档见 https://www.fineui.com/
