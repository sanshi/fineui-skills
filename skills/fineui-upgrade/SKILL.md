---
name: fineui-upgrade
description: >
  把现有 FineUI 项目升级到更高的大版本（**仅 v10 及以上**），识别破坏性变更、生成迁移清单并逐项应用。
  适用于 F.js（JavaScript）、Pro（WebForms）、以及 FineUICore 的 MVC / RazorForms / RazorPages。
  Trigger phrases（触发词）: "FineUI 升级", "升级 FineUI", "FineUI upgrade", "升级到 v1x",
  "破坏性变更", "不兼容", "迁移 FineUI", "EncodeText", "raw 标签", "DateParseString".
  当用户要把 FineUI 从旧版本升级到新版本、或问“升级会不会有不兼容”时使用本技能。
compatibility: 覆盖 v10.0 → v15.2 的破坏性变更；只处理 v10 及以上大版本升级。
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 版本升级技能（Upgrade）

把现有 FineUI 项目从旧的大版本升级到更高的大版本，**只覆盖 v10 及以上**（v9 及以下不在范围内）。

## 版本自检（先做）

- 本技能内置的破坏性变更清单覆盖到 **v15.2**。
- **若用户的目标版本高于 v15.2** → 告知用户：本技能可能未包含更新版本的破坏性变更，建议先更新本技能（`npx skills update`），再继续。
- **若源版本或目标版本 ≤ v9** → 告知用户：本技能只覆盖 v10 及以上，v9 及以下请参考官方在线发布历史 https://fineui.com/versions/ 。

## 升级流程

按顺序执行，不要跳步。**每一步用中括号标注是「告知用户后继续」还是「询问用户并等待」。**

1. **确定版本区间** —— [询问用户] 当前 FineUI 版本、目标版本。若用户不确定当前版本，从项目里的 `FineUI.js` 版本注释、NuGet 包版本或 `web.config`/`csproj` 判断，并 [告知用户] 你的判断。
2. **确定写法** —— [告知用户] 项目属于哪种写法（F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages）。判定线索见 `fineui-foundation` 技能。**只需处理该写法相关的破坏性变更**（清单里 `[JS]`/`[Pro]`/`[Core]` 标注的按需取用）。
3. **对照破坏性变更** —— 读 [references/breaking-changes.md](references/breaking-changes.md)，把**版本区间内**每一条与项目代码比对。跨多个大版本时，逐版本过一遍，**优先处理“HTML 编码安全主线”**（v10 EncodeText → v15 提示 → v15.2 菜单/RawHtml），这是最容易导致页面显示异常的一类。
4. **生成迁移清单** —— 产出一个 Markdown 文件（如 `FINEUI_UPGRADE_PLAN.md`），逐项列出：受影响的文件/代码位置、变更点、**具体怎么改**（给出改前/改后）。**先写清单，不要直接改代码。**
5. **逐项确认** —— [询问用户] 对“行为变更类”（如 HBox 高度填充、行高定义、DropDownList.Text 语义）逐项确认是接受新行为还是保持旧行为；对“改名类”（DateParseString、closeArgument、Active→Activate 等）默认直接改。
6. **应用变更** —— 按确认后的清单改代码。改完 [告知用户] 改了哪些文件。
7. **验证** —— [告知用户] 建议用户：JS 端 `npm run build-f` 或直接跑页面；Core 端 `dotnet build`；Pro 端用 VS2022 msbuild 编译；并**人工核对**受 HTML 编码影响的页面（菜单、标题、提示里原本的 HTML 是否仍正常显示）。

## 参考文档

| 文件 | 内容 |
|------|------|
| [references/breaking-changes.md](references/breaking-changes.md) | v10+ 各大版本破坏性变更清单 + 迁移办法 + HTML 编码主线 |

完整逐条以官方在线发布历史为准：https://fineui.com/versions/ （在其中搜索“不兼容”）。

## 约束与规则（Constraints & Rules）

1. **只处理 v10+**：源/目标版本 ≤ v9 时明确告知不覆盖。
2. **先出清单、后改代码**：第 4 步产出迁移清单并让用户过目，不要一上来就批量改。
3. **按写法过滤**：一个项目通常只用一种写法，只处理该写法相关的破坏性变更；`[JS]`/`[Pro]`/`[Core]` 标注要看清。
4. **ESM / class 化不是使用者的破坏性变更**：v15.0 的组件层 ESM 化、ES2022 class 化官方明确“不影响对外 API”，**不要**据此改用户代码。
5. **HTML 编码是重灾区**：跨 v10/v15/v15.2 升级时，重点排查原本依赖“文本按 HTML 渲染”的地方（菜单、标题、提示、空状态 EmptyText、消息框），逐一改为可信 HTML 声明（RawHtml，见 `fineui-foundation` 的 rawhtml.md）。区分可信内容与用户输入——用户输入不要声明为可信。
6. **绝不编造变更**：不确定某属性/方法在目标版本是否变化时，查在线发布历史（https://fineui.com/versions/ ）或官网，别猜。

## 官方资源

- 在线发布历史：https://fineui.com/versions/ （权威，逐版本“不兼容”标注）
- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
