---
name: fineui-theming
description: >
  设置或切换 FineUI 已有的内置主题和自定义主题，处理全局配置、PageManager 覆盖与 Cookie 偏好。
  覆盖 F.js、Pro、Core 三种模式及 Java。新增主题、品牌配色和 theme.config 生成任务
  使用 fineui-custom-theme。
metadata:
  author: FineUI
  version: "16.0.0-rc.3"
  compatibility: FineUI v15+（主题系统重构为 CSS Variables）
---

# FineUI 主题设置与切换

> v15 起主题系统从 SCSS 预编译重构为 **CSS Variables**：运行时只需加载 `themes/{主题}/theme.css` 覆盖颜色变量，无需 sass。

## 何时使用

- 设置站点全局主题
- 让用户运行时切换主题（下拉/图墙）
- 应用已经生成的自定义主题；创建新配色时使用 [fineui-custom-theme](../fineui-custom-theme/SKILL.md)

## 内置主题

`Theme` 枚举（配置里用帕斯卡名，F.js 用小写）：

- **Pure 系列**：`Pure_Black`（默认）、`Pure_Green`、`Pure_Blue`、`Pure_Purple`、`Pure_Orange`
- **jQuery UI 系列**：`Cupertino`、`Start`、`Dark_Hive`、`Flick`、`South_Street`
- 另有 `chinese_red` 等；深色主题如 `Dark_Hive`。

## 设置全局主题（速览）

```javascript
// ① F.js —— F.init 里设初始主题（小写）
F.init({ theme: 'pure_black' });
```
```xml
<!-- ② Pro（WebForms）—— Web.config 的 <FineUI.Pro> 段 -->
<FineUI.Pro DebugMode="false" Theme="Pure_Black" EnableAnimation="true" />
```
```json
// ③ Core（MVC / RazorForms / RazorPages 三套一致）—— appsettings.json
"FineUI": { "Theme": "Pure_Black", "EnableAnimation": true }
```
```properties
# ④ FineUI.Java（Spring Boot）—— application.properties 的 fineui.* 键（键名 kebab-case、主题名小写）
fineui.theme=pure_black
fineui.enable-animation=true
```

> Java 全局默认在 `application.properties`；页面级/按用户切换实现 `FineUIPageManagerInitializer` bean（`pm.theme(...)`），详见 [references/set-theme.md](references/set-theme.md)。**主题 CSS 变量与 F.js 完全一致**（同一套 `themes/{主题}/theme.css`）。

## 参考文档

| 文件 | 何时读 |
|------|--------|
| [references/set-theme.md](references/set-theme.md) | 全局默认、PageManager 覆盖、运行时切换（Cookie + 刷新） |
| [references/custom-theme.md](references/custom-theme.md) | 自定义主题的文件、目录和接入入口 |

## 相关技能

- `fineui-foundation`：页面骨架（`_Layout` / PageManager 位置）
- [fineui-custom-theme](../fineui-custom-theme/SKILL.md)：创建、生成、验证和交付自定义主题

## 约束与规则

1. **主题名大小写**：C# 配置/枚举用 `Pure_Black`（帕斯卡 + 下划线）；F.js `F.init({ theme: 'pure_black' })` 用小写；**Java `application.properties` 用小写 `fineui.theme=pure_black`**（`themes/` 目录名即小写，大小写不敏感）。
2. **全局默认入口**：Pro 走 `Web.config` 的 `<FineUI.Pro Theme=".."/>`；Core 三套走 `appsettings.json` 的 `"FineUI":{"Theme":".."}`；**Java 走 `application.properties` 的 `fineui.theme=..`（`fineui.*` 键，kebab-case）**。
3. **没有 `F.setTheme` 这个公开方法**：示例常用**写 `Theme` Cookie → 刷新页面 → 服务端读 Cookie 设 PageManager**。应用需要区分内置和自定义主题；Java 示例当前的 Cookie 处理只调用 `pm.theme(...)`，使用自定义主题时需要扩展现有初始化器。详见 [references/set-theme.md](references/set-theme.md)。
4. **自定义主题不要手改 `theme.css`**：修改 `theme.config` 后，用应用 `res/themes` 自带的 `generate-theme.mjs` 生成。Pro、Core、Java 公开示例均提供生成器。Java 自定义主题通过 `pm.customTheme(...)` 引用，不能用 `fineui.theme` 或 `pm.theme(...)` 代替。
5. **深色主题**：`is-dark-background = true` 控制生成时的派生样式，不等同于客户端 `F.isDarkTheme`，不保证自动添加 `f-theme-darkbg` 类。

## 官方资源

- 在线文档与主题预览：https://www.fineui.com/
- 应用侧生成命令：在带生成器的 `res/themes` 目录执行 `node generate-theme.mjs 主题名`
