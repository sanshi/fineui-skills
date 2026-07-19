---
name: fineui-theming
description: >
  帮助开发者使用 FineUI 的主题系统（v15 起基于 CSS Variables）：设置全局主题、运行时切换主题、
  自定义主题（theme.config + generate-theme）。覆盖 F.js（JavaScript）、Pro（WebForms）、
  FineUICore 的 MVC / RazorForms / RazorPages。
  Trigger phrases（触发词）: "FineUI 主题", "换主题", "切换主题", "Theme", "Pure_Black",
  "深色主题", "dark theme", "自定义主题", "theme.config", "generate-theme", "CSS Variables 主题",
  "PageManager Theme", "CustomTheme".
compatibility: FineUI v15+（主题系统重构为 CSS Variables）
metadata:
  author: FineUI
  version: "15.2"
---

# FineUI 主题技能（Theming）

> v15 起主题系统从 SCSS 预编译重构为 **CSS Variables**：运行时只需加载 `themes/{主题}/theme.css` 覆盖颜色变量，无需 sass。

## 何时使用（When to Use）

- 设置站点全局主题
- 让用户运行时切换主题（下拉/图墙）
- 制作自定义主题（改配色）

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
<!-- ② Pro（WebForms）—— Web.config 的 <FineUIPro> 段 -->
<FineUIPro DebugMode="false" Theme="Pure_Black" EnableAnimation="true" />
```
```json
// ③ Core（MVC / RazorForms / RazorPages 三套一致）—— appsettings.json
"FineUI": { "Theme": "Pure_Black", "EnableAnimation": true }
```

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/set-theme.md](references/set-theme.md) | 全局默认、PageManager 覆盖、运行时切换（Cookie + 刷新） |
| [references/custom-theme.md](references/custom-theme.md) | 自定义主题：theme.config + generate-theme 生成 |

## 相关技能（Related Skills）

- `fineui-foundation`：页面骨架（`_Layout` / PageManager 位置）

## 约束与规则（Constraints & Rules）

1. **主题名大小写**：配置/枚举用 `Pure_Black`（帕斯卡 + 下划线）；F.js `F.init({ theme: 'pure_black' })` 用小写。
2. **全局默认入口**：Pro 走 `Web.config` 的 `<FineUIPro Theme=".."/>`；Core 三套走 `appsettings.json` 的 `"FineUI":{"Theme":".."}`。
3. **运行时切换没有纯客户端 `setTheme`**：仓库统一做法是**写 `Theme` Cookie → 刷新页面 → 服务端读 Cookie 设 PageManager**。详见 [references/set-theme.md](references/set-theme.md)。
4. **自定义主题不要手改 `theme.css`**：它由 `theme.config` 经 `generate-theme` 自动生成；改配色改 `theme.config` 再重新生成。详见 [references/custom-theme.md](references/custom-theme.md)。
5. **深色主题**：`theme.config` 里 `is-dark-background = true`；框架会给 body 加 `f-theme-darkbg` 类。

## 官方资源（Official Resources）

- 在线文档与主题预览：https://www.fineui.com/
- 生成主题脚本：`npm run theme-gen` / `F\generate-theme.bat`
