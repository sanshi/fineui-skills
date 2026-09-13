---
name: fineui-buttons-toolbar
description: >
  帮助开发者使用 FineUI 的按钮与菜单：Button（语义色/图标/尺寸/徽标）、服务端与客户端点击、确认按钮、
  LinkButton、ButtonGroup（按钮分组/互斥按下/多按下）、Menu/MenuButton 下拉菜单（MenuHyperLink/MenuCheckBox/MenuText/MenuSeparator）。
  覆盖 F.js（JavaScript）、Pro（WebForms）、FineUICore 的 MVC（Fluent API）/ RazorForms / RazorPages（TagHelper），
  以及 FineUIJava（Spring Boot + Thymeleaf 方言标签，kebab-case）。
  Trigger phrases（触发词）: "FineUI 按钮", "F.Button", "Button", "ButtonColor", "语义颜色按钮",
  "OnClick", "ClickHandler", "客户端点击", "确认按钮", "ConfirmText", "LinkButton", "下拉菜单", "MenuButton",
  "Menu", "MenuHyperLink", "MenuCheckBox", "徽标", "Badge", "IconFont",
  "ButtonGroup", "按钮分组", "pressGroup", "互斥按下", "EnablePress", "EnablePressGroup",
  "FineUIJava", "Spring Boot", "Thymeleaf", "@FineUIPage", "f:button", "button-color", "enable-press-group".
metadata:
  author: FineUI
  version: "16.0"
  compatibility: FineUI v16.0（ClickHandler 与事件驱动回发）
---

# FineUI 按钮与菜单技能（Buttons & Menu）

> 工具栏（Toolbar，放在 Panel/Grid 的 `<Toolbars>` 里）见 `fineui-panel` 技能。本技能聚焦按钮、按钮分组与下拉菜单控件本身。

## 何时使用（When to Use）

- 按钮：语义色、图标、尺寸、徽标；服务端/客户端点击；点击前确认
- 按钮分组 ButtonGroup：无间距拼接、互斥按下（单选）、多按下、纵向
- 超链接按钮 LinkButton
- 下拉菜单 Menu / MenuButton（含子菜单、可勾选菜单项）

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / Java（Spring Boot）（判定见 `fineui-foundation`）。
2. **点击要不要回发服务端？** 服务端事件用 `OnClick` / `on-click`；纯客户端用页面具名函数 + `ClickHandler` / `click-handler`。

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
```html
<!-- FineUIJava（Thymeleaf 方言）：标签/属性全 kebab-case；on-click="方法名" -->
<f:button id="btnPrimary" text="主按钮" button-color="Primary" on-click="btnPrimary_Click"></f:button>
```
```java
// FineUIJava 页面类：@FineUIPage + extends FineUIPageBase；处理器返回 void
@FineUIPage("button/button-click")
public class ButtonClick extends FineUIPageBase {
    public void btnPrimary_Click(Object sender, EventArgs e) { showNotify("这是服务器端事件"); }
}
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/button.md](references/button.md) | 语义色/图标/尺寸/徽标、服务端与客户端点击、确认按钮、LinkButton |
| [references/menu.md](references/menu.md) | Menu / MenuButton 下拉菜单、MenuHyperLink/MenuCheckBox/MenuText/MenuSeparator |

## 相关技能（Related Skills）

- `fineui-foundation`：写法判定、命名约定、RawHtml（按钮文本含 HTML 时）
- `fineui-panel`：Toolbar 工具栏（放在容器 `<Toolbars>`）
- `fineui-window`：Confirm 确认框、消息框

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js `color`/`handler`（camelCase）；C# `ButtonColor`/`OnClick`（PascalCase）；**Java `button-color`/`on-click`（kebab-case），值仍 PascalCase**。
2. **语义色属性名**：F.js `color: 'primary'`；C# `ButtonColor="Primary"` / `.ButtonColor(ButtonColor.Primary)`；**Java `button-color="Primary"`**（值：Primary/Success/Danger/Warning/Info）。**注意 C# 的 `Type` 是 `ButtonType{Button,Submit,Reset}`（表单提交/重置），不是颜色。**
3. **服务端 vs 客户端点击**：服务端 `OnClick`（MVC `Url.Action` / RazorForms 方法名 / RazorPages `@Url.Handler` / Java `on-click`）；客户端用页面具名函数 + `ClickHandler`（Java `click-handler`）。新代码不使用 `OnClientClick`，也不把脚本串写进 Handler。
4. **Pro 官方推荐配置**：站点统一设置 `EnableImplicitPostBack="false"` 与 `EnableImplicitChangeEvents="false"`。此时纯客户端按钮无需逐个写 `EnablePostBack="false"`，声明 `OnClick` 的按钮会自动回发；显式控件属性始终优先。
5. **确认按钮**：声明式 `ConfirmText="..." ConfirmTarget="Top"`（点击先弹确认，确认后才回发）。`ConfirmTitle`/`ConfirmIcon` 也是控件属性（默认图标 Warning）。
6. **下拉菜单项用 MenuButton/MenuHyperLink 等，不是 `MenuItem`（C#/Java）**：C# 菜单项是 `MenuButton`（可点/带子菜单）、`MenuHyperLink`（链接）、`MenuCheckBox`（可勾选）、`MenuText`（标题）、`MenuSeparator`（分隔）；**Java 对应 `<f:menu-button>`/`<f:menu-hyper-link>`/`<f:menu-check-box>`/`<f:menu-text>`/`<f:menu-separator>`**。F.js 用 `type: 'MenuItem'`。详见 [references/menu.md](references/menu.md)。
7. **按钮分组「按下」用 `enablePress`（不是废弃的 `enableToggle`）**：F.js `enablePress`/`pressGroup`；C# `EnablePress`/`EnablePressGroup`（**不是 `PressGroup`**）；Java `enable-press`/`enable-press-group`。
8. **绝不编造 API**：不确定就查官网 API 或 `F/doc/` JSDoc。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
- **FineUIJava**：控件属性 kebab-case、值同 Core，客户端 F.js API（`F.ui.btn.*`）与 JS 端完全相同；查属性时参考 Core API 再按命名约定转 kebab-case。
