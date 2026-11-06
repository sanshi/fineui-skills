# FineUI 破坏性变更清单（v10+）

按大版本从新到旧列出**面向使用者的破坏性变更**及迁移办法。**只覆盖 v10 及以上**；v9 及以下不在范围内。

> **完整逐条以官方在线发布历史为准**：https://fineui.com/versions/ （在其中搜索“不兼容”）。本清单是高影响项的精炼；升级跨越多个大版本时，把区间内每个版本的“不兼容”条目都过一遍。

## v16.1.1 兼容补充

Core RazorForms 已恢复本页旧 Url.Handler/OnPost/Parameter 的兼容，手写 `F.doPostBack` 也可调用本页公开 OnPost；编码网址及固定结构的匿名流式控件可保持原写法，自动 ID 不作为业务接口。Core 旧 Listener 调用语句在普通策略下按函数体兼容，严格 CSP 仍用具名函数。合法业务 JSON 可由成功回调返回 `false` 接管，未接管时仍按完整 UI 响应校验。下方旧版本的迁移记录不再表示这些写法在当前版被禁止。Core RazorForms/Java 的纯动态方法名事件保留首次绑定，可按严格签名回退；模板、明确登记和主动解绑仍优先，符合签名的方法须按可请求入口校验业务权限。Core 的旧 OnGet 动态字段数组及本页 OnPost 也可保留。UIHelper 仍只保证即时更新。详情见 [旧事件兼容](../../foundation/references/razorforms-compatibility.md)和[事件与回发](../../foundation/references/events-postback.md)。

Core 升级到 v16.1.1 时，还须处理两项接口不兼容：`ControlBase.GetPostBackEventReference` 的全部重载已停用，源码调用会编译失败，旧程序集调用会抛出异常；`Parameter.FromEventArgument` 的第二参数由形参名称改为从零开始的数字下标，须修改调用并重新编译宿主及预编译视图。具体替代入口和改前、改后示例见 [v16.1 升级检查](v16.1.md)。这两项变更仅适用于 Core；Pro 的 `GetPostBackEventReference` 保持原行为。

Pro 的 `Grid.PageSizeOptions` 从 v16.1.1 起也改为 `int[]`，与 Core 对齐。后台字符串数组须改为整数数组，并重新编译依赖旧 getter/setter 签名的程序集及预编译网站。ASPX 逗号写法保留，非法选项改为立即报错，`null` 或空数组使用默认值；具体改前、改后代码见 [v16.1 升级检查](v16.1.md)。

v16.1.1 另新增 Core RazorForms / Java 的普通表格回发修改能力。Core RazorForms 普通表格推荐直接修改属性和列集合，旧 `ConfigColumns` 只作为迁移识别项，不再推荐使用；接口保留兼容，见[动态表格升级检查](v16.1.md#dynamic-grid-upgrade)。不要将其误报为接口移除，也不要将复杂编辑表格批量改成普通换列。

## v16.1.0（2026-10-28）

完整操作见 [v16.1 升级检查](v16.1.md)，覆盖以下影响：

- Core 三种模式统一 JSON；Core RazorForms 每轮处理模板，初始化数据按 Page_Init/Page_Load 用途迁移，单页模式改用 `[NoRazorForms]`。
- 严格 CSP 的脚本、动态参数和窗体回调改用结构化动作及具名函数；服务器与浏览器资源同时升级并刷新旧页面。
- Java 首屏 Page_Load 时机改变；选择、查询和页面动作采用新名称，数组及可空类型按新签名读取，已移除的消息脚本入口更换为 show/Command。
- FormRow 按钮直接按列宽布局，窄按钮用 `100% auto`；分页选项、事件参数、列定位及默认值按新版检查。
- 单选 null 与空字符串、日期格式精度、RawHtml 清空和 Raw 显示标记均有明确边界；不能依赖旧值或把显示标记当作内容审查结果。

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
- **[Core] 从 v16.0.0 起，`AddFineUI` 自动登记 `JArray`/`JObject` 模型绑定器，并在启用 RazorForms 时登记页面过滤器**。升级时检索 `Startup.cs`、`Program.cs` 及宿主的 MVC 配置扩展，删除框架过滤器 `RazorFormsFilter`（旧名 `WebFormsFilter`）的手工登记和多余的 `JsonModelBinderProvider` 登记。保留 `AddFineUI`、宿主的 MVC/Razor Pages 注册及 `.AddNewtonsoftJson()`。例如 v16.1.1 中保留 `options.Filters.Add<RazorFormsFilter>()` 或 `AddService<RazorFormsFilter>()` 会使同一次回发重复恢复状态并返回 400。详见[启动配置迁移](v16.1.md#core-razorforms-startup)。
- **[Core-RazorForms] 事件方法按完整名称兼容调用并等待 `Task`**。标签省略 `Async` 仍是推荐写法，保留异步优先，后台可切换 `void` 同步方法或 `async Task` 异步方法；显式写 `SaveAsync` 时不再优先找 `SaveAsyncAsync`，依赖旧行为的页面应明确填写实际需要调用的完整方法名。普通控件事件方法缺失会报错，`async void`、空任务及未支持返回类型须改为有效的 `void` 或 `Task`。
- **[Core-RazorForms，旧版本迁移记录；v16.1.1 已恢复本页 Parameter 兼容] 点击时给后台带额外数据曾建议不再用 `OnClickParameter1..N`**，推荐改为自定义回发（`ClickHandler` + `F.customEvent`）；需要落进 `[BindProperty]` 命名表单字段时用 `F.doPostBack({ eventName, params })`。MVC 与 RazorPages 不受影响，继续使用 `OnClickParameter`。
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
