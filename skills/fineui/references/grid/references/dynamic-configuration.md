# 回发时动态修改普通表格（v16.1.1）

适用于 **FineUI.Core RazorForms 与 FineUI.Java v16.1.1 起的普通表格**。后台事件直接修改表格属性、已有列或列集合，页面随之更新，后续回发可以继续读取修改后的配置。不把这项能力当作 v16.1.0 已有的完整支持。

Core MVC/RazorPages 继续使用 `UIHelper` 等原有接口；Pro 按自身生命周期与接口处理，不能照搬下面的集合替换写法。纯 F.js 的 `grid.configColumns(...)` 是客户端方法，不等于后台控件属性赋值。

## 选择写法

| 需求 | Core RazorForms | Java |
| --- | --- | --- |
| 选择方式、外观 | `EnableCheckBoxSelect`、`EnableMultiSelect`、`EnableAlternateRowColor` | 对应的 `setEnable...` 方法 |
| 修改已有列 | `FindColumn("Name").HeaderText = "客户名称"` | `findColumn("Name").setHeaderText("客户名称")` |
| 增删列 | `Columns.Add(...)`、`Columns.Remove(...)`、`Columns.Clear()` | `addColumn(...)`、`getColumns().remove(...)`、`getColumns().clear()` |
| 替换整组列 | `Columns = new GridColumnCollection { ... }` | `setColumns(List.of(...))` |
| 创建多表头 | `GroupField.Columns.Add(...)` | `GroupField.addColumn(...)` |
| 排序、分页 | `AllowSorting`、`AllowPaging`、`PageSize`、`PageIndex`、`PageSizeOptions` | 对应的 `set...` 方法 |

按 `ColumnID` 查列；`DataField` 是数据字段名，不代替列标识。取得列后可修改标题、宽度、对齐、显隐和显示格式。替换时新建列对象，不复用其他表格已经拥有的列；清空使用空集合或 `Clear/clear`，不要传 `null`。

## 后台事件示例

下面的方法放在已有 `Grid1` 的页面类中，由按钮的 `OnClick` / `on-click` 绑定；首次数据仍在 `Page_Load` 的首次加载分支绑定。示例假定已加载数据含 `Name` 和 `EntranceYear`，所以切换列不重新查询数据。

Core RazorForms 使用 `FineUI` 命名空间：

```csharp
protected void ChangeSelection_Click(object sender, EventArgs e)
{
    Grid1.EnableCheckBoxSelect = true;
    Grid1.EnableMultiSelect = true;
    Grid1.EnableAlternateRowColor = false;
}

protected void ReplaceColumns_Click(object sender, EventArgs e)
{
    Grid1.Columns = new GridColumnCollection
    {
        new RenderField
        {
            ColumnID = "Name",
            DataField = "Name",
            HeaderText = "姓名",
            Width = 180
        },
        new RenderField
        {
            ColumnID = "EntranceYear",
            DataField = "EntranceYear",
            HeaderText = "入学年份",
            FieldType = FieldType.Int
        }
    };
}
```

Java 使用 `com.fineui.java.core.controls` 的列类型、`com.fineui.java.core.enums.FieldType`、`com.fineui.java.core.EventArgs` 和 `java.util.List`：

```java
public void ChangeSelection_Click(Object sender, EventArgs e) {
    Grid1.setEnableCheckBoxSelect(true);
    Grid1.setEnableMultiSelect(true);
    Grid1.setEnableAlternateRowColor(false);
}

public void ReplaceColumns_Click(Object sender, EventArgs e) {
    RenderField name = new RenderField();
    name.setColumnId("Name");
    name.setDataField("Name");
    name.setHeaderText("姓名");
    name.setWidth(180);

    RenderField year = new RenderField();
    year.setColumnId("EntranceYear");
    year.setDataField("EntranceYear");
    year.setHeaderText("入学年份");
    year.setFieldType(FieldType.Int);

    Grid1.setColumns(List.of(name, year));
}
```

## 排序、分页与数据绑定

| 修改 | 数据处理 |
| --- | --- |
| 标题、宽度、对齐、显隐、选择和行外观 | 通常不重新绑定 |
| 使用已有字段增删、重排或分组表头 | 字段及类型不变时复用已有数据 |
| 新字段、字段类型或行 ID 字段改变 | 提供与新配置匹配的数据并绑定 |
| 内存排序、内存分页 | 先加载完整数据，后续由客户端排序、切页 |
| 数据库排序、数据库分页 | 按当前条件查询并绑定；分页同时设置总数，关闭分页时提供要展示的完整数据 |

`PageSizeOptions` 在这两种模式中可于回发修改，配合 `ShowPageSizeSelector` 使用；`PagingToolbarVisible` 控制分页栏显隐。总数由业务明确设置时会保留，即使当前没有开启分页。列锁定和合计也可动态修改，合计数据按业务重新计算并设置。

## 支持边界与升级核对

- 暂不支持重建单元格编辑器、表头过滤器及其所属弹层和字段，也不支持替换重绘区域内的业务控件、业务 `Bars/Tools/PageItems` 集合。已有业务工具栏实例保留，框架分页部件按需更新。
- 树表格、行分组、大数据、卡片等特殊模式的通用切换不在范围内；普通 `GroupField` 多表头可以使用。不要将任意 renderer 中创建的业务控件当作可自动重建的普通单元格。
- Core RazorForms 普通表格统一推荐直接修改属性和 `Columns`，不再推荐 `ConfigColumns`；Java 推荐对应 setter。旧接口保留兼容但不用于新示例；复杂编辑表格单独处理，不要为使用新写法而删除编辑器、过滤器或直接放行同名控件 ID。
- 验证修改后的实际显示，再触发下一次回发核对后台读取值；分页、排序需点击真实表头和分页按钮。换列还要核对每行数据是否与列对应，不能只看响应存在或 HTTP 200。

## 官方指南与示例

- [Core RazorForms：回发时动态修改表格](https://fineui.com/docs/#/FineUICore_RazorForms/1510_grid_dynamic_configuration.md)
- [Java：回发时动态修改表格](https://fineui.com/docs/#/FineUIJava/151_grid_dynamic_configuration.md)
- 示例分组“动态修改表格”提供七组成对页面：显示与选择、增删列与多表头、内存/数据库排序、内存/数据库分页、列锁定与合计。以项目所用版本的示例为准。
