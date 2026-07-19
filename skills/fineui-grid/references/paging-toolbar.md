# 分页工具栏与 PageManager 全局配置（Paging Toolbar & Global Config）

分页工具栏的页大小选择器、窄屏简洁分页、自定义分页项，以及“全局设一次、各 Grid 生效”的 PageManager/配置文件写法。

> 先决条件：分页本身见 [data-loading.md](data-loading.md)。本文只讲工具栏与全局配置。

## 1. 页大小选择器

内置选择器：开 `ShowPageSizeSelector`，可选 `PageSizeOptions`（不设用默认）。

```javascript
// F.js
paging: true, pageSize: 10, showPageSizeSelector: true   // 可选：pageSizeOptions: [10, 20, 50, 100]
```
```aspx
<%-- Pro --%>
<f:Grid ... AllowPaging="true" ShowPageSizeSelector="true" PageSizeOptions="5,10,15,20"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid()...AllowPaging(true).ShowPageSizeSelector(true).PageSizeOptions("5,10,15,20"))
```
```html
<!-- Core-TagHelper（RazorForms/RazorPages）-->
<f:Grid ... AllowPaging="true" ShowPageSizeSelector="true" PageSizeOptions="5,10,15,20"> ... </f:Grid>
```

---

## 2. 窄屏自动简洁分页（pagerAutoSimpleMode）

分页栏放不下时自动降级为“上一页 + 当前/总页 + 下一页”，拉宽后还原。**可逐 Grid，也可全局**。

### 逐 Grid

```javascript
// F.js
pagerAutoSimpleMode: true
```
```aspx
<%-- Pro --%>
<f:Grid ... PagerAutoSimpleMode="true"> ... </f:Grid>
```
```csharp
// Core-MVC（Fluent）
@(F.Grid()...PagerAutoSimpleMode(true))
```
```html
<!-- Core-TagHelper -->
<f:Grid ... PagerAutoSimpleMode="true"> ... </f:Grid>
```

### 全局（一次开启，所有 Grid 生效）

```javascript
// F.js —— 初始化时
F.init({ gridPagerAutoSimpleMode: true });
```
```aspx
<%-- Pro —— 页面级 PageManager --%>
<f:PageManager ID="PageManager1" runat="server" GridPagerAutoSimpleMode="true" />
```
```csharp
// Core 三模式通用 —— 页面顶部流式（F = Html.F()）
@{ F.PageManager.GridPagerAutoSimpleMode(true); }
```

---

## 3. 自定义分页栏项（在分页栏放自己的控件）

```javascript
// F.js —— pageItems 放工具栏项
pageItems: [
    { type: 'ToolbarText', text: '每页记录数：' },
    { type: 'DropDownList', width: 100, data: [['5','5'],['10','10'],['20','20']],
      listeners: { change: function (e, v) { F.ui.Grid1.setPageSize(parseInt(v)); } } }
]
```
```aspx
<%-- Pro —— <PageItems> --%>
<f:Grid ID="Grid1" runat="server" AllowPaging="true" PageSize="5">
    <Columns> ... </Columns>
    <PageItems>
        <f:ToolbarText runat="server" Text="每页记录数：" />
        <f:DropDownList runat="server" ID="ddlPageSize" Width="100px" AutoPostBack="true"
            OnSelectedIndexChanged="ddlPageSize_SelectedIndexChanged">
            <f:ListItem Text="5" Value="5" /><f:ListItem Text="10" Value="10" /><f:ListItem Text="20" Value="20" />
        </f:DropDownList>
    </PageItems>
</f:Grid>
```
```csharp
// Pro 后台：改 PageSize
protected void ddlPageSize_SelectedIndexChanged(object sender, EventArgs e) {
    Grid1.PageSize = Convert.ToInt32(ddlPageSize.SelectedValue);
}
```

> Core 也有 `PageItems`（Fluent `.PageItems(...)` / TagHelper `<PageItems>`），用法同上。

---

## 4. PageManager / 配置文件全局项

“全局设一次、页面可覆盖”的 Grid 级默认。**放在两处之一：**

```aspx
<%-- Pro —— Web.config 的 <FineUIPro> 段（全局默认）--%>
<FineUIPro GridPagingToolbarVisible="true" GridPagerAlignRight="true"
           GridPagingType="Arrow" GridPagerAutoSimpleMode="true" ... />
```
```json
// Core —— appsettings.json 的 "FineUI" 段（三模式通用，Startup 里 AddFineUI(Configuration) 读取）
"FineUI": {
    "GridPagingToolbarVisible": true,
    "GridPagerAlignRight": true,
    "GridPagingType": "Arrow",
    "GridPagerAutoSimpleMode": true
}
```

常用 Grid 全局项：`GridPagingToolbarVisible`（分页栏可见）、`GridPagerAlignRight`（分页栏右对齐）、`GridPagingType`（分页样式，如 `Arrow`）、`GridPagerAutoSimpleMode`（窄屏简洁分页）。页面单控件可用同名实例属性覆盖。

---

## 关键约束

1. **没有“全局每页条数”配置**：`PageSize` **只能逐 Grid 设**。Core **不存在** `GridPageSize` 全局项，Pro 亦无 `GridPageSize` 全局键——不要凭空写。
2. **`pagerAutoSimpleMode` 全局键名**：F.js `gridPagerAutoSimpleMode`（`F.init`）；Pro/Core 全局 `GridPagerAutoSimpleMode`；实例属性 F.js `pagerAutoSimpleMode` / C# `PagerAutoSimpleMode`（四段命名规律）。
3. **全局项两处入口**：Pro = `Web.config` 的 `<FineUIPro>` 或页面 `<f:PageManager>`；Core = `appsettings.json` 的 `FineUI` 段或页面 `F.PageManager.GridXxx(...)`。
4. **分页行号**：行号列跨页连续编号用 `EnablePagingNumber="true"`（Core）/ F.js `columnType:'rownumberfield'` + `pagingNumber:true`。

## See also

- [data-loading.md](data-loading.md)：分页模式（内存 / 数据库）与翻页事件
