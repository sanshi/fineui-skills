# DataList 选择、分组与导航

## 单选与多选

| 用途 | F.js | Pro / Core | FineUIJava |
|------|------|------------|------------|
| 允许选择 | `selectable: true` | `EnableSelect="true"` | `enable-select="true"` |
| 允许多选 | `multiSelect: true` | `EnableMultiSelect="true"` | `enable-multi-select="true"` |
| 初始单选 | `selectedItems: ['cn']` | `SelectedValue="cn"` | `selected-value="cn"` |
| 初始多选 | `selectedItems: ['cn','us']` | Pro 后台赋 `string[]`；Core 标签传 `string[]` | `selected-value-array="cn,us"` |
| 读取单选 | `getSelectedItem()` | `SelectedValue` | `getSelectedValue()` |
| 读取多选 | `getSelectedItems()` | `SelectedValueArray` | `getSelectedValueArray()` |

```html
<!-- FineUICore RazorForms -->
<f:DataList ID="DataList1" EnableSelect="true" EnableMultiSelect="true"
    SelectedValueArray="@(new string[] { "cn", "us" })"
    DataValueField="Id"></f:DataList>
```

```html
<!-- FineUIJava -->
<f:data-list id="DataList1" enable-select="true" enable-multi-select="true"
    selected-value-array="cn,us" data-value-field="Id"></f:data-list>
```

```csharp
// Pro / Core RazorForms：在按钮回发中读取
string[] selectedValues = DataList1.SelectedValueArray;
ShowNotify("选中项：" + String.Join(", ", selectedValues));
```

```java
public void btnSubmit_Click(Object sender, EventArgs e) {
    showNotify("选中项：" + String.join(", ", DataList1.getSelectedValueArray()));
}
```

Core MVC / RazorPages 没有有状态页面控件字段时，将客户端 `F.ui.DataList1.getSelectedItems()` 作为明确的回发参数提交给 action/handler。

## 多选时让普通点击累积选择

`KeepCurrentSelection` / `keep-current-selection` 只影响用户点击列表项时的多选行为。开启后，普通点击会在现有选择上切换当前项，无需按住 Ctrl；关闭时，普通点击会用当前项替换已有选择，按住 Ctrl 仍可逐项增减。

它不能代替首次设置 `SelectedValueArray`，也不负责在重新绑定数据后恢复选择。需要重新绑定后保留哪些值时，应由业务代码保存仍有效的 Value，并在新数据上重新设置选中值。

## 分组

开启 `EnableGroup`，并为每项提供 Group；数据绑定时使用 `DataGroupField`：

```aspx
<f:DataList ID="DataList1" runat="server" EnableGroup="true" DataGroupField="Region" />
```

```html
<f:data-list id="DataList1" enable-group="true" data-group-field="Region"></f:data-list>
```

F.js 数据需要在 `fields` 中包含 `group`。分组只改变列表的展示结构，不等同于选择分组，也不会自动生成折叠面板。

## 导航项

列表项设置 `NavigateUrl`、`Target` 和 `ShowArrow` 后可以作为移动端导航入口：

```html
<f:DataListItem Text="产品文档" NavigateUrl="~/docs/" Target="_self" ShowArrow="true" />
```

```html
<f:data-list-item text="产品文档" navigate-url="/docs/" target="_self" show-arrow="true"></f:data-list-item>
```

有真实地址时使用 NavigateUrl，不要把普通选择项伪装成链接。需要点击后切换同页 Panel 时，可以监听列表项链接的客户端点击，再调用 `F.slideLeft` / `F.slideRight`；这属于页面交互逻辑，不是 DataList 的服务端选择事件。
