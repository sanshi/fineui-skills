# Grid 列配置（Columns）

四端并列。概念相同、属性名不同；每段为最小代码。

## 列类型一览

| 用途 | F.js `columnType` | Pro / Core 标签 | Core 流式 |
|------|-------------------|-----------------|-----------|
| 普通渲染列（默认） | `'renderfield'`（默认，可省） | `<f:RenderField>` | `F.RenderField()` |
| 行号列 | `'rownumberfield'` | `<f:RowNumberField>` | `F.RowNumberField()` |
| 布尔展示列 | `'checkboxfield'` | `<f:RenderCheckField>` | `F.RenderCheckField()` |
| 行展开列 | `'rowexpanderfield'` | `<f:RowExpanderField>` | `F.RowExpanderField()` |

> Pro 另有服务端渲染的声明式列：`<f:BoundField>`、`<f:TemplateField>`、`<f:CheckBoxField>`、`<f:HyperLinkField>`（Core 不提供，用 `RenderField` + 渲染函数替代）。为四端一致，优先用 `RenderField`。

---

## 基础列（标题 / 字段 / 宽度）

列宽三选一：`width`（固定）、`flex`/`ExpandUnusedSpace`（占满剩余）、`minWidth`（最小宽）。

### F.js

```javascript
columns: [
    { columnType: 'rownumberfield' },
    { text: '姓名', field: 'Name', width: 120 },
    { text: '性别', field: 'Gender', align: 'center' },
    { text: '所学专业', field: 'Major', flex: 1, minWidth: 150 }
]
```

### FineUIPro (aspx)

```aspx
<Columns>
    <f:RowNumberField />
    <f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名" Width="120px" />
    <f:RenderField ColumnID="Gender" DataField="Gender" HeaderText="性别" TextAlign="Center" />
    <f:RenderField ColumnID="Major" DataField="Major" HeaderText="所学专业" ExpandUnusedSpace="true" MinWidth="150px" />
</Columns>
```

### Core 流式（MVC）

```csharp
.Columns(
    F.RowNumberField(),
    F.RenderField().HeaderText("姓名").DataField("Name").Width(120),
    F.RenderField().HeaderText("性别").DataField("Gender").TextAlign(TextAlign.Center),
    F.RenderField().HeaderText("所学专业").DataField("Major").ExpandUnusedSpace(true).MinWidth(150)
)
```

### Core TagHelper（RazorForms / RazorPages）

```html
<Columns>
    <f:RowNumberField />
    <f:RenderField HeaderText="姓名" DataField="Name" Width="120" />
    <f:RenderField HeaderText="性别" DataField="Gender" TextAlign="Center" />
    <f:RenderField HeaderText="所学专业" DataField="Major" ExpandUnusedSpace="true" MinWidth="150" />
</Columns>
```

---

## 格式化列：日期与数字

C# 三端用**内置渲染器** `Renderer` + `RendererArgument`（日期 `yyyy/MM/dd`；数字 `N2` 千分位两位小数、`P1` 百分比、`D10` 十进制整数），并配 `FieldType`。F.js 用 `fieldType` + `fieldFormat`（内部走 `F.format.dateRenderer` / `numberRenderer`）。

### F.js

```javascript
columns: [
    { text: '入学日期', field: 'EntranceDate', fieldType: 'date', fieldFormat: 'yyyy/MM/dd' },
    { text: '工资',     field: 'Salary',       fieldType: 'float', fieldFormat: 'n2' }
]
```

### FineUIPro (aspx)

```aspx
<f:RenderField ColumnID="EntranceDate" DataField="EntranceDate" HeaderText="入学日期"
    FieldType="Date" Renderer="Date" RendererArgument="yyyy/MM/dd" Width="150px" />
<f:RenderField ColumnID="Salary" DataField="Salary" HeaderText="工资"
    FieldType="Double" Renderer="Number" RendererArgument="N2" Width="150px" />
```

### Core 流式（MVC）

```csharp
F.RenderField().HeaderText("入学日期").DataField("EntranceDate")
    .FieldType(FieldType.Date).Renderer(Renderer.Date).RendererArgument("yyyy/MM/dd").Width(150),
F.RenderField().HeaderText("工资").DataField("Salary")
    .FieldType(FieldType.Double).Renderer(Renderer.Number).RendererArgument("N2").Width(150)
```

### Core TagHelper

```html
<f:RenderField HeaderText="入学日期" DataField="EntranceDate" FieldType="Date" Renderer="Date" RendererArgument="yyyy/MM/dd" Width="150" />
<f:RenderField HeaderText="工资" DataField="Salary" FieldType="Double" Renderer="Number" RendererArgument="N2" Width="150" />
```

> Pro 声明式 `BoundField` 也可格式化日期：`<f:BoundField DataField="LogTime" DataFormatString="{0:yyyy/MM/dd}" HeaderText="注册日期" />`。

---

## 自定义渲染

需要返回自定义 HTML（如链接、彩色标签）时：**F.js 直接写 `renderer` 内联函数**；**C# 三端用 `RendererFunction` 指向页面内的 JS 函数名**（函数签名同 F.js 的 `renderer`）。

### F.js（内联函数）

```javascript
{
    text: '状态', field: 'Status', width: 100,
    renderer: function (value, params) {
        var color = value === '在职' ? 'green' : 'gray';
        return '<span style="color:' + color + '">' + value + '</span>';
    }
}
```

### C# 三端（RendererFunction 指向 JS 函数）

```aspx
<%-- Pro --%>
<f:RenderField ColumnID="Status" DataField="Status" HeaderText="状态" RendererFunction="renderStatus" />
```
```csharp
// Core 流式
F.RenderField().HeaderText("状态").DataField("Status").RendererFunction("renderStatus")
```
```html
<!-- Core TagHelper -->
<f:RenderField HeaderText="状态" DataField="Status" RendererFunction="renderStatus" />
```
```html
<!-- 页面内定义 JS（三端通用），签名与 F.js renderer 一致 -->
<script>
    function renderStatus(value, params) {
        var color = value === '在职' ? 'green' : 'gray';
        return '<span style="color:' + color + '">' + value + '</span>';
    }
</script>
```

> **安全**：渲染函数返回的字符串会作为 HTML 插入单元格，函数内需自行保证内容可信、对不可信数据做转义（见 SKILL.md 约束 3）。

---

## 布尔展示列（只读复选框）

展示布尔字段（如"是否在校"）用只读复选框列。**注意**：这是**数据列**，与"整行复选框多选"（`checkboxSelect` / `EnableCheckBoxSelect`，见 [selection.md](selection.md)）是两回事。

```javascript
// F.js
{ text: '是否在校', field: 'AtSchool', columnType: 'checkboxfield' }
```
```aspx
<%-- Pro --%>
<f:RenderCheckField ColumnID="AtSchool" DataField="AtSchool" HeaderText="是否在校" EnableColumnEdit="false" />
```
```csharp
// Core 流式
F.RenderCheckField().HeaderText("是否在校").DataField("AtSchool").RenderAsStaticField(true)
```
```html
<!-- Core TagHelper -->
<f:RenderCheckField HeaderText="是否在校" DataField="AtSchool" RenderAsStaticField="true" />
```

---

## 固定（锁定）列

F.js：`lockable`（允许锁定）+ `locked`（当前锁定）。C# 三端：`EnableLock` + `Locked`。

```javascript
// F.js
{ text: '姓名', field: 'Name', lockable: true, locked: true }
```
```aspx
<%-- Pro --%>
<f:RenderField ColumnID="Name" DataField="Name" HeaderText="姓名" EnableLock="true" Locked="true" />
```
```csharp
// Core 流式
F.RenderField().HeaderText("姓名").DataField("Name").EnableLock(true).Locked(true)
```
```html
<!-- Core TagHelper -->
<f:RenderField HeaderText="姓名" DataField="Name" EnableLock="true" Locked="true" />
```

---

## F.js 列属性速查（`FineUI.GridColumn`）

| 属性 | 类型/可选值 | 说明 | C# 对照 |
|------|------------|------|---------|
| `text` | string | 列标题 | `HeaderText` |
| `field` | string | 数据字段 | `DataField` |
| `columnId` | string | 列标识符 | `ColumnID` |
| `width` / `flex` / `minWidth` / `maxWidth` | number | 列宽 | 同名（Pro 带 `px`） |
| `align` / `headerAlign` | left/right/center | 单元格/表头对齐 | `TextAlign` / `HeaderTextAlign` |
| `fieldType` | string/int/float/double/boolean/date | 字段类型 | `FieldType` |
| `fieldFormat` | string | 格式串（配合 fieldType 自动渲染） | `FieldFormat` |
| `renderer` | function(value, params) | 自定义渲染 | `RendererFunction`（JS 函数名） |
| `sortable` / `sortField` | boolean / string | 排序 | `SortField` |
| `lockable` / `locked` | boolean | 允许/当前锁定 | `EnableLock` / `Locked` |
| `hidden` | boolean | 隐藏（仍加载） | `Hidden` |
| `editable` / `editor` | boolean / Field | 单元格编辑 | 见 editing.md（规划中） |
| `columnType` | renderfield/rownumberfield/checkboxfield/rowexpanderfield | 列类型 | 对应列标签 |
| `summaryType` / `summaryRenderer` | sum/min/max/count/avg / fn | 合计行 | 见 paging-toolbar.md（规划中） |

## See also

- [selection.md](selection.md)：整行复选框选择（与布尔展示列不同）
- `fineui-form` 技能：单元格编辑器（`editor`）用到表单字段（规划中）
