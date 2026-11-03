# FineUI DataList 数据列表技能

从 **16.1.1** 起，FineUI.Pro、FineUI.Core 三种开发模式和 FineUI.Java 的社区版均可使用完整 DataList；旧社区版需要先升级配套 DLL/JAR 与客户端资源。普通示例位于“导航 → 数据列表（DataList）”。全部移动支持及原移动示例仍属于企业版，使用 DataList 不会自动开放移动布局、工具栏或动画。纯 JavaScript 产品不区分社区版和企业版。

## 何时使用（When to Use）

- 显示普通页面中的纵向文字、图文或分组数据列表；企业版还可组合移动布局
- 用静态 DataListItem 或服务端数据生成列表项
- 实现单选、多选、分组、导航或“加载更多”
- 读取当前选中值，或让多选列表在普通点击时继续累积选择

DataList 与 Grid 都显示集合数据，但用途不同：需要列、排序、分页、编辑或表头时用 Grid；需要简洁的纵向图文项、移动端导航或卡片式列表时用 DataList。

## 开始前（Before You Start）

1. 先从项目结构判断写法：F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / FineUI.Java，不混用属性名称。
2. 明确列表项是普通文本还是可信 HTML。普通文本默认编码；图文模板只有在内容经过编码或来自可信常量时才能使用 RawHtml。
3. 需要选择时同时设置 `EnableSelect`；多选再设置 `EnableMultiSelect`，并为每项提供稳定的 Value。

## 各写法速览（静态列表）

```javascript
// F.js
F.create({
    type: 'DataList', id: 'DataList1', renderTo: '#wrap',
    fields: ['text', 'value'],
    data: [['中国', 'cn'], ['美国', 'us']]
});
```

```aspx
<%-- FineUI.Pro --%>
<f:DataList ID="DataList1" runat="server">
    <f:DataListItem Text="中国" Value="cn" />
    <f:DataListItem Text="美国" Value="us" />
</f:DataList>
```

```csharp
// FineUI.Core MVC
@(F.DataList().ID("DataList1").Items(
    F.DataListItem().Text("中国").Value("cn"),
    F.DataListItem().Text("美国").Value("us")))
```

```html
<!-- FineUI.Core RazorForms / RazorPages -->
<f:DataList ID="DataList1">
    <f:DataListItem Text="中国" Value="cn" />
    <f:DataListItem Text="美国" Value="us" />
</f:DataList>
```

```html
<!-- FineUI.Java -->
<f:data-list id="DataList1">
    <f:data-list-item text="中国" value="cn"></f:data-list-item>
    <f:data-list-item text="美国" value="us"></f:data-list-item>
</f:data-list>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/items-data.md](references/items-data.md) | 静态项、服务端绑定、普通文本与 RawHtml 图文项、加载更多 |
| [references/selection-navigation.md](references/selection-navigation.md) | 单选/多选、读取选中值、普通点击累积选择、分组与导航项 |

## 相关分类（Related Skills）

- `基础`：技术栈判断、页面骨架、RawHtml 安全模型
- `容器`：普通面板、布局与工具栏；移动页面支持仍需企业版
- `表格`：需要表头、列、排序、分页和编辑的数据展示

## 约束与规则（Constraints & Rules）

1. **属性名随写法变化**：F.js 用 `selectable` / `multiSelect` / `selectedItems`；C# 用 `EnableSelect` / `EnableMultiSelect` / `SelectedValueArray`；Java 标签用 `enable-select` / `enable-multi-select` / `selected-value-array`。
2. **选择必须有稳定 Value**：可选项设置 `Value`；服务端数据绑定设置 `DataValueField`。不要用显示文本充当业务标识。
3. **普通文本默认编码**：声明项使用 `Text`；图文 HTML 使用 `TextRawHtml` / `_TextRawHtml` / `RawHtml`，其中的动态数据必须先编码。
4. **重新绑定与追加不同**：替换整表时清空并重新绑定；“加载更多”使用 `AppendData` / `appendData`，不要每次重建已有列表。
5. **DataList 不是 Grid**：不要生成 Columns、分页器、单元格编辑或 Grid 的选择 API。
6. **只使用公开控件与 API**：不从源码类名猜测用户控件；把握不准时核对三栈源码和官方示例。
7. **KeepCurrentSelection 只影响点击选择**：多选时开启它，普通点击会在现有选择上切换当前项，无需按 Ctrl；它不负责在重新绑定数据后恢复选择。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- FineUI.Java 的标签属性按 Core 语义转 kebab-case；运行时客户端 API 与 F.js 一致。
