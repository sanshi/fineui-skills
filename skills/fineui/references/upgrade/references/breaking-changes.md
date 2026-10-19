# FineUI 破坏性变更清单（v10+）

按大版本从新到旧列出**面向使用者的破坏性变更**及迁移办法。**只覆盖 v10 及以上**；v9 及以下不在范围内。

> **完整逐条以官方在线发布历史为准**：https://fineui.com/versions/ （在其中搜索“不兼容”）。本清单是高影响项的精炼；升级跨越多个大版本时，把区间内每个版本的“不兼容”条目都过一遍。

## 待发布：Pro/Core 的 FormRow 按钮直接参与列布局

- Pro 和 Core 的 RazorForms、MVC、RazorPages 不再给 `FormRow.Items` 中的按钮自动加一层 `Panel`。按钮直接接受这一列的宽度；Java 原本如此，没有这项旧行为需要迁移。
- **先查页面**：找出“字段 + 按钮”直接放在同一 `FormRow` 中的地方。不设置 `ColumnWidths` 和明确写 `50% 50%` 都是两列均分；升级后按钮也占半行。业务本来就想让按钮占半行时无需修改。
- **需要窄按钮时**：给该行设置 `ColumnWidths="100% auto"`；Core MVC 使用 `.ColumnWidths("100% auto")`。下面的 Core TagHelper 示例只改 `FormRow` 起始标签，按钮仍保留原来的 ID、文字和事件：

  ```cshtml
  <!-- 改前：未指定列宽，按钮升级后也占半行 -->
  <f:FormRow ID="SearchRow">
      <Items>
          <f:TextBox ID="SearchText" Label="关键词"></f:TextBox>
          <f:Button ID="SearchButton" Text="查询"></f:Button>
      </Items>
  </f:FormRow>

  <!-- 改后：文本框使用剩余空间，按钮保持内容宽度 -->
  <f:FormRow ID="SearchRow" ColumnWidths="100% auto">
      <Items>
          <f:TextBox ID="SearchText" Label="关键词"></f:TextBox>
          <f:Button ID="SearchButton" Text="查询"></f:Button>
      </Items>
  </f:FormRow>
  ```

  Pro 的服务器控件继续保留原有 `runat="server"`；Java 标签写 `column-widths="100% auto"`，但 Java 原本就直接布局，没有这项迁移要求。
- `50% auto` 会让字段只占扣除按钮后剩余宽度的一半，行尾仍有空白；不要把它误当作保留旧窄按钮效果的通用替代。`auto` 列保留明确设置的 `Width`，否则按控件默认或内容宽度显示。窄窗口下仍需检查标签与输入框是否有足够空间。

## 待发布：Core 三种模式统一 JSON 与严格 CSP

- 默认首屏与受控回发改为 JSON，普通控件回发使用当前完整 URL，不再自动增加 `handler`。无需新增架构开关；MVC、RazorPages 使用相同首屏/响应 JSON 的无状态子集，保留原生路由与参数绑定；独立业务数据、下载保留原结果合同。
- `AuthenticationTimeout` 等搜索响应 JS 的拦截代码必须改为解析 JSON，查找 `commands` 中的 `page.redirect`；不提供旧脚本正文模拟。
- 普通策略保留 `RegisterStartupScript`、旧引用方法和字符串消息回调。严格内容安全策略（Content Security Policy，CSP）下使用结构化动作、`XxxAjaxHelper` 和具名函数。`CspScriptsAllowUnsafeEval` 未显式配置时三种 Core 模式均为 `false`；nonce 不能放行动态编译脚本。
- 官方 Core RazorForms、MVC、RazorPages 示例和空项目，以及 AppBox、QuickStart，已显式启用同源脚本和每请求 nonce，关闭 `unsafe-inline`、`unsafe-eval`；类库仍默认关闭 CSP。客户项目若开启严格策略，须检查自己的脚本、消息回调、内联事件与第三方组件。
- 全局开启 RF 时，特定原生页面在模型类或共同基类上添加可继承的 `[NoRazorForms]`。删除 PageManager 每页模式设置、旧名及对应标签/流式设置；这些入口已废弃，setter 明确报迁移错误。普通独立上传、下载、JSON 的 OnPost 接口不要求标记。Core 请求删除 __FPROTOCOL，没有增加签名模式凭据。
- MVC/RazorPages 的旧 Parameter 字符串表达式在严格 CSP 下改为 FromFunction/FromEventArgument/FromData；不执行响应脚本文本。首屏配置中只有顶层 debug，不新增数字协议 version。
- 显式表格 `ColumnID` 必须在整个列树中唯一，点号转下划线后也不能冲突。匿名动态列和编辑器由框架分配 ID。
- 更新或回滚时一起替换服务器 DLL 与客户端资源，并刷新旧页签；协议不匹配或保存请求失败后不自动重发，先核对保存结果。
- 随官方项目分发的 `node tools/razorforms/check-razorforms-upgrade.mjs .` 只读报告协议、严格 CSP 和人工审计项，不自动改业务代码。

## v16.0.0（2026-09-30）

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
- **[Core] 最低运行环境提高到 .NET 8**；NuGet 对 `Microsoft.AspNetCore.Mvc.NewtonsoftJson` 仅要求 8.0.31 或更高版本，不设上限。仍在 .NET 6/7 的项目必须先升级；较高版本的宿主应显式引用与自身 ASP.NET Core 主版本一致的适配包。
- **[Pro] 开启 `EnableFStateValidation` 的项目必须显式配置非 `AutoGenerate` 的 `<machineKey validationKey="...">`**；Web 农场节点使用同一密钥。
- **[Pro] 单元格编辑改为 `GetModifiedData()`**；`GetModifiedDict` / `GetDeletedList` / `GetNewAddedList` 已废弃。
- **[Core] `AddFineUI` 自动登记 `JArray`/`JObject` 模型绑定器，并在启用 RazorForms 时登记所需过滤器**。Startup 不再需要手工添加这两项；老项目保留原手工登记也不会重复执行，建议升级后删除以简化 Startup。
- **[Core-RazorForms] 事件方法按完整名称兼容调用并等待 `Task`**。标签省略 `Async` 仍是推荐写法，保留异步优先，后台可切换 `void` 同步方法或 `async Task` 异步方法；显式写 `SaveAsync` 时不再优先找 `SaveAsyncAsync`，依赖旧行为的页面应明确填写实际需要调用的完整方法名。普通控件事件方法缺失会报错，`async void`、空任务及未支持返回类型须改为有效的 `void` 或 `Task`。
- **[Core-RazorForms] 点击时给后台带额外数据不再用 `OnClickParameter1..N`**，推荐改为自定义回发（`ClickHandler` + `F.customEvent`）；需要落进 `[BindProperty]` 命名表单字段时用 `F.doPostBack({ eventName, params })`。MVC 与 RazorPages 不受影响，继续使用 `OnClickParameter`。
- **[Core] 配置项 `ExcludedURLs`（不使用 FineUI 中间件的网址列表）已废弃**，不再需要配置；已有配置继续生效，无需修改。
- **[JS] 不再支持 IE**；移除 `F.isIE8()` / `F.isIE89()` / `F.isIE9()` / `F.isIE()`。

完整点击与回发迁移见 `基础` 的 `events-postback.md`。

---

## 贯穿主线：HTML 编码安全（v10 → v16）

FineUI 从 v10 起逐步把“控件文本默认按 HTML 编码、可信 HTML 需显式声明”落地。**跨过这几个版本升级时，这是最可能让页面显示异常（HTML 变成纯文本）的一类变更**：

| 版本 | 变更 | 迁移 |
|------|------|------|
| **v10.0** | 新增全局 `EncodeText`（默认 `true`）：面板标题、表格列、消息框、树、按钮、菜单等**所有控件文本默认 HTML 编码**。当时引入 `<raw>…</raw>`。 | 升到当前版本时不要再采用历史 `<raw>`，直接迁移为 RawHtml 声明。 |
| **v15.0** | 表格**单元格提示**也默认 HTML 编码（`EncodeText=true`）；`EncodeText=false` 时自动移除 `<script>`。 | 提示里的可信 HTML 同样需声明。 |
| **v15.2** | **菜单项文本改为默认 HTML 编码**（MenuItem/MenuButton/MenuHyperLink/MenuText/MenuCheckBox）；用 **RawHtml 声明式**（`F.rawHtml`/`XxxRawHtml`/`new RawHtml`）**取代 `<raw>`**；新增全局开关 `AllowDangerousRawTag`。 | 见下方 v15.2 条目；旧 `<raw>` 改为 RawHtml 写法（见 `基础` 的 rawhtml.md）。 |
| **v16.0** | 新增 `AllowDangerousScriptTag`，并在官方项目中关闭 `<raw>` 与文案内 `<script>` 两类兼容入口。 | 设置两个开关为 `false`；可信 HTML 用 RawHtml，客户端逻辑移到具名函数并用 `ClickHandler` 引用。 |

---

## v15.2.0（2026-07-26）

- **菜单项文本默认 HTML 编码**（不兼容）。升级后菜单项里写的 HTML（`<div>`/`<i>`/`<img>` 等）会被当纯文本显示。
  - 迁移（仅对开发者确信可信的 HTML；用户输入/数据库内容不要声明）：
    - **F.js**：`text: '<i>..</i>'` → `text: F.rawHtml('<i>..</i>')`
    - **Pro**：`Text="<i>..</i>"` → `TextRawHtml="<i>..</i>"`；后置 `x.Text=".."` → `x.TextRawHtml = new RawHtml("..")`
    - **Core-MVC**：`.Text("<i>..</i>")` → `.TextRawHtml(new RawHtml("<i>..</i>"))`
    - **Core-RazorForms/RazorPages**：`Text="<i>..</i>"` → `_TextRawHtml="<i>..</i>"`（或 `TextRawHtml="@(new RawHtml(".."))"`）
  - 详见 `基础` 技能的 [rawhtml.md]。

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
- **FineUI.Pro 表格事件 `PageIndexChange` → `PageIndexChanged`**（改名）。
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
