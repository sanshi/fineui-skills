# HBox / VBox 弹性盒子布局

`HBox` 横向排列子控件，`VBox` 纵向排列子控件；`BoxFlex` 按比例分配剩余空间，适合左侧固定、右侧自适应，以及上方查询表单、下方表格铺满等布局。

## 尺寸、对齐与间隔

- 子项尺寸：HBox 用 `Width` 固定宽度，VBox 用 `Height` 固定高度；弹性子项用 `BoxFlex`。
- 交叉方向对齐：容器的 `BoxConfigAlign`，默认 `Stretch`，也支持 `Start`、`Center`、`End`、`StretchMax`、`StretchMaxCenter`、`StretchMaxEnd`。
- 排列方向对齐：容器的 `BoxConfigPosition`，支持 `Start`（默认）、`Center`、`End`。
- **统一子项间隔优先使用 `BoxConfigSpace`**，单位为像素；HBox 设置横向间隔，VBox 设置纵向间隔，最后一个可见子项不追加间隔。
- 四周留白使用容器的 `BodyPadding`；单个子项需要特殊边距时才使用它自身的 `Margin`、`MarginRight`、`MarginBottom` 等属性。

F.js 的布局参数写在 `layout` 对象中，弹性值使用 `flex`，不要生成顶层 `boxConfigSpace`、`boxConfigAlign`、`boxFlex` 等不存在的客户端配置。

## HBox：左侧固定、右侧自适应

下面的两栏之间留 10px，右侧末项不再留间隔。

```javascript
// F.js：布局参数放在 layout 对象内。
{
    type: 'Panel',
    layout: { type: 'hbox', align: 'stretch', space: 10 },
    items: [
        { type: 'Panel', width: 200, title: '左侧固定' },
        { type: 'Panel', flex: 1, title: '右侧自适应' }
    ]
}
```

```aspx
<%-- FineUI.Pro；Core 标签写法相同，但不需要 runat="server"。 --%>
<f:Panel runat="server" Layout="HBox" BoxConfigSpace="10">
    <Items>
        <f:Panel runat="server" Width="200" Title="左侧固定" />
        <f:Panel runat="server" BoxFlex="1" Title="右侧自适应" />
    </Items>
</f:Panel>
```

```csharp
// FineUI.Core MVC。
@(F.Panel()
    .Layout(LayoutType.HBox)
    .BoxConfigSpace(10)
    .Items(
        F.Panel().Width(200).Title("左侧固定"),
        F.Panel().BoxFlex(1).Title("右侧自适应")
    ))
```

```html
<!-- FineUI.Java：属性名使用小写连字符，布局值保持 PascalCase。 -->
<f:panel layout="HBox" box-config-space="10">
    <f:items>
        <f:panel width="200" title="左侧固定"></f:panel>
        <f:panel box-flex="1" title="右侧自适应"></f:panel>
    </f:items>
</f:panel>
```

两栏等分时，两个子项都设置 `BoxFlex="1"`（Java 为 `box-flex="1"`，F.js 为 `flex: 1`）。

## VBox：上方查询表单、下方表格

表单的最后一行字段不再留底部边距，由父容器统一安排表单与表格之间的 10px 间隔。

```html
<!-- FineUI.Core RazorForms / RazorPages。 -->
<f:Panel ID="Panel1" IsViewPort="true" Layout="VBox" BoxConfigSpace="10" BodyPadding="10">
    <Items>
        <f:Form ID="SearchForm" RemoveLastFieldsMargin="true" BodyPadding="10">
            <Rows>
                <f:FormRow>
                    <Items>
                        <f:TextBox ID="Keyword" Label="关键字" />
                    </Items>
                </f:FormRow>
            </Rows>
        </f:Form>
        <f:Grid ID="Grid1" BoxFlex="1" />
    </Items>
</f:Panel>
```

Pro 额外声明 `runat="server"`；MVC 使用 `.Layout(LayoutType.VBox).BoxConfigSpace(10)`；Java 使用 `layout="VBox" box-config-space="10"`，表单属性为 `remove-last-fields-margin="true"`。

F.js 对应的父容器布局为 `layout: { type: 'vbox', space: 10 }`，表单配置为 `removeLastFieldsMargin: true`，表格配置为 `flex: 1`。弹性子项如果是包裹表格的面板，使用 `Layout="Fit"` 并只放一个表格；表格直接作为 VBox 子项时无需另套 Fit 面板。

## 与普通子项、末项边距的等效关系

旧属性仍支持，但只在讲解对照、兼容旧代码或分别控制各方向边距时使用。普通业务示例不要同时设置两套配置。

| 方向 | 推荐写法 | 等效边距写法 |
| --- | --- | --- |
| HBox | `BoxConfigSpace="10"` | `BoxConfigChildMargin="0 10 0 0" BoxConfigLastChildMargin="0"` |
| VBox | `BoxConfigSpace="10"` | `BoxConfigChildMargin="0 0 10 0" BoxConfigLastChildMargin="0"` |

F.js 对应 `layout.space: 10`，等效配置为 HBox 的 `layout.childMargin: '0 10 0 0'` 或 VBox 的 `layout.childMargin: '0 0 10 0'`，再配合 `layout.lastChildMargin: '0'`。Java 对照属性为 `box-config-child-margin`、`box-config-last-child-margin`。

边界需要分清：

- `BoxConfigSpace` 会覆盖这两个旧的布局边距属性，子项自身的 `Margin` 及分方向边距仍优先。
- 仅设置 `BoxConfigChildMargin` 而没有清零末项，会给末项留下尾部空隙，与 `BoxConfigSpace` 的语义不同。迁移时先判断这段空隙是否有用途；不要为恢复旧效果机械地补 `MarginBottom` / `MarginRight`。只有一个子项时没有相邻间隔，四周已有 `BodyPadding` 就无需再给末项补一份边距。
- 原来仅用于清零末项布局边距的 `Margin="0"` 可以删除；有实际业务用途的子项边距应保留。
- 不设置 `BoxConfigSpace` 时沿用已有边距；显式设置 `0` 表示清除布局边距。`0` 不应与“未设置”混淆。
- 隐藏子项不参与排列，末项指最后一个可见子项。

## 属性对照

| 概念 | F.js | C#（Pro / Core） | Java |
| --- | --- | --- | --- |
| 布局 | `layout: { type: 'hbox' }` | `Layout="HBox"` / `.Layout(LayoutType.HBox)` | `layout="HBox"` |
| 弹性尺寸 | `flex: 1` | `BoxFlex="1"` / `.BoxFlex(1)` | `box-flex="1"` |
| 交叉方向对齐 | `layout.align: 'stretch'` | `BoxConfigAlign="Stretch"` | `box-config-align="Stretch"` |
| 排列方向对齐 | `layout.position: 'start'` | `BoxConfigPosition="Start"` | `box-config-position="Start"` |
| 子项间隔 | `layout.space: 10` | `BoxConfigSpace="10"` / `.BoxConfigSpace(10)` | `box-config-space="10"` |

## 相关资料

- [layout-types.md](layout-types.md)：Fit / Region / Block / Column / Anchor。
- 官方示例“布局 / Box / HBox（子项间隔）”与“VBox（子项间隔）”：同页比较推荐写法和等效边距配置。
