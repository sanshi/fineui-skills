# Grid 高级功能（列锁定 / 拖拽排序 / 单元格合并 / 大数据）

四个进阶能力。**要点：合并与拖拽的实际动作都在客户端 JS 完成，没有服务端"合并/拖拽"属性；持久化列/行顺序靠自定义回发。**

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent) | Core-TagHelper |
|------|------|----------------|-------------------|----------------|
| 启用列锁定 | `columnLocking: true` | `AllowColumnLocking="true"` | `.AllowColumnLocking(true)` | `AllowColumnLocking="true"` |
| 允许锁到右侧 | `columnLockingRight: true` | `ColumnLockingRight="true"` | `.ColumnLockingRight(true)` | `ColumnLockingRight="true"` |
| 列可锁 / 初始锁 | 列 `lockable` / `locked` | `EnableLock` / `Locked` | `.EnableLock().Locked()` | `EnableLock` / `Locked` |
| 列锁到右侧 | 列 `lockedPosition:'right'` | `LockedPosition="Right"` | `.LockedPosition(LockedPosition.Right)` | `LockedPosition="Right"` |
| 启用列拖拽 | `columnMoving: true` | `EnableColumnMove="true"` | `.EnableColumnMove(true)` | `EnableColumnMove="true"` |
| 启用大数据 | `bigData: true` | `EnableBigData="true"` | `.EnableBigData(true)` | `EnableBigData="true"` |

---

## 1. 列锁定（Column Locking）

Grid 级开 `AllowColumnLocking`（**不是 `EnableLock`**），列级用 `EnableLock`（可锁）+ `Locked`（初始锁）。右侧锁定需 Grid `ColumnLockingRight="true"` + 列 `LockedPosition="Right"`。

```javascript
// F.js —— 左锁两列 + 一列锁右侧
{ type: 'Grid', columnLocking: true, columnLockingRight: true,
  columns: [
      { text: '姓名', field: 'Name', lockable: true, locked: true },
      { text: '入学年份', field: 'EntranceYear', lockable: true, locked: true, lockedPosition: 'right' }
  ] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ... AllowColumnLocking="true" ColumnLockingRight="true">
    <Columns>
        <f:RenderField DataField="Name" HeaderText="姓名" EnableLock="true" Locked="true" />
        <f:RenderField DataField="EntranceYear" HeaderText="入学年份" EnableLock="true" Locked="true" LockedPosition="Right" />
    </Columns>
</f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().AllowColumnLocking(true).ColumnLockingRight(true)
    .Columns(F.RenderField().HeaderText("姓名").DataField("Name").EnableLock(true).Locked(true)) ...)
```

> 列锁定可与**行展开、多表头、合计行**共存，无需额外属性——同时配置即可。

---

## 2. 列拖拽排序（Column Move）

Grid 开 `EnableColumnMove="true"`（F.js `columnMoving: true`）。列顺序变化触发客户端 `columnmove` 事件，**没有内置服务端事件**——要持久化顺序得自定义回发。

```javascript
// F.js / 各栈的 columnmove 监听器（签名相同）
function onGrid1ColumnMove(event, targetColumnId, sourceColumnId, operation) {
    var grid = this;
    var columnIds = $.map(grid.columns, function (c) { return c.columnId; });
    // 把 columnIds 存后台（自定义回发）
}
```
```aspx
<%-- Pro / Core-TagHelper —— 监听 columnmove --%>
<f:Grid ... EnableColumnMove="true">
    <Listeners><f:Listener Event="columnmove" Handler="onGrid1ColumnMove" /></Listeners>
</f:Grid>
```

**服务端持久化列顺序**（Core 用自定义事件 `Page_CustomEvent`；应用保存的顺序用列的 `ColumnOrder`）：

```csharp
// Core-RazorForms —— 接收自定义事件
protected void Page_CustomEvent(object sender, CustomEventArgs e) {
    if (e.EventName == "Grid1_ColumnMove") {
        var columnIds = e.EventArgumentsAsJObject.Value<JArray>("columnIds");
        HttpContext.Session.SetObject(KEY, columnIds);
    }
}
// 回显：Grid1.FindColumn(columnId).ColumnOrder = order;
```

> 只允许同组内移动：`EnableSameGroupColumnMove="true"`（配合多表头）。

---

## 3. 行拖拽排序（Row Move）

**没有启用原生行拖拽的 Grid 属性**（`EnableRowDragDrop`/`AllowRowMove` 均不存在）。"行移动"是按钮 + 客户端方法实现：

```javascript
// 上移/下移选中行
grid.moveRowUp(grid.getSelectedRow());
grid.moveRowDown(grid.getSelectedRow());

// 两个表格间移动行
function moveRight(rowIds) {
    var rowDatas = $.map(rowIds, function (id) { return gridLeft.getRowData(id); });
    gridRight.addNewRecords(rowDatas, true);   // 加到右表
    gridLeft.deleteRows(rowIds, true);         // 从左表删
}
```

**服务端保存行顺序**（Core 自定义事件收 `JArray rowIds`）：

```csharp
protected void Page_CustomEvent(object sender, CustomEventArgs e) {
    if (e.EventName == "Grid1_RowMove") {
        JArray rowIds = e.EventArgumentsAsJObject.Value<JArray>("rowIds");
        DataTable newTable = original.Clone();
        foreach (string rowId in rowIds) newTable.ImportRow(FindRow(rowId, original));
        HttpContext.Session.SetObject(KEY, newTable);
        ShowNotify("数据保存成功！");
    }
}
```

> Pro 用 `__doPostBack('', 'RowMove_' + rowIds.join('#'))`，在 `Page_Load` 里 `GetRequestEventArgument()` 解析。

---

## 4. 单元格合并（Merge）

**合并全是客户端 Grid 方法，在 `dataload` 监听器里调用；没有服务端合并属性。** 各栈方法名与参数一致，只是监听器接线语法不同。通常配 `EnableColumnLines="true"` 显竖线。

| 方法 | 作用 |
|------|------|
| `mergeCells([{ rowId, columnId, rowspan, colspan }])` | 手动指定单元格合并，可跨行(`rowspan`)**且跨列**(`colspan`) |
| `mergeColumns(['Major','Group','LogTime'])` | 对每列把上下相邻、值相同的单元格**纵向**合并 |
| `mergeColumns([...], { depends: true })` | 后列合并依赖前列是否已合并 |

```javascript
// F.js —— dataload 里调用（Pro/Core 用 <Listeners><f:Listener Event="dataload" .../>）
listeners: {
    dataload: function (event) {
        this.mergeCells([
            { rowId: 'R1', columnId: 'EntranceYear', rowspan: 3 },
            { rowId: 'R9', columnId: 'Group1', rowspan: 4, colspan: 2 }   // 跨行且跨列
        ]);
        // 或按列纵向合并相同值：this.mergeColumns(['Major', 'Group', 'LogTime']);
    }
}
```

> **`mergeCells`（手动、可跨列）≠ `mergeColumns`（自动、按列纵向）**。**单元格合并与行扩展列不能同时用**（行扩展时每行是独立 `<table>`）。Core 有独立 `GridMerge` 示例区；Pro 合并示例在 `grid` 目录（`grid_mergecells.aspx` / `grid_mergecolumns.aspx`）。

---

## 5. 大数据表格（Big Data，虚拟化）

海量行虚拟渲染。开 `EnableBigData="true"` + `FixedRowHeight="true"`（固定行高是前提）+ 通常隐藏分页栏 `PagingToolbarVisible="false"`，可开行提示 `EnableBigDataRowTip="true"`。**五个栈都有大数据示例，不是 Pro 专属。**

```javascript
// F.js —— 万级数据
{ type: 'Grid', bigData: true, pagingToolbarVisible: false, bigDataRowTip: true,
  columns: [ ... ], idField: 'Id', dataUrl: './data_bigdata_1000.txt' }
// 大数据 + 分页：再加 paging: true, pageSize: 120
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Grid ID="Grid1" runat="server" Height="500px" EnableBigData="true" FixedRowHeight="true"
        EnableBigDataRowTip="true" PagingToolbarVisible="false"> <Columns> ... </Columns> </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid().EnableBigData(true).FixedRowHeight(true).EnableBigDataRowTip(true).PagingToolbarVisible(false) ...)
```

> **大数据模式限制**（示例注释）：不支持树表格、行分组、单元格编辑、列锁定、单元格合并、模板列放输入字段；要求每行行高相同（`FixedRowHeight="true"`）且表格高度固定/在布局中。

---

## 关键约束

1. **Grid 级 vs 列级锁定名字不同**：Grid `AllowColumnLocking`；列 `EnableLock`+`Locked`。别互换。
2. **列/行拖拽无服务端事件**：`columnmove` 客户端签名 `(event, targetColumnId, sourceColumnId, operation)`；持久化靠自定义回发（Core `Page_CustomEvent`，Pro `__doPostBack`）。行拖拽本身是 `moveRowUp`/`moveRowDown`/`addNewRecords`/`deleteRows` 客户端方法，无 `EnableRowDragDrop` 属性。
3. **合并是客户端方法**：`mergeCells`（手动跨行列）/ `mergeColumns`（自动纵向），在 `dataload` 里调，无服务端属性；与行扩展列互斥。
4. **大数据前提是固定行高**，且与树/分组/编辑/锁定/合并互斥。

## See also

- [columns.md](columns.md)：列锁定基础属性；[header.md](header.md)：多表头（同组列拖拽）
- [row-features.md](row-features.md)：行扩展列（与合并互斥）、行高
- [row-group.md](row-group.md) / [tree-grid.md](tree-grid.md)：大数据不支持的两类
