---
name: fineui-buttons-toolbar
description: >
  帮助开发者使用 FineUI 的按钮与菜单：Button（语义色/图标/尺寸/徽标）、服务端与客户端点击、确认按钮、
  LinkButton、Menu/MenuButton 下拉菜单（MenuHyperLink/MenuCheckBox/MenuText/MenuSeparator）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper）。
  Trigger phrases（触发词）: "FineUI 按钮", "F.Button", "Button", "ButtonColor", "语义颜色按钮",
  "OnClick", "OnClientClick", "确认按钮", "ConfirmText", "LinkButton", "下拉菜单", "MenuButton",
  "Menu", "MenuHyperLink", "MenuCheckBox", "徽标", "Badge", "IconFont".
compatibility: FineUI v15.2+（ESM + ES2022 class；RawHtml 安全模型）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 按钮与菜单技能（Buttons & Menu）

> 工具栏（Toolbar，放在 Panel/Grid 的 `<Toolbars>` 里）见 `fineui-layout` 技能。本技能聚焦按钮与下拉菜单控件本身。

## 何时使用（When to Use）

- 按钮：语义色、图标、尺寸、徽标；服务端/客户端点击；点击前确认
- 超链接按钮 LinkButton
- 下拉菜单 Menu / MenuButton（含子菜单、可勾选菜单项）

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages（判定见 `fineui-foundation`）。
2. **点击要不要回发服务端？** 服务端事件（OnClick）vs 纯客户端（handler / OnClientClick + `EnablePostBack="false"`）。

## 各写法速览（普通按钮 + 主按钮 + 服务端点击）

```javascript
// ① F.js —— color 语义色；handler 客户端点击
F.create({ type: 'Button', renderTo: '#wrap', text: '普通按钮', cls: 'marginr' });
F.create({ type: 'Button', renderTo: '#wrap', text: '主按钮', color: 'primary',
    handler: function () { showNotify('点击了'); } });
```
```aspx
<%-- ② Pro（WebForms）：ButtonColor 语义色；OnClick 服务端 --%>
<f:Button ID="btnDefault" runat="server" Text="普通按钮" CssClass="marginr" />
<f:Button ID="btnPrimary" runat="server" Text="主按钮" ButtonColor="Primary" OnClick="btnPrimary_Click" />
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Button().ID("btnDefault").Text("普通按钮").CssClass("marginr"))
@(F.Button().ID("btnPrimary").Text("主按钮").ButtonColor(ButtonColor.Primary).OnClick(Url.Action("btnPrimary_Click")))
```
```html
<!-- ④ Core-RazorForms（TagHelper）：OnClick="方法名" -->
<f:Button ID="btnPrimary" Text="主按钮" ButtonColor="Primary" OnClick="btnPrimary_Click"></f:Button>
```
```html
<!-- ⑤ Core-RazorPages（TagHelper）：OnClick="@Url.Handler(...)" -->
<f:Button ID="btnPrimary" Text="主按钮" ButtonColor="Primary" OnClick="@Url.Handler(&quot;btnPrimary_Click&quot;)"></f:Button>
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/button.md](references/button.md) | 语义色/图标/尺寸/徽标、服务端与客户端点击、确认按钮、LinkButton |
| [references/menu.md](references/menu.md) | Menu / MenuButton 下拉菜单、MenuHyperLink/MenuCheckBox/MenuText/MenuSeparator |

## 相关技能（Related Skills）

- `fineui-foundation`：写法判定、命名约定、RawHtml（按钮文本含 HTML 时）
- `fineui-layout`：Toolbar 工具栏（放在容器 `<Toolbars>`）
- `fineui-window`：Confirm 确认框、消息框

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js `color`/`handler`（camelCase）；C# `ButtonColor`/`OnClick`（PascalCase）。
2. **语义色属性名**：F.js `color: 'primary'`；C# `ButtonColor="Primary"` / `.ButtonColor(ButtonColor.Primary)`（值：Primary/Success/Danger/Warning/Info）。**注意 C# 的 `Type` 是 `ButtonType{Button,Submit,Reset}`（表单提交/重置），不是颜色。**
3. **服务端 vs 客户端点击**：服务端 `OnClick`（MVC `Url.Action` / RazorForms 方法名 / RazorPages `@Url.Handler`）；客户端 `OnClientClick="js"` + `EnablePostBack="false"`，或 `Listener` click / F.js `handler`。
4. **确认按钮**：声明式 `ConfirmText="..." ConfirmTarget="Top"`（点击先弹确认，确认后才回发）。`ConfirmTitle`/`ConfirmIcon` 也是控件属性（默认图标 Warning）。
5. **下拉菜单项用 MenuButton/MenuHyperLink 等，不是 `MenuItem`（C#）**：C# 菜单项是 `MenuButton`（可点/带子菜单）、`MenuHyperLink`（链接）、`MenuCheckBox`（可勾选）、`MenuText`（标题）、`MenuSeparator`（分隔）。F.js 用 `type: 'MenuItem'`。详见 [references/menu.md](references/menu.md)。
6. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
