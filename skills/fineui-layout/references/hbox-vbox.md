# HBox / VBox 弹性盒子布局（重点）

`HBox`（水平）/ `VBox`（垂直）把子控件按方向排列，用 `BoxFlex` 弹性分配剩余空间。是做"左固定+右自适应""上工具栏+下表格铺满"这类布局的首选（比 Column/Anchor 更灵活，官方也推荐用它替代 Column）。

## 核心概念

- **子项尺寸**：`BoxFlex="n"` 按比例分配剩余空间（HBox 分宽、VBox 分高）；不设 `BoxFlex` 则用固定 `Width`（HBox）/ `Height`（VBox）。
- **交叉轴对齐**（容器上，`BoxConfigAlign`）：`Stretch`（拉伸铺满，默认）/ `Center` / `End` / `Start` / `StretchMax`。
- **主轴排列**（容器上，`BoxConfigPosition`）：`Start` / `Center` / `End`。
- **子项间距**（容器上，`BoxConfigChildMargin`）：如 `"0 5 0 0"`（上 右 下 左）。

## HBox —— 水平盒子

例：左侧固定 200px，右侧弹性铺满。

```javascript
// F.js
{ type: 'Panel', layout: 'hbox', boxConfigAlign: 'stretch', boxConfigChildMargin: '0 5 0 0', items: [
    { type: 'Panel', width: 200, title: '左固定' },
    { type: 'Panel', boxFlex: 1, title: '右弹性' }
] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel runat="server" Layout="HBox" BoxConfigAlign="Stretch" BoxConfigChildMargin="0 5 0 0">
    <Items>
        <f:Panel Width="200px" Title="左固定" runat="server" />
        <f:Panel BoxFlex="1" Title="右弹性" runat="server" />
    </Items>
</f:Panel>
```
```csharp
// Core-MVC（Fluent）
@(F.Panel().Layout(LayoutType.HBox).BoxConfigAlign(BoxLayoutAlign.Stretch).BoxConfigChildMargin("0 5 0 0")
    .Items(
        F.Panel().Width(200).Title("左固定"),
        F.Panel().BoxFlex(1).Title("右弹性")
    ))
```
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:panel layout="HBox" box-config-align="Stretch" box-config-child-margin="0 5 0 0">
    <f:items>
        <f:panel width="200" title="左固定"></f:panel>
        <f:panel box-flex="1" title="右弹性"></f:panel>
    </f:items>
</f:panel>
```

两栏等分：两个子项都 `BoxFlex="1"`（Java `box-flex="1"`）。

## VBox —— 垂直盒子

例：上方工具栏固定高，下方表格铺满剩余高度（常用于列表页）。

```javascript
// F.js —— 下方 boxFlex:1 撑满，内部再 layout:'fit' 放 Grid
{ type: 'Panel', layout: 'vbox', boxConfigAlign: 'stretch', items: [
    { type: 'Panel', height: 40, title: '工具区' },
    { type: 'Panel', boxFlex: 1, layout: 'fit', items: [{ type: 'Grid', ... }] }
] }
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Panel runat="server" Layout="VBox" BoxConfigAlign="Stretch">
    <Items>
        <f:Panel Height="40px" Title="工具区" runat="server" />
        <f:Panel BoxFlex="1" Layout="Fit" runat="server"><Items><f:Grid runat="server"> ... </f:Grid></Items></f:Panel>
    </Items>
</f:Panel>
```
```csharp
// Core-MVC（Fluent）
@(F.Panel().Layout(LayoutType.VBox).BoxConfigAlign(BoxLayoutAlign.Stretch)
    .Items(
        F.Panel().Height(40).Title("工具区"),
        F.Panel().BoxFlex(1).Layout(LayoutType.Fit).Items(F.Grid().ID("Grid1"))
    ))
```
```html
<!-- FineUIJava（Thymeleaf 方言）：下方 box-flex="1" 撑满，内部 layout="Fit" 放 Grid -->
<f:panel layout="VBox" box-config-align="Stretch">
    <f:items>
        <f:panel height="40" title="工具区"></f:panel>
        <f:panel box-flex="1" layout="Fit"><f:items><f:grid id="Grid1"> ... </f:grid></f:items></f:panel>
    </f:items>
</f:panel>
```

## 属性对照

| 概念 | F.js | C#（Pro / Core） | Java（Thymeleaf 方言） |
|------|------|------------------|------------------------|
| 布局 | `layout: 'hbox'` / `'vbox'` | `Layout="HBox"` / `"VBox"` / `.Layout(LayoutType.HBox)` | `layout="HBox"` / `"VBox"` |
| 弹性尺寸 | `boxFlex: 1` | `BoxFlex="1"` / `.BoxFlex(1)` | `box-flex="1"` |
| 交叉轴对齐 | `boxConfigAlign: 'stretch'` | `BoxConfigAlign="Stretch"` / `.BoxConfigAlign(BoxLayoutAlign.Stretch)` | `box-config-align="Stretch"` |
| 主轴排列 | `boxConfigPosition: 'start'` | `BoxConfigPosition="Start"` / `.BoxConfigPosition(BoxLayoutPosition.Start)` | `box-config-position="Start"` |
| 子项间距 | `boxConfigChildMargin: '0 5 0 0'` | `BoxConfigChildMargin="0 5 0 0"` | `box-config-child-margin="0 5 0 0"` |
| 子项间隔 | `boxConfigSpace: 10` | `BoxConfigSpace="10"` | `box-config-space="10"` |

> Java 的 `box-config-align` 值与 C# 一致：`Stretch`（默认）/ `Center` / `End` / `Start` / `StretchMax` / `StretchMaxCenter` / `StretchMaxEnd`。

## 常见坑

- **HBox/VBox 未设容器高度时**（`BoxConfigAlign` 默认 `Stretch`）：未设高度的子项会被拉伸填充容器高度（v11 起的行为）。要固定就给子项 `Height`/`Width`，要弹性就用 `BoxFlex`。
- **"撑满剩余 + 内部放 Grid/Form"**：弹性子项加 `Layout="Fit"`，再放一个 Grid/Form，让它铺满该弹性区域。

## See also

- [layout-types.md](layout-types.md)：Fit / Region / Block / Column / Anchor
- `fineui-panel`：Panel 容器
