# FineUI 版本升级

本分类覆盖 v10 至 v16.2.0。先从业务项目依赖、脚本头或现有资源确认源版本与目标版本，再按开发模式筛选变更。官网只保留当前版本资料，版本不匹配时优先使用同版源码、示例及发布历史。

## 选择版本范围

| 版本范围 | 读取资料 |
| --- | --- |
| 升级到 v16.2.0，含跨过 v16.1.2 | [v16.2.0 升级检查](references/v16.2.md) |
| 跨过 v16.1.0、v16.1.1 | [v16.1 迁移](references/v16.1.md) |
| 跨 v10 至 v16 的 API、安全输出和布局变化 | [破坏性变更](references/breaking-changes.md) |

源或目标版本为 v9 及更早时，使用相应版本的官方发布历史，本分类不提供完整覆盖。目标版本高于 v16.2.0 时先读取目标版本的实际变更并更新技能，不能把本清单当作未来版本说明。

## 迁移方式

1. 确认源版本、目标版本和 F.js/Pro/Core MVC/Core RazorForms/Core RazorPages/Java 模式。已有依赖足以判断时直接使用证据；关键版本无法确定时再询问用户。
2. 按版本区间读取变更，搜索项目中的受影响 API。Java 首发为 v16.0，历史 C# 专属迁移不套用到 Java。
3. 区分必须处理的编译或行为变化与可选的新功能，列出受影响位置及替代写法。用户只要求方案时交付方案；用户已明确要求执行升级时，完成已授权的修改和验证。有多种业务语义可选且无法由代码判断时，再给出具体选择。
4. 先处理实际兼容问题，再按需要更新推荐写法。保留项目已有业务权限校验、模型绑定及存储方式。
5. 验证真实页面操作，包括首屏、后续回发、保存和异常处理。程序集、客户端 JS/CSS 及编辑器或图表独立资源保持一致。

Core v16.0.0 起使用 `AddFineUI` 的宿主不再手工重复登记框架模型绑定器和 RazorForms 过滤器，迁移见 [启动配置](references/v16.1.md#core-razorforms-startup)。Core v16.1.1 的本页旧事件兼容见 [RazorForms 旧事件](../foundation/references/razorforms-compatibility.md)，不能一律建议关闭 RazorForms。

## 常见判断

- 业务数据展示为可信 HTML 前先验证来源并清洗，不能为兼容旧页面而批量取消编码。历史 `<raw>` 迁移为明确的 `RawHtml` 声明。
- Core RazorForms/Java 的普通表格可直接修改属性和列集合。旧 `ConfigColumns` 保留兼容，不当作已移除 API，也不对编辑器、过滤器等复杂表格机械换列。
- ESM、class 等内部源码重构不自动构成使用者的 API 迁移；只处理影响业务代码的公开行为变化。
- 不确定的改动以实际目标包和对应发布历史核实，不推断未发布能力。

[官方发布历史](https://fineui.com/versions/) · [Core API](https://fineui.com/core/api/) · [Pro API](https://fineui.com/pro/api/) · [JS API](https://fineui.com/js/api/)
