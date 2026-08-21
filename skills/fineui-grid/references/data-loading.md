# Grid 数据加载与分页（Data Loading & Paging）

两种分页模式，务必先分清：

| 模式 | 何时用 | 关键点 |
|------|--------|--------|
| **内存分页** | 数据量小、一次性取回全部 | 只开分页；Grid 在前端切页 |
| **数据库（服务端）分页** | 数据量大、每页向服务端取 | 开数据库分页 + **设总记录数** + **翻页事件回发按页取数** |

---

## 一、内存分页

一次性给全部数据，Grid 前端切页。

```javascript
// F.js —— data 传全量数组，paging 切页
F.create({
    type: 'Grid', id: 'Grid1', isFluid: true, renderTo: '#wrap',
    paging: true, pageSize: 10, showPageSizeSelector: true,
    columns: [ /* ... */ ], idField: 'Id', data: allRows
});
```
```aspx
<%-- Pro：AllowPaging，不开 IsDatabasePaging；后台一次性绑定全量 --%>
<f:Grid ID="Grid1" runat="server" AllowPaging="true" PageSize="10" ShowPageSizeSelector="true"> ... </f:Grid>
<%-- 后台：Grid1.DataSource = allTable; Grid1.DataBind(); --%>
```
```csharp
// Core-MVC（Fluent API）
@(F.Grid().AllowPaging(true).PageSize(10).ShowPageSizeSelector(true)
    .Columns( /* ... */ ).DataSource(ViewBag.Grid1DataSource))   // 全量
```
```html
<!-- Core-TagHelper（RazorForms/RazorPages）：RazorPages 内联全量 DataSource，RazorForms 后台 DataBind -->
<f:Grid ID="Grid1" AllowPaging="true" PageSize="10" ShowPageSizeSelector="true" DataSource="@Model.GetAll()"> ... </f:Grid>
```
```html
<!-- FineUIJava（Thymeleaf 方言）：同 RazorForms，标签不写 data-source，页面类 Page_Load 一次性绑全量 -->
<f:grid id="Grid1" allow-paging="true" page-size="10" show-page-size-selector="true"> ... </f:grid>
```
```java
// FineUIJava 页面类：内存分页首屏绑一次，回发无需重绑
@FineUIPage("grid-paging/paging")
public class Paging extends FineUIPageBase {
    com.fineui.java.core.controls.Grid Grid1;
    public void Page_Load(Object sender, EventArgs e) {
        if (!isPostBack()) { Grid1.setDataSource(getAll()); Grid1.dataBind(); }
    }
}
```

---

## 二、数据库（服务端）分页

三步铁律：**① 开数据库分页 ② 每次绑定都设总记录数 ③ 翻页事件里按页取数并重绑**。

### F.js

```javascript
F.create({
    type: 'Grid', id: 'Grid1', isFluid: true, renderTo: '#wrap',
    paging: true, databasePaging: true, pageSize: 5,   // databasePaging=true 开启服务端分页
    columns: [ /* ... */ ], idField: 'Id',
    dataUrl: '/api/students'   // 翻页时携带页码请求；服务端返回该页数据 + 总记录数
});
```

### Pro（WebForms，aspx）

```aspx
<f:Grid ID="Grid1" runat="server" AllowPaging="true" IsDatabasePaging="true" PageSize="5"
        DataKeyNames="Id" OnPageIndexChanged="Grid1_PageIndexChanged"> ... </f:Grid>
```
```csharp
protected void Page_Load(object sender, EventArgs e) { if (!IsPostBack) BindGrid(); }

private void BindGrid() {
    Grid1.RecordCount = GetTotalCount();                                 // ① 必须设总记录数
    Grid1.DataSource  = GetPagedTable(Grid1.PageIndex, Grid1.PageSize);  // ② 按当前页取数
    Grid1.DataBind();
}
protected void Grid1_PageIndexChanged(object sender, GridPageEventArgs e) { BindGrid(); }  // ③ 翻页重绑
```

### Core-MVC（Fluent API）

```csharp
// View
@(F.Grid().ID("Grid1").AllowPaging(true).IsDatabasePaging(true).PageSize(5)
    .OnPageIndexChanged(Url.Action("Grid1_PageIndexChanged"), "Grid1")
    .Columns( /* ... */ )
    .RecordCount(ViewBag.Grid1RecordCount).DataSource(ViewBag.Grid1DataSource))
```
```csharp
// Controller
public IActionResult Index() {
    ViewBag.Grid1RecordCount = GetTotalCount();
    ViewBag.Grid1DataSource  = GetPagedTable(0, 5);
    return View();
}
[HttpPost, ValidateAntiForgeryToken]
public IActionResult Grid1_PageIndexChanged(string[] Grid1_fields, int Grid1_pageIndex) {  // 参数名为框架约定
    var grid1 = UIHelper.Grid("Grid1");
    grid1.RecordCount(GetTotalCount());
    grid1.DataSource(GetPagedTable(Grid1_pageIndex, 5), Grid1_fields);
    return UIHelper.Result();
}
```

### Core-RazorForms（TagHelper，后台事件）

```html
<f:Grid ID="Grid1" AllowPaging="true" IsDatabasePaging="true" PageSize="5"
        DataIDField="Id" OnPageIndexChanged="Grid1_PageIndexChanged"> <Columns> ... </Columns> </f:Grid>
```
```csharp
protected void Page_Load(object sender, EventArgs e) { if (!IsPostBack) LoadData(); }
private void LoadData() {
    Grid1.RecordCount = GetTotalCount();
    Grid1.DataSource  = GetPagedTable(Grid1.PageIndex, Grid1.PageSize);
    Grid1.DataBind();
}
protected void Grid1_PageIndexChanged(object sender, GridPageEventArgs e) { LoadData(); }
```

### Core-RazorPages（TagHelper，OnPost 处理器）

```html
<f:Grid ID="Grid1" AllowPaging="true" IsDatabasePaging="true" PageSize="5"
        RecordCount="@ViewBag.Grid1RecordCount" DataSource="@ViewBag.Grid1DataSource"
        OnPageIndexChanged="@Url.Handler(&quot;Grid1_PageIndexChanged&quot;)" OnPageIndexChangedFields="Grid1">
    <Columns> ... </Columns>
</f:Grid>
```
```csharp
public void OnGet() {
    ViewBag.Grid1RecordCount = GetTotalCount();
    ViewBag.Grid1DataSource  = GetPagedTable(0, 5);
}
public IActionResult OnPostGrid1_PageIndexChanged(string[] Grid1_fields, int Grid1_pageIndex) {
    var grid1 = UIHelper.Grid("Grid1");
    grid1.RecordCount(GetTotalCount());
    grid1.DataSource(GetPagedTable(Grid1_pageIndex, 5), Grid1_fields);
    return UIHelper.Result();
}
```

### FineUIJava（Thymeleaf 方言，后台事件）

结构与 RazorForms 一致：`on-page-index-changed` 指向页面类 `void` 处理器；处理器读 `Grid1.getPageIndex()` 按页取数、每次都 `setRecordCount(...)`。

```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:grid id="Grid1" allow-paging="true" is-database-paging="true" page-size="5"
        data-id-field="Id" on-page-index-changed="Grid1_PageIndexChanged"> <f:columns> ... </f:columns> </f:grid>
```
```java
// FineUIJava 页面类
@FineUIPage("grid-paging/database")
public class Database extends FineUIPageBase {
    com.fineui.java.core.controls.Grid Grid1;
    public void Page_Load(Object sender, EventArgs e) { if (!isPostBack()) loadData(); }
    private void loadData() {
        Grid1.setRecordCount(getTotalCount());                                     // ① 每次绑定都设总记录数
        Grid1.setDataSource(getPaged(Grid1.getPageIndex(), Grid1.getPageSize()));  // ② 按当前页取数
        Grid1.dataBind();
    }
    public void Grid1_PageIndexChanged(Object sender, GridPageEventArgs e) { loadData(); }  // ③ 翻页重绑（void）
}
```

---

## 关键约束

1. **数据库分页必须设总记录数**：F.js 由服务端随数据返回；C# 三模式每次绑定都 `RecordCount = 总数`（Fluent `.RecordCount(...)` / TagHelper `RecordCount="..."` / 属性 `Grid1.RecordCount`）；**Java 每次绑定都 `Grid1.setRecordCount(总数)`**。漏设 → 分页栏页数不对。
2. **开关属性名**：F.js `databasePaging: true`；C# 三模式 `IsDatabasePaging="true"`；**Java `is-database-paging="true"`**（不开则为内存分页）。
3. **翻页事件各栈不同**：
   - Core-MVC：`.OnPageIndexChanged(Url.Action("..."), "Grid1")` → Controller `Xxx(string[] Grid1_fields, int Grid1_pageIndex)`。
   - Core-RazorForms：`OnPageIndexChanged="方法名"` → 后台 `方法名(object sender, GridPageEventArgs e)`（服务端事件，读 `Grid1.PageIndex`）。
   - Core-RazorPages：`OnPageIndexChanged="@Url.Handler(\"方法名\")"` + `OnPageIndexChangedFields="Grid1"` → `OnPost方法名(string[] Grid1_fields, int Grid1_pageIndex)`。
   - Pro：`OnPageIndexChanged="方法名"` → `方法名(object sender, GridPageEventArgs e)`。
   - **Java**：`on-page-index-changed="方法名"` → 页面类 `public void 方法名(Object sender, GridPageEventArgs e)`（服务端事件，读 `Grid1.getPageIndex()`；**返回 void**）。
4. **回发按页取数用 `Grid1_pageIndex`（MVC/RazorPages）或 `Grid1.PageIndex`/`Grid1.getPageIndex()`（Pro/RazorForms/Java）**，配合 `Grid1_fields`（MVC/RazorPages）保持列。
5. **“加载更多/流式追加”**：Core-MVC 翻页处理器里用 `grid1.AppendData(dataSource, Grid1_fields)` 代替 `DataSource(...)`；**Java 用 `Grid1.appendData(nextPageList)` 追加下一页**（可把已加载页码存进 `Grid1.setAttribute("data-index", ...)` 随回发往返）。

## See also

- [paging-toolbar.md](paging-toolbar.md)：分页工具栏、页大小选择器、窄屏简洁分页
- [selection.md](selection.md)：`DataKeyNames` 与服务端读取选中行
