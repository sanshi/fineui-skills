# 图表

v16.2.0 新增基于 Apache ECharts 的公开 `Chart` 控件，适用于 Pro、Core 三种模式、Java 及纯 JS。服务端图表继承 `BoxComponent`，F.js 对应 `Component`；`ChartSeries` 和 `ChartValueAxis` 是图表子配置，不能当作独立页面控件创建。

## 按任务选择配置

| 需求 | 公开入口 |
| --- | --- |
| 柱图、折线、面积、饼图及组合 | 系列 `Type` 为 `Bar/Line/Area/Pie`；C# 枚举为 `ChartSeriesType` |
| 横向条形图 | 图表 `Horizontal` |
| 折线平滑及标记点 | 系列 `Smooth`、`ShowSymbol` |
| 堆叠与渐变 | 系列 `Stack`、`AreaGradient`、`AreaOpacity` |
| 多数值轴 | `YAxis` 子项及系列 `YAxisIndex`，下标从 0 开始 |
| 环形、扇区间距及圆角 | `PieRadius`、`PiePadAngle`、`PieBorderRadius` |
| 业务配色 | `Palette`；默认首色跟随当前 FineUI 主题主色 |
| 图例与提示 | `ShowLegend`、`LegendType`、`LegendPosition`、`TooltipFormatter` |

图表和系列的显示选项在创建时确定。后台筛选、钻取及联动通过重绑数据刷新既有图表；不要因存在 setter 就假定系列集合或配色支持回发修改。未配置的细节用相应版本的公开 API 和示例核对。

## 数据字段与稳定业务键

`DataCategoryField` 指定分类文字字段，`DataIDField` 指定数据点的业务键，系列 `DataValueField` 指定数值字段。各系列的 `Key` 应互不重复。需要点击回发时，数据点业务键必须稳定且非空，例如 `east` 或 `search`；名称可重复，数组下标不能代替业务标识。

```html
<!-- Core RazorForms；Pro 增加 runat="server"。 -->
<f:Chart ID="ChartChannelDetails" Height="320"
    DataCategoryField="Category" DataIDField="Id"
    OnPointClick="ChartChannelDetails_PointClick">
    <Series>
        <f:ChartSeries Key="amount" Type="Pie" Name="访问量" DataValueField="Amount" />
    </Series>
</f:Chart>
```

```csharp
// Core RazorForms 或 Pro，rows 为应用提供的记录集合。
ChartChannelDetails.DataSource = rows;
ChartChannelDetails.DataBind();
```

```html
<f:chart id="ChartChannelDetails" height="320" data-category-field="Category"
    data-id-field="Id" on-point-click="ChartChannelDetails_PointClick">
    <f:chart-series key="amount" type="Pie" name="访问量" data-value-field="Amount" />
</f:chart>
```

Java 使用 `setDataSource(rows)` 后调用 `dataBind()`；方法名是 `setDataIdField`，模板属性是 `data-id-field`。Core MVC 用 `Html.F().Chart()` 创建，例如下面的柱图；RazorPages 标签也使用相同的字段和系列属性。

```csharp
F.Chart().ID("ChartSales").Height(320).DataCategoryField("Category")
    .Series("amount", ChartSeriesType.Bar, "销售额", "Amount")
    .DataSource(Model)
```

## 点击与后台联动

Pro/Core RazorForms 事件参数类型为 `ChartPointEventArgs`，读取 `e.SeriesKey` 与 `e.PointID`；Java 同名参数读取 `e.getSeriesKey()` 与 `e.getPointID()`。后台必须按业务键校验记录及访问权限后，再查询明细或重新绑定图表。

Core MVC/RazorPages 用各自的事件地址和事件参数入口接线，不使用 RazorForms 的后台控件恢复方式。F.js 的客户端 `pointclick` 签名为 `function (seriesKey, pointId)`，首参是系列键；不要给这个事件额外添加一个事件对象参数。

## 资源与容器

主包提供 `Chart` 声明和加载器，`FineUI.Chart.js`、`FineUI.Chart.css` 独立交付。服务端页面按需引入；纯 JS 可以提前引用，也可以通过 `chartResources: { script: 脚本地址, css: 样式地址 }` 配置地址。不要把网页使用的完整 ECharts 任意 option 当作 Chart 的公开配置直接传入。

图表跟随容器尺寸调整。隐藏页签首次显示后才初始化，页面删除控件时释放图表实例；容器必须有可用的高度。后台传入空数据源并绑定会清空图表。Java 数据源为 `Iterable<?>`，Pro/Core 支持数据表或可枚举对象。

[渠道明细](https://fineui.com/core/demo/#/Chart/Pie/ChannelDetails) · [区域钻取](https://fineui.com/core/demo/#/Chart/Bar/DrillDown) · [Java 图表联动](https://fineui.com/java/demo/#/chart/pie/channel-details) · [Pro 图表联动](https://fineui.com/pro/demo/#/chart/pie/channel_details.aspx)
