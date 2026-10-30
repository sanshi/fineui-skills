# FineUI 版本升级技能（Upgrade）

Core v16.1.1 补齐开启 RazorForms 时的本页旧 Url.Handler/OnPost/Parameter 兼容。升级 v16.1.0 出现 Parameter 或原生 UI 入口错误时，先读 [旧事件兼容](../foundation/references/razorforms-compatibility.md)，不要一律建议关闭 RazorForms。DLL 与资源一起替换并刷新页面；严格 CSP 下字符串表达式仍需迁移为具名参数提供器。

把现有 FineUI 项目升级到更高版本，**只覆盖 v10 及以上**（v9 及以下不在范围内）。

## 版本自检（先做）

- 本分类的变更清单覆盖到 **v16.1.1**；迁移细节见 [v16.1 升级检查](references/v16.1.md)。版本号以实际目标包为准，发布日期尚未到达时先确认使用的是候选包还是正式包。
- **若目标版本高于 v16.1.1**：先核对目标包及对应发布历史，必要时运行 `npx skills update` 更新技能；不能把本清单当作后续版本的完整说明。
- **若源版本或目标版本 ≤ v9** → 告知用户：本分类只覆盖 v10 及以上，v9 及以下请参考官方在线发布历史 https://fineui.com/versions/ 。

## 升级流程

Core 模式升级须核查三种模式共同 JSON、严格 CSP 的缺省值及 `[NoRazorForms]` 页面标记迁移；普通独立接口不要求标记。Pro/Core 页面还应查找 FormRow 内直接放置的按钮，判断升级后是否需要 `100% auto` 保留窄按钮。具体影响与替代写法见破坏性变更清单。

按顺序执行，不要跳步。**每一步用中括号标注是「告知用户后继续」还是「询问用户并等待」。**

1. **确定版本区间** —— [询问用户] 当前 FineUI 版本、目标版本。若用户不确定当前版本，从项目里的 `FineUI.js` 版本注释、NuGet 包版本或 `web.config`/`csproj` 判断，并 [告知用户] 你的判断。
2. **确定写法** —— [告知用户] 项目属于哪种写法（F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / Java）。判定线索见 `基础` 技能。**先处理该写法相关的破坏性变更，再核对推荐写法更新**（清单里 `[JS]`/`[Pro]`/`[Core]` 标注的按需取用）。**若项目是 FineUI.Java**：它首发为 v16.0，随后已发布 v16.0.1；升级到 v16.1.0 同样需要核对 Java 生命周期、选择 API、空值和类型变化；更早版本的 C#/.NET 专属条目不适用，F.js 客户端变化按实际版本区间核对。
3. **对照版本变化** —— 读 [references/breaking-changes.md](references/breaking-changes.md)，把**版本区间内**每一条与项目代码比对；候选版条目先确认目标包是否包含该改动。跨多个大版本时，逐版本过一遍，优先处理“HTML 编码安全主线”、v16 的事件/回发迁移，以及受影响页面的 FormRow 按钮宽度。
   目标为 v16.1.1 或更新版本时，另检查 Core RazorForms 的 `ConfigColumns`、Java 的 `configColumns`、`GridConfigOptions` 和回发换列代码；按[动态表格升级检查](references/v16.1.md#dynamic-grid-upgrade)更新推荐写法。Core RazorForms 普通表格统一推荐直接修改属性和 `Columns`，不再推荐 `ConfigColumns`；旧接口仅作为迁移识别项。带编辑器、过滤器等复杂控件的表格单独核对，不套用普通换列流程。

4. **生成迁移清单** —— 产出一个 Markdown 文件（如 `FINEUI_UPGRADE_PLAN.md`），逐项列出：受影响的文件/代码位置、变更点、**具体怎么改**（给出改前/改后）。**区分必须修复的兼容问题与推荐写法更新，先写清单，不要直接改代码。**
5. **逐项确认** —— [询问用户] 对“行为变更类”（如 HBox 高度填充、行高定义、DropDownList.Text 语义）逐项确认是接受新行为还是保持旧行为；对“改名类”（DateParseString、closeArgument、Active→Activate 等）默认直接改。
6. **应用变更** —— 按确认后的清单改代码。改完 [告知用户] 改了哪些文件。
7. **验证** —— [告知用户] 建议用户：JS 端构建或直接跑页面；Core 端 `dotnet build`；Pro 端用 VS2022 msbuild 编译。**人工核对**受 HTML 编码影响的页面（菜单、标题、提示里原本的 HTML 是否仍正常显示）；受 FormRow 改动影响的页面还应在桌面和窄窗口下检查按钮宽度、字段输入和点击事件。

## 参考文档

| 文件 | 内容 |
|------|------|
| [references/breaking-changes.md](references/breaking-changes.md) | v10+ 版本变更总览与 HTML 编码主线 |
| [references/v16.1.md](references/v16.1.md) | v16.0.x / v16.1.0 升级到 v16.1.1 的模式、API、类型与动态表格推荐写法 |

完整逐条以官方在线发布历史为准：https://fineui.com/versions/ （在其中搜索“不兼容”）。

## 约束与规则（Constraints & Rules）

1. **只处理 v10+**：源/目标版本 ≤ v9 时明确告知不覆盖。
2. **先出清单、后改代码**：第 4 步产出迁移清单并让用户过目，不要一上来就批量改。
3. **按写法过滤**：一个项目通常只用一种写法，只处理该写法相关的破坏性变更；`[JS]`/`[Pro]`/`[Core]` 标注要看清。
4. **ESM / class 化不是使用者的破坏性变更**：v15.0 的组件层 ESM 化、ES2022 class 化官方明确“不影响对外 API”，**不要**据此改用户代码。
5. **HTML 编码是重灾区**：跨 v10 至 v16 升级时，重点排查原本依赖“文本按 HTML 渲染”的地方（菜单、标题、提示、空状态 EmptyText、消息框），逐一改为可信 HTML 声明（RawHtml，见 `基础` 的 rawhtml.md）。区分可信内容与用户输入——用户输入不要声明为可信。
6. **绝不编造变更**：不确定某属性/方法在目标版本是否变化时，查在线发布历史（https://fineui.com/versions/ ）或官网，别猜。

## 官方资源

- 在线发布历史：https://fineui.com/versions/ （权威，逐版本“不兼容”标注）
- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
