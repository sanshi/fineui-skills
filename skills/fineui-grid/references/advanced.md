# Grid 高级功能（列锁定 / 拖拽排序 / 单元格合并 / 大数据）

四个进阶能力。**要点：合并与拖拽的实际动作都在客户端 JS 完成，没有服务端"合并/拖拽"属性；持久化列/行顺序靠自定义回发。**

## 概念 → 各写法属性名对照

| 概念 | F.js | Pro (WebForms) | Core-MVC (Fluent) | Core-TagHelper | Java（Thymeleaf 方言） |
|------|------|----------------|-------------------|----------------|------------------------|
| 启用列锁定 | `columnLocking: true` | `AllowColumnLocking="true"` | `.AllowColumnLocking(true)` | `AllowColumnLocking="true"` | `allow-column-locking="true"` |
| 允许锁到右侧 | `columnLockingRight: true` | `ColumnLockingRight="true"` | `.ColumnLockingRight(true)` | `ColumnLockingRight="true"` | `column-locking-right="true"` |
| 列可锁 / 初始锁 | 列 `lockable` / `locked` | `EnableLock` / `Locked` | `.EnableLock().Locked()` | `EnableLock` / `Locked` | `enable-lock` / `locked` |
| 列锁到右侧 | 列 `lockedPosition:'right'` | `LockedPosition="Right"` | `.LockedPosition(LockedPosition.Right)` | `LockedPosition="Right"` | `locked-position="Right"` |
| 启用列拖拽 | `columnMoving: true` | `EnableColumnMove="true"` | `.EnableColumnMove(true)` | `EnableColumnMove="true"` | `enable-column-move="true"` |
| 启用大数据 | `bigData: true` | `EnableBigData="true"` | `.EnableBigData(true)` | `EnableBigData="true"` | `enable-big-data="true"` |

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
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:grid ... allow-column-locking="true" column-locking-right="true">
    <f:columns>
        <f:render-field data-field="Name" header-text="姓名" enable-lock="true" locked="true"></f:render-field>
        <f:render-field data-field="EntranceYear" header-text="入学年份" enable-lock="true" locked="true" locked-position="Right"></f:render-field>
    </f:columns>
</f:grid>
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
    F.customEvent('Grid1_ColumnMove', { columnIds: columnIds });
}
```
```aspx
<%-- Pro / Core-TagHelper —— 监听 columnmove --%>
<f:Grid ... EnableColumnMove="true">
    <Listeners><f:Listener Event="columnmove" Handler="onGrid1ColumnMove" /></Listeners>
</f:Grid>
```
```html
<!-- FineUIJava（Thymeleaf 方言）—— 监听 columnmove（handler 的 JS 体同 F.js，不重复贴）-->
<f:grid ... enable-column-move="true">
    <f:listeners><f:listener event="columnmove" handler="onGrid1ColumnMove" /></f:listeners>
</f:grid>
```

**服务端持久化列顺序**（Pro / Core RazorForms / Java 均用 `Page_CustomEvent`；应用保存的顺序用列的 `ColumnOrder`）：

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
```java
// FineUIJava —— 同样用 Page_CustomEvent 接收前端上报的列布局；回显用列的 setColumnOrder/setWidth/setHidden
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("Grid1_ColumnMove".equals(e.getEventName())) {
        session().setAttribute(KEY, e.getArgument());   // 保存 JSON（含各列 columnId/width/hidden）
    }
}
// 回显（Page_Load 里）：GridColumn col = Grid1.getColumnById(columnId);
//                       col.setColumnOrder(order); col.setWidth(w); col.setHidden(true);
```

> **注**：Java 示例里的 `session()` 是**页面类里自定义的私有辅助方法**（`return ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest().getSession();`），**不是** `FineUIPageBase` 内置方法——照抄时需自行定义，或直接注入/获取 `HttpSession`。

> 只允许同组内移动：`EnableSameGroupColumnMove="true"`（配合多表头）。Java 侧多表头列拖拽示例仅见 `enable-column-move="true"`（同组限制属性未在示例中出现，用时以官网 API 为准）。

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

```java
// FineUIJava —— 同 Core：客户端 F.customEvent('Grid1_RowMove', {rowIds:[...]}) 触发，页面类 Page_CustomEvent 接收保存
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("Grid1_RowMove".equals(e.getEventName())) {
        session().setAttribute(KEY, e.getArgument());   // 按上报的 rowIds 顺序重排数据源
        showNotify("数据保存成功！");
    }
}
```

Pro、Core RazorForms 与 Java 统一使用结构化自定义事件：

```javascript
F.customEvent('Grid1_RowMove', { rowIds: rowIds });
```

Pro 后台同样在 `Page_CustomEvent` 中通过 `e.EventName` 分派，并从 `e.EventArgumentsAsJObject` 读取 `rowIds`。不要使用 `__doPostBack` 拼接 `$` / `#` 字符串，再在 `Page_Load` 中手工拆分。

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

```html
<!-- FineUIJava（Thymeleaf 方言）：dataload 监听 + 客户端方法（JS 与 F.js 完全相同，不重复贴） -->
<f:grid id="Grid1" ... enable-column-lines="true">
    <f:columns> ... </f:columns>
    <f:listeners><f:listener event="dataload" handler="onGridDataLoad"></f:listener></f:listeners>
</f:grid>
<!-- script 槽：function onGridDataLoad(event){ this.mergeCells([...]); }（或 this.mergeColumns(['Major','Group']);） -->
```

> **`mergeCells`（手动、可跨列）≠ `mergeColumns`（自动、按列纵向）**。**单元格合并与行扩展列不能同时用**（行扩展时每行是独立 `<table>`）。Core 有独立 `GridMerge` 示例区；Java 对应 `grid-merge/` 目录（`cells.html` / `columns.html`），合并均为客户端方法，无服务端属性。

---

## 5. 大数据表格（Big Data，虚拟化）

海量行虚拟渲染。开 `EnableBigData="true"` + `FixedRowHeight="true"`（固定行高是前提）+ 通常隐藏分页栏 `PagingToolbarVisible="false"`，可开行提示 `EnableBigDataRowTip="true"`。**各栈（含 Java）都有大数据示例，不是 Pro 专属。**

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
```html
<!-- FineUIJava（Thymeleaf 方言）：网址数据源用 data-url；大数据 + 分页再加 allow-paging + page-size -->
<f:grid id="Grid1" height="500" data-url="/grid-big-data/big-data-url-data?total=10000&amp;type=simple"
        enable-big-data="true" fixed-row-height="true" enable-big-data-row-tip="true" paging-toolbar-visible="false">
    <f:columns> ... </f:columns>
</f:grid>
```

> **大数据模式限制**（示例注释）：不支持树表格、行分组、单元格编辑、列锁定、单元格合并、模板列放输入字段；要求每行行高相同（`FixedRowHeight="true"`）且表格高度固定/在布局中。

---

## 关键约束

1. **Grid 级 vs 列级锁定名字不同**：Grid `AllowColumnLocking`（Java `allow-column-locking`）；列 `EnableLock`+`Locked`（Java `enable-lock`+`locked`）。别互换。
2. **列/行拖拽无服务端事件**：`columnmove` 客户端签名 `(event, targetColumnId, sourceColumnId, operation)`；持久化统一用 `F.customEvent` + `Page_CustomEvent`（C# 读 `e.EventArgumentsAsJObject`，Java 读 `e.getEventName()` / `e.getArgument()`；回显用列 `ColumnOrder` / `setColumnOrder(...)`）。行拖拽本身是 `moveRowUp`/`moveRowDown`/`addNewRecords`/`deleteRows` 客户端方法，无 `EnableRowDragDrop` 属性。
3. **合并是客户端方法**：`mergeCells`（手动跨行列）/ `mergeColumns`（自动纵向），在 `dataload` 里调，无服务端属性；与行扩展列互斥。
4. **大数据前提是固定行高**，且与树/分组/编辑/锁定/合并互斥。

## See also

- [columns.md](columns.md)：列锁定基础属性；[header.md](header.md)：多表头（同组列拖拽）
- [row-features.md](row-features.md)：行扩展列（与合并互斥）、行高
- [row-group.md](row-group.md) / [tree-grid.md](tree-grid.md)：大数据不支持的两类
