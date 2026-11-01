# 清空控件选择（v16.1.1）

Grid、DropDownList、DropDownBox、DataList、Tree、CheckBoxList、RadioButtonList 统一使用 `ClearSelection()`（Pro / Core）或 `clearSelection()`（Java / F.js）。单选和多选使用同一个方法，不用先判断控件模式，也不必用空字符串表示未选择。

## 清空范围

| 控件 | 清空内容 | 保留内容 |
| --- | --- | --- |
| Grid | 全部行选择和单元格选择，含跨页保存、不可选行的旧选择 | 行数据、总数、编辑值和业务复选框字段 |
| DropDownList、DropDownBox | 选中值、显示文字、已选标签及对应弹出选择状态 | 选项和弹出控件的数据 |
| DataList | 全部选中项，含不可选项的旧选择 | 列表数据 |
| Tree | 全部节点高亮选择，含不可选节点的旧选择 | 节点、展开状态、复选框勾选和半选状态 |
| CheckBoxList | 全部选项勾选，含已禁用的旧选项 | 选项数据 |
| RadioButtonList | 当前单选，恢复未选择状态 | 选项数据 |

这些方法不触发选择或勾选变化事件，也不自动回发；字段校验沿用赋值规则。明确清空后不恢复模板默认选择，也不会因 `AutoSelectFirstItem` 自动选回第一项。

## 官方示例的组织

六套官方示例都在“表单 → 杂项”提供“统一清空选择”和“下拉列表赋值顺序”两页。前者将七类控件放入独立区域，以 FineUI 原生 Block 布局实现宽屏两列、窄屏一列，树取消勾选按钮位于树区域；后者独立展示可编辑下拉列表、两种赋值顺序和读取结果。说明放在控件下方，不混用两组操作的结果区，也不手写响应式 CSS。

示例列表项使用非空业务值，需要未选择提示时使用空状态、EmptyText 或 AutoSelectFirstItem=false，不加入空字符串值的占位项。底层对空字符串值的兼容只在测试中验证。

## 真实后台控件

Pro、Core RazorForms 和 Java 的真实控件在调用后立即同步后台选中状态；同一事件继续读取时，选中数组为空，单选值表示未选择。服务器响应随后更新浏览器，下一轮回发仍为空。

```csharp
// Pro / Core RazorForms：清空后本轮读取也已无选择。
Grid1.ClearSelection();
DropDownList1.ClearSelection();
DropDownBox1.ClearSelection();
DataList1.ClearSelection();
Tree1.ClearSelection();
CheckBoxList1.ClearSelection();
RadioButtonList1.ClearSelection();

string[] selectedIds = Grid1.SelectedRowIDArray;
```

```java
// FineUI.Java：处理器直接操作本页控件字段。
Grid1.clearSelection();
DropDownList1.clearSelection();
DropDownBox1.clearSelection();
DataList1.clearSelection();
Tree1.clearSelection();
CheckBoxList1.clearSelection();
RadioButtonList1.clearSelection();

String[] selectedIds = Grid1.getSelectedRowIDArray();
```

清空后仍可在同一轮重新选择，例如 `Grid1.ClearSelection()` 后设置 `SelectedRowIDArray`，最终保留新选择；Java 使用 `setSelectedRowIDArray(...)`。不要在清空方法后再追加一段无条件清空脚本，否则会覆盖后面的业务赋值。

## Core MVC / RazorPages 与客户端

MVC / RazorPages 在回发处理器中使用 `UIHelper`，它只更新浏览器，不回写本次已经绑定的请求参数：

```csharp
UIHelper.Grid("Grid1").ClearSelection();
UIHelper.DropDownList("DropDownList1").ClearSelection();
UIHelper.DropDownBox("DropDownBox1").ClearSelection();
UIHelper.DataList("DataList1").ClearSelection();
UIHelper.Tree("Tree1").ClearSelection();
UIHelper.CheckBoxList("CheckBoxList1").ClearSelection();
UIHelper.RadioButtonList("RadioButtonList1").ClearSelection();
return UIHelper.Result();
```

F.js、Core 和 Java 的页面脚本使用 `F.ui.Grid1.clearSelection()`；其它六类控件使用相同方法。Pro 脚本应先声明服务器渲染的客户端 ID，再调用 `F(Grid1ClientID).clearSelection()`，不能假定声明 ID 等于浏览器 ID。

## 三种不同操作

- **清空选择**：使用 `ClearSelection()` / `clearSelection()`，保留数据。
- **清空数据**：使用控件已有的 `ClearData()` / `clearData()`；不要为只清选择而重新绑定空数据源。
- **恢复初始值**：使用字段的 `Reset()` / `reset()`，初始值本来已选中时会重新选中。

Grid 的 `DeselectAllRows()` / `deselectAllRows()` 仅取消当前页可选行，保留其它分页和单元格选择，事件行为也不同，不能代替彻底清空。

Tree 的复选框勾选独立处理：Pro / Core 使用 `UncheckAllNodes()`，Java 使用 `uncheckAllNodes()`。它们取消后台当前节点集合的勾选，再加载当前节点；不会查询尚未加载的子节点。`EnableCheckBox=false` 的节点保留自身勾选，禁用复选框仍会取消勾选，半选结果按原级联规则重新计算。只清节点高亮时，直接用 `ClearSelection()`。

客户端取消当前勾选使用真实公开方法 `tree.uncheckNodes(tree.getCheckedNodes())`，不能把后台的 `UncheckAllNodes` / `uncheckAllNodes` 当作 JavaScript 方法名。MVC / RazorPages 需要从后台发起该动作时，可用具名页面函数与结构化调用，保持严格 CSP。

## 旧项目迁移

Grid 的复数 `ClearSelections()` / `clearSelections()` 及相关复数脚本入口已废弃，新代码使用单数。Core、Java 的旧复数方法仍只发送客户端动作，保留本次请求已采集的后台选择；迁移到单数后，同一事件的后台读取立即为空。需要处理原选择时，先读取并保存所需 ID，再清空。

Pro 原有单数 `ClearSelection()` 现在也同步清空后台选择；依赖调用后还能读到旧值的代码同样应把读取提前。客户端与服务端资源应配套更新；协议级测试使用单数内部命令 `grid.clearSelection`。

DropDownList 的 `SelectedValue=null` / `setSelectedValue(null)` 仍可表示无选择，空字符串仍可能是实际选项值；清空整套选择和显示文字推荐直接调用统一方法，不用两次赋值拼接清空逻辑。
