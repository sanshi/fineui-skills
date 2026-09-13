# FineUI 破坏性变更清单（v10+）

按大版本从新到旧列出**面向使用者的破坏性变更**及迁移办法。**只覆盖 v10 及以上**；v9 及以下不在范围内。

> **完整逐条以官方在线发布历史为准**：https://fineui.com/versions/ （在其中搜索“不兼容”）。本清单是高影响项的精炼；升级跨越多个大版本时，把区间内每个版本的“不兼容”条目都过一遍。

## v16.0.0（2026-09-13）

- **[Core/Pro] 客户端点击统一到具名函数 + `ClickHandler`**。新代码不再使用 `OnClientClick`，也不把脚本串塞进 Handler；Java 只提供 `click-handler`。
  - 回调首参统一为 `event`；要中断确认框、导航或回发必须显式 `return false`。
  - Core/Pro 的执行顺序调整为“表单验证 → beforeclick → 兼容 OnClientClick → click（含 ClickHandler）→ 确认框与回发”，迁移含返回值的菜单项和校验按钮时需回归行为。
- **[Pro] 新增 `EnableImplicitPostBack` 与 `EnableImplicitChangeEvents`**。类库默认均为 `true` 保持老项目行为；新项目推荐都设为 `false`。
  - 动作控件：声明 `OnClick` 等服务端事件即自动回发；纯客户端按钮只写 `ClickHandler`，不再逐个写 `EnablePostBack="false"`。
  - 变化控件：声明 `SelectedIndexChanged`、`TextChanged`、`CheckedChanged` 等即自动回发，不再重复写 `AutoPostBack="true"`；其他控件回发不会连带触发当前字段的变化事件。
  - 显式 `EnablePostBack` / `AutoPostBack` 始终优先。
- **[Pro/Core-RazorForms/Java] 页面级自定义回发统一为 `F.customEvent` + `Page_CustomEvent`**。对象/数组直接作为 JSON 参数；不要再手写 `__doPostBack` 拼接字符串。
  - [Pro] `PageManager.CustomEvent`（ASPX `OnCustomEvent`）与 `GetCustomEventReference` 已废弃，仅保留运行兼容。
- **[Pro/Core-RazorForms/Java] 普通控件回发参数改为以 F.js 客户端事件名开头**。Pro 不兼容空默认参数、旧 `Command` / `Page` / `ClearIcon` 标识和手写旧控件 target；若业务代码直接构造 `__doPostBack`，改为标准控件事件或 `F.customEvent`。
- **[Pro] Grid 新代码用 `RenderField.Commands`**；`Command` 只属于 `RenderField`。`LinkButtonField` / `WindowField` 保留兼容，但不再作为行内编辑/删除的推荐方案。
- **[Core/Pro] 新项目设置 `AllowDangerousRawTag=false` 与 `AllowDangerousScriptTag=false`**。把 `<raw>` 改为 RawHtml，把控件文案中的脚本移到页面具名函数。
- **[Core] 最低运行环境提高到 .NET 8**，Newtonsoft.Json 依赖提高到 13.x。仍在 .NET 6/7 或引用旧 Newtonsoft.Json 的项目必须先升级。
- **[Pro] 开启 `EnableFStateValidation` 的项目必须显式配置非 `AutoGenerate` 的 `<machineKey validationKey="...">`**；Web 农场节点使用同一密钥。
- **[Pro] 单元格编辑改为 `GetModifiedData()`**；`GetModifiedDict` / `GetDeletedList` / `GetNewAddedList` 已废弃。
- **[JS] 不再支持 IE**；移除 `F.isIE8()` / `F.isIE89()` / `F.isIE9()` / `F.isIE()`。

完整点击与回发迁移见 `fineui-foundation` 的 `events-postback.md`。

---

## 贯穿主线：HTML 编码安全（v10 → v16）

FineUI 从 v10 起逐步把“控件文本默认按 HTML 编码、可信 HTML 需显式声明”落地。**跨过这几个版本升级时，这是最可能让页面显示异常（HTML 变成纯文本）的一类变更**：

| 版本 | 变更 | 迁移 |
|------|------|------|
| **v10.0** | 新增全局 `EncodeText`（默认 `true`）：面板标题、表格列、消息框、树、按钮、菜单等**所有控件文本默认 HTML 编码**。当时引入 `<raw>…</raw>`。 | 升到当前版本时不要再采用历史 `<raw>`，直接迁移为 RawHtml 声明。 |
| **v15.0** | 表格**单元格提示**也默认 HTML 编码（`EncodeText=true`）；`EncodeText=false` 时自动移除 `<script>`。 | 提示里的可信 HTML 同样需声明。 |
| **v15.2** | **菜单项文本改为默认 HTML 编码**（MenuItem/MenuButton/MenuHyperLink/MenuText/MenuCheckBox）；用 **RawHtml 声明式**（`F.rawHtml`/`XxxRawHtml`/`new RawHtml`）**取代 `<raw>`**；新增全局开关 `AllowDangerousRawTag`。 | 见下方 v15.2 条目；旧 `<raw>` 改为 RawHtml 写法（见 `fineui-foundation` 的 rawhtml.md）。 |
| **v16.0** | 新增 `AllowDangerousScriptTag`，并在官方项目中关闭 `<raw>` 与文案内 `<script>` 两类兼容入口。 | 设置两个开关为 `false`；可信 HTML 用 RawHtml，客户端逻辑移到具名函数并用 `ClickHandler` 引用。 |

---

## v15.2.0（2026-07-26）

- **菜单项文本默认 HTML 编码**（不兼容）。升级后菜单项里写的 HTML（`<div>`/`<i>`/`<img>` 等）会被当纯文本显示。
  - 迁移（仅对开发者确信可信的 HTML；用户输入/数据库内容不要声明）：
    - **F.js**：`text: '<i>..</i>'` → `text: F.rawHtml('<i>..</i>')`
    - **Pro**：`Text="<i>..</i>"` → `TextRawHtml="<i>..</i>"`；后置 `x.Text=".."` → `x.TextRawHtml = new RawHtml("..")`
    - **Core-MVC**：`.Text("<i>..</i>")` → `.TextRawHtml(new RawHtml("<i>..</i>"))`
    - **Core-RazorForms/RazorPages**：`Text="<i>..</i>"` → `_TextRawHtml="<i>..</i>"`（或 `TextRawHtml="@(new RawHtml(".."))"`）
  - 详见 `fineui-foundation` 技能的 [rawhtml.md]。

## v15.0.0（2026-06-28）

- **表格单元格提示默认 HTML 编码**（`EncodeText=true`）；`EncodeText=false` 时自动移除 `<script>`。提示里的可信 HTML 需声明。
- **主题系统从 SCSS 预编译改为 CSS 变量（CSS Variables）**。**影响自定义主题**：不再需要 Sass；自定义主题改为在 `res/themes/` 下复制 `theme.config` 改颜色，用 `generate-theme` 生成。用内置主题者无需改动。
- **（内部，无需迁移）** 组件层迁移到 ESM、ES2022 class 替代 John Resig Class.js——**官方明确“不影响对外 API”**。使用者代码无需为此改动。

## v13.1.0（2026-02-28）

- **[Core] 表格列 `FieldFormat` 改名为 `DateParseString`**（不兼容）。用于 `FieldType=Date` 且数据为字符串时的日期解析。
  - 迁移：Core 端把日期列的 `FieldFormat="..."` 改为 `DateParseString="..."`。（F.js 端 `fieldFormat` 语义不同，用于渲染格式，未改。）

## v12.0.0（2025-03-14）

- **自定义 CSS 选择器变更**：`.f-grid-row .f-grid-cell-inner` → `.f-grid-row .f-grid-cell-text`（不兼容）。检查项目里针对表格单元格的自定义样式并改名。
- **表格行高定义不再包含单元格内边距**（不兼容）。若之前依赖固定行高像素值，需重新核对。

## v11.x（2024–2025）

- **[Core] 标签属性 `ValidateForms`、`DataDisplayFields` 类型由 `string` 改为 `string[]`**（不兼容）。按数组传值。
- **容器 HBox 布局未设高度时**，`BoxConfigAlign` 默认 `Stretch`，未设高度的子项会被拉伸填充容器高度（不兼容）。检查依赖旧行为的布局。

## v10.x（v10.0 之后的小版本）

- **`Active` 与 `Activate` 区分**（`Active` 作形容词）（不兼容）：`TabStrip.GetActiveTabReference` → `GetActivateTabReference`，客户端 `activeTab` → `activateTab`；`Tab.GetActiveReference` → `GetActivateReference`，客户端 `active` → `activate`。属性如 `ActiveTabIndex`/`ActivePanelIndex` 保持形容词形式。
- **窗体关闭参数名 `closeArgument` → `Window1_closeArgument`**（不兼容）。
  - Core 回发：`OnPostWindow1_Close(string[] Grid1_fields, string Window1_closeArgument)`；JS `close: function(event, closeArgument){}` 从事件参数取。
- **FineUIPro 表格事件 `PageIndexChange` → `PageIndexChanged`**（改名）。
- **`DropDownList.Text` 语义变化**：现在任何情况下都表示输入框显示文本（之前仅用户输入时有效）。多选取文本可直接用 `.Text`；判断是否用户输入用 `IsUserInput`。

## v10.0.0（2024-03-20）—— 安全基石

- **新增全局配置项 `EncodeText`（默认 `true`）**（不兼容）：所有控件文本属性渲染前 HTML 编码（面板标题、表格列、消息框正文、树节点、按钮、菜单等）。
  - 例外（不编码）：ListItem 的 `Display`、表格列渲染函数、Mvc/Core 面板的 `Content` 扩展方法。
  - 当前迁移：想输出可信 HTML 的地方直接使用 RawHtml 声明式（见主线表），不要新增历史 `<raw>`。
  - 作用域：`Web.config`/`appsettings.json`（站点）、`PageManager`（页面）、`Grid`（表格）、`GridColumn`（列）分级可设 `EncodeText`。

---

## 使用提示

- **只升一个大版本**时：直接看目标版本条目。
- **跨多个大版本**时（如 v10 → v16）：按主线表处理 HTML 编码，再把区间每个版本的“不兼容”条目过一遍（以 https://fineui.com/versions/ 为准）。
- 大量“不兼容”条目是**某一端专属**（标注 `[JS]`/`[Pro]`/`[Core]`）——只关心你项目所用的写法。
