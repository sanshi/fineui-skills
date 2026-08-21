---
name: fineui-layout
description: >
  帮助开发者使用 FineUI 的布局系统——容器的 `Layout` 属性如何排布子控件：
  Fit（填充）/ Region（区域上下左右中）/ HBox·VBox（弹性盒子）/ Block（响应式栅格）/ Column / Anchor，
  以及视口自适应（IsViewPort / AutoSizePanelID）。覆盖 F.js、Pro、FineUICore（MVC/RazorForms/RazorPages）、
  FineUIJava（Spring Boot + Thymeleaf 方言标签，kebab-case，含 `<f:region-panel>` 便捷控件）。
  Trigger phrases（触发词）: "FineUI 布局", "Layout", "Region 布局", "区域布局", "RegionPosition",
  "HBox", "VBox", "BoxFlex", "弹性布局", "Fit 布局", "Block 响应式", "BlockMD", "栅格布局",
  "IsViewPort", "视口自适应", "Column 布局", "Anchor 布局", "FineUIJava", "Spring Boot", "Thymeleaf",
  "region-panel", "region-position", "box-flex", "is-view-port".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 布局技能（Layout System）

> 本技能讲**容器内部子控件如何排布**（`Layout` 属性）。容器组件本身（Panel/选项卡/手风琴）见 `fineui-panel`；整页骨架（ViewPort + Region + PageManager）见 `fineui-foundation` 的 page-scaffold.md。

## 何时使用（When to Use）

- 选并配置某种布局：填充、区域、弹性盒子、响应式栅格、分栏
- 让布局随浏览器窗口/屏幕尺寸自适应

## 布局类型速查（`Layout` 属性）

| 值 | 用途 | 详见 |
|----|------|------|
| `Container`（默认） | 子控件按自身高度依次排列 | — |
| `Fit` | **唯一**子控件铺满容器（放一个 Grid/Form 常用） | layout-types |
| `Region` | 区域布局：子面板 `RegionPosition` 定位 Top/Left/Center/Right/Bottom（Center 必填） | layout-types |
| **`HBox` / `VBox`** | **弹性盒子（重点）**，子用 `BoxFlex` 分配空间 | **hbox-vbox** |
| `Block` | 响应式栅格：子用 `Block/BlockSM/BlockMD/BlockLG`（1–12，行内和 12） | layout-types |
| `Column` | 列布局（官方推荐改用 HBox） | layout-types |
| `Anchor` | 锚点布局（表单默认），子用 `AnchorValue` | layout-types |
| `InlineBlock` / `Table` / `Absolute` / `Accordion` / `Card` | 行内块 / 表格 / 绝对 / 手风琴 / 卡片 | — |

## 速览：HBox 左固定 + 右自适应（最常用弹性布局）

```javascript
// ① F.js
{ type: 'Panel', layout: 'hbox', boxConfigAlign: 'stretch', items: [
    { type: 'Panel', width: 200, title: '左固定' }, { type: 'Panel', boxFlex: 1, title: '右弹性' }
] }
```
```aspx
<%-- ② Pro / ④ Core-TagHelper --%>
<f:Panel runat="server" Layout="HBox" BoxConfigAlign="Stretch">
    <Items><f:Panel Width="200px" Title="左固定" runat="server" /><f:Panel BoxFlex="1" Title="右弹性" runat="server" /></Items>
</f:Panel>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Panel().Layout(LayoutType.HBox).BoxConfigAlign(BoxLayoutAlign.Stretch)
    .Items(F.Panel().Width(200).Title("左固定"), F.Panel().BoxFlex(1).Title("右弹性")))
```
```html
<!-- ⑤ FineUIJava（Thymeleaf 方言：标签/属性全 kebab-case，布局值 layout="HBox" 保持 PascalCase）-->
<f:panel layout="HBox" box-config-align="Stretch">
    <f:items>
        <f:panel width="200" title="左固定"></f:panel>
        <f:panel box-flex="1" title="右弹性"></f:panel>
    </f:items>
</f:panel>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/hbox-vbox.md](references/hbox-vbox.md) | **HBox / VBox 弹性盒子（重点）**：BoxFlex、对齐、常见组合 |
| [references/layout-types.md](references/layout-types.md) | Fit / Region / Block / Column / Anchor + 视口自适应 |

## 相关技能（Related Skills）

- `fineui-panel`：Panel / TabStrip / Accordion 容器组件
- `fineui-foundation`：整页骨架（ViewPort + Region + PageManager）
- `fineui-grid` / `fineui-form`：放进 Fit / 弹性区域的内容

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js camelCase（`layout:'hbox'`/`boxFlex`）；C# PascalCase（`Layout="HBox"`/`BoxFlex`）；**Java 属性名 kebab-case（`layout="HBox"`/`box-flex`/`region-position`/`box-config-align`），但布局值/枚举值仍 PascalCase（`layout="HBox"`、`region-position="Center"`、`box-config-align="Stretch"`）**。
2. **Fit 只放一个子控件**：`Layout="Fit"`（Java `layout="Fit"`）时容器只应有一个子控件（它铺满）。
3. **Region 的 Center 必填**：区域布局必须有一个 `RegionPosition="Center"`（Java `region-position="Center"`）的面板占满剩余；左右用 `Width`、上下用 `Height`；`RegionSplit="true"`（Java `region-split="true"`，分隔条宽度 `region-split-width="3"`）加拖动条。**Java 另有便捷控件 `<f:region-panel>`/`<f:regions>`/`<f:region region-position=...>`**（等价于 `<f:panel layout="Region">` + 子 `<f:panel region-position=...>`）。
4. **视口自适应两种入口**：根面板 `IsViewPort="true"`（Java `is-view-port="true"`），或 Pro `<f:PageManager AutoSizePanelID="Panel1">`。
5. **Block 一行总和 12**：子项 `Block/BlockSM/BlockMD/BlockLG`（Java `block/block-sm/block-md/block-lg`）值 1–12，行内和为 12（超出换行）；间距 `BlockConfigSpace`（Java `block-config-space`）；`InlineBlock` 是另一种布局，别与 `Block` 混淆。
6. **HBox/VBox 弹性用 `BoxFlex`**（Java `box-flex`）：弹性子项 `BoxFlex`，固定子项用 `Width`/`Height`；"撑满 + 放 Grid"给弹性子项加 `Layout="Fit"`。见 [references/hbox-vbox.md](references/hbox-vbox.md)。
7. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc；Java 属性名 = Core 属性名转 kebab-case。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- **FineUIJava**：布局属性语义同 Core（属性名转 kebab-case、值保持 PascalCase），客户端 F.js 布局引擎与 JS 端完全相同；Region 另有 `<f:region-panel>` 便捷控件。
