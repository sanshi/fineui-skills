# Grid 树表格（Tree Grid）

行与行之间是父子层级的表格（如文件夹树）。**树表格不是独立组件——就是普通 `Grid` 加几个"树"属性**。父子关系用**扁平列表**（每行带自己的 id + 父 id）表达，**不是嵌套 children 数组**。

> 区分：**这是"表格的行成树"**。独立的 Tree 控件见 `fineui-tree` 技能；列的分组（多表头）见 [header.md](header.md)；数据行按字段分组见 [row-group.md](row-group.md)。

## 概念 → 各写法属性名对照

| 概念 | F.js | .NET（Pro / Core-MVC / Core-TagHelper） | Java（Thymeleaf 方言） |
|------|------|------------------------------------------|------------------------|
| 开启树 | `tree: { columnId: 'Name' }` | `EnableTree="true"` + `TreeColumn="Name"` | `enable-tree="true"` + `tree-column="Name"` |
| 行 ID 字段 | `idField: 'Id'` | `DataIDField="Id"` | `data-id-field="Id"` |
| 父 ID 字段 | `parentIdField: 'ParentId'` | `DataParentIDField="ParentId"` | `data-parent-id-field="ParentId"` |
| 文本字段 | `textField: 'Name'` | `DataTextField="Name"` | `data-text-field="Name"` |
| 树序号列 | 列 `treeNumber: true` | `<f:RowNumberField EnableTreeNumber="true" />` | `<f:row-number-field enable-tree-number="true">` |
| 展开全部 | `tree: { expanded: true }` | `ExpandAllTreeNodes="true"` | `expand-all-tree-nodes="true"` |
| 关闭图标 | `tree: { icons: false }` | `EnableTreeIcons="false"` | `enable-tree-icons="false"` |
| 行复选框 | `tree: { checkbox: true }` | `TreeCheckBox="true"` | `tree-check-box="true"` |
| 级联勾选 | `tree: { cascadeCheck: true }` | `TreeCascadeCheck="true"` | `tree-cascade-check="true"` |
| 仅叶子/仅目录可勾 | `tree: { onlyLeafCheck / onlyFolderCheck: true }` | `TreeOnlyLeafCheck / TreeOnlyFolderCheck="true"` | `tree-only-leaf-check / tree-only-folder-check="true"` |

---

## 1. 基础树表格

**数据是扁平表**：每行有 `Id` 与 `ParentId`。**根节点**：F.js JSON 里 `ParentId` 为空串 `""`；C# `DataTable` 里约定用 `-1`。

```javascript
// F.js —— tree.columnId 指定"显示层级缩进"的列
F.create({
    type: 'Grid', isFluid: true, id: 'grid1', renderTo: '#wrap', title: '树表格', checkboxSelect: true,
    columns: [
        { columnType: 'rownumberfield' },
        { text: '名称', field: 'Name', flex: 1, minWidth: 150 },
        { text: '类型', field: 'Type', width: 150 }
    ],
    idField: 'Id', textField: 'Name', parentIdField: 'ParentId',
    tree: { columnId: 'Name' },
    dataUrl: './data_treegrid.txt'    // 扁平数组，根行 ParentId 为 ""
});
```
```aspx
<%-- Pro：叶子列使用 RenderField，与 Core / Java 保持一致 --%>
<f:Grid ID="Grid1" runat="server" IsFluid="true" Title="树表格" DataKeyNames="Id,Name"
        EnableTree="true" TreeColumn="Name" DataIDField="Id" DataParentIDField="ParentId">
    <Columns>
        <f:RowNumberField />
        <f:RenderField ColumnID="Name" DataField="Name" HeaderText="名称" ExpandUnusedSpace="true" MinWidth="150px" />
        <f:RenderField ColumnID="Type" DataField="Type" HeaderText="类型" Width="150px" />
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().ID("Grid1").IsFluid(true).Title("树表格").EnableTree(true).TreeColumn("Name")
    .DataIDField("Id").DataParentIDField("ParentId").DataTextField("Name")
    .Columns(F.RowNumberField(), F.RenderField().HeaderText("名称").DataField("Name").ExpandUnusedSpace(true).MinWidth(150))
    .DataSource(ViewBag.Grid1DataSource))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages）-->
<f:Grid ID="Grid1" IsFluid="true" Title="树表格" DataIDField="Id" DataTextField="Name"
        DataSource="@DataSourceUtil.GetTreeDataTable()" EnableTree="true" TreeColumn="Name" DataParentIDField="ParentId">
    <Columns> ... </Columns>
</f:Grid>
```
```html
<!-- FineUIJava（Thymeleaf 方言）：三个树属性 + 树列，数据（扁平 id/parentId 列表）在页面类 Page_Load 绑定 -->
<f:grid id="Grid1" is-fluid="true" title="树表格" data-id-field="Id" data-text-field="Name"
        data-parent-id-field="ParentId" enable-tree="true" tree-column="Name">
    <f:columns>
        <f:row-number-field></f:row-number-field>
        <f:render-field column-id="Name" header-text="名称" data-field="Name" expand-unused-space="true" min-width="150"></f:render-field>
        <f:render-field header-text="类型" data-field="Type" width="150"></f:render-field>
    </f:columns>
</f:grid>
```
```java
// FineUIJava 页面类：扁平树数据（每行含 Id/ParentId），服务端绑定
@FineUIPage("grid-tree/grid-tree")
public class GridTree extends FineUIPageBase {
    com.fineui.java.core.controls.Grid Grid1;
    public void Page_Load(Object sender, EventArgs e) {
        if (!isPostBack()) { Grid1.setDataSource(getTreeRows()); Grid1.dataBind(); }
    }
}
```

### C# 服务端建树数据（扁平表，根 `ParentId = -1`）

```csharp
public static DataTable GetTreeDataTable() {
    DataTable table = new DataTable();
    table.Columns.Add(new DataColumn("Id", typeof(int)));
    table.Columns.Add(new DataColumn("ParentId", typeof(int)));
    table.Columns.Add(new DataColumn("Name", typeof(String)));
    // 根节点：ParentId = -1
    DataRow row = table.NewRow(); row["Id"] = 50; row["ParentId"] = -1; row["Name"] = "basic"; table.Rows.Add(row);
    // 子节点：ParentId = 父的 Id
    row = table.NewRow(); row["Id"] = 54; row["ParentId"] = 50; row["Name"] = "Captcha"; table.Rows.Add(row);
    return table;
}
// 绑定：Grid1.DataSource = GetTreeDataTable(); Grid1.DataBind();
```

---

## 2. 展开、序号、图标

- **树序号列**：`<f:RowNumberField EnableTreeNumber="true" />`（F.js 列 `treeNumber: true`）。
- **展开全部**：Grid `ExpandAllTreeNodes="true"`（F.js `tree: { expanded: true }`）。
- **展开指定行 / 单行图标**：各栈方式不同（见下）。

**Pro 用服务端 `GridRowEventArgs`**；**Core 三模式与 F.js 用客户端 `RowDataBoundFunction` / `rowDataBound`**：

```csharp
// Pro —— 服务端行数据绑定事件
protected void Grid1_RowDataBound(object sender, GridRowEventArgs e) {
    if (e.RowID == "50" || e.RowID == "60") e.TreeNodeExpanded = true;
    // 图标：e.TreeNodeIconFont = IconFont._Pic;  或  e.TreeNodeIconUrl = "~/res/images/filetype/vs_png.png";
}
```
```javascript
// F.js / Core（RowDataBoundFunction 指向的 JS 函数，签名相同）
function onGrid1RowDataBound(rowData) {
    if (rowData.id == '50') rowData.expanded = true;
    // 图标：rowData.iconFont = 'f-iconfont-pic';  或  rowData.icon = '.../vs_png.png';
}
```

> **关键差异**：Pro 走服务端 `e.TreeNode*`（`TreeNodeExpanded`/`TreeNodeChecked`/`TreeNodeCheckBoxDisabled`/`TreeNodeIconFont`/`TreeNodeIconUrl`）；Core 三模式与 **Java** 声明 `RowDataBoundFunction`（**Java `row-data-bound-function="onGrid1RowDataBound"`**），在 JS 里设 `rowData.expanded`/`checked`/`checkboxDisabled`/`icon`/`iconFont`（函数体各栈相同）。

---

## 3. 复选框树 + 级联

```javascript
// F.js
tree: { columnId: 'Name', checkbox: true, cascadeCheck: true }   // 加 onlyLeafCheck / onlyFolderCheck 限制
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... EnableTree="true" TreeColumn="Name" DataIDField="Id" DataParentIDField="ParentId" DataTextField="Name"
        TreeCheckBox="true" TreeCascadeCheck="true"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().EnableTree(true).TreeColumn("Name").TreeCheckBox(true).TreeCascadeCheck(true) ...)
```
```html
<!-- FineUIJava（Thymeleaf 方言）：加 tree-only-leaf-check / tree-only-folder-check 限制可勾范围 -->
<f:grid ... enable-tree="true" tree-column="Name" data-id-field="Id" data-parent-id-field="ParentId" data-text-field="Name"
        tree-check-box="true" tree-cascade-check="true"> ... </f:grid>
```

**初始勾选 / 禁用某行复选框**：Pro 服务端 `e.TreeNodeChecked = true` / `e.TreeNodeCheckBoxDisabled = true`；Core/F.js/Java 客户端 `rowData.checked = true` / `rowData.checkboxDisabled = true`（Java 用 `row-data-bound-function` 指向该 JS）。

**读取勾选行**：F.js/Core/Java 客户端 `F.ui.Grid1.getCheckedRows(true)`（Java 收集后可 `F.customEvent('GetCheckedRows', result)` 回发到页面类 `Page_CustomEvent`）；Pro 服务端 `Grid1.GetCheckedRows()`。

---

## 4. 分页、右键菜单、大数据

- **分页**：就用普通分页属性（`AllowPaging` / `IsDatabasePaging` / `PageSize`）——**只对顶层节点分页**，每页把顶层节点的整棵子树一并带出。总记录数按顶层节点数（`ParentId == -1` 的行数）。
- **右键菜单**：无树专用属性——用 Grid 事件 `beforerowcontextmenu` + 客户端方法 `F.ui.grid1.expandRow(rowId, true)` / `collapseRow(rowId, true)` / `getRowData(rowId).leaf`。
- **大数据树**：无独立属性，就是节点多的普通树表格（配 `treeNumber` 序号列）。

```javascript
// F.js —— 右键菜单里用树方法
listeners: { beforerowcontextmenu: function (event, rowId) {
    var leaf = F.ui.grid1.getRowData(rowId).leaf;   // 叶子？决定"展开"菜单是否禁用
    // 菜单项动作：F.ui.grid1.expandRow(rowId, true) / F.ui.grid1.collapseRow(rowId, true)
} }
```

---

## 关键约束

1. **树表格 = Grid + 树属性**，不是独立组件；数据是**扁平 id/parentId 列表**（根：F.js `""`、C# `-1`），不是嵌套 children。
2. **F.js 嵌套 `tree:{}` vs .NET/Java 拍平**：F.js 所有树选项在 `tree:{}` 内（`columnId`/`checkbox`/`cascadeCheck`/`onlyLeafCheck`/`onlyFolderCheck`/`expanded`/`icons`）；.NET 与 Java 是 `EnableTree`/`TreeColumn`/`TreeCheckBox`/`TreeCascadeCheck`/... 独立属性（**Java kebab-case**：`enable-tree`/`tree-column`/`tree-check-box`/`tree-cascade-check`/`tree-only-leaf-check`/`enable-tree-icons`）。
3. **逐行定制两套 API**：Pro 服务端 `GridRowEventArgs.e.TreeNode*`；Core/F.js/**Java** 客户端 `RowDataBoundFunction`/`row-data-bound-function`/`rowDataBound` 设 `rowData.*`。
4. **分页只分顶层节点**，无"树分页"专用开关。

## See also

- `fineui-tree`：独立的 Tree 控件（与树表格不同）
- [selection.md](selection.md)：`getCheckedRows` 读勾选
- [header.md](header.md)：多表头（列分组）；[row-group.md](row-group.md)：行分组
