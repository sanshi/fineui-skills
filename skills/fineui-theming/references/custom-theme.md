# 自定义主题

创建主题使用 [fineui-custom-theme](../../fineui-custom-theme/SKILL.md)。该技能覆盖配色、生成、三栈接入、实际运行验证和下载交付；附带可复制的“暮紫薄荷”配置。

## 文件与生成入口

每个主题一个目录：`theme.config` 是手写配置，`theme.css` 是生成物，`theme-extra.css` 是可选的附加样式，生成时合并进 `theme.css`。

| 项目 | 主题目录 |
|------|----------|
| Pro / 纯 F.js 示例 | `res/themes/{主题名}/` |
| Core 三种模式 | `wwwroot/res/themes/{主题名}/` |
| Java | `src/main/resources/static/res/themes/{主题名}/` |

上述公开示例均提供 `res/themes/generate-theme.mjs`。进入带脚本的目录执行：

```powershell
node generate-theme.mjs my_theme
```

不传主题名会生成全部主题；增加 `--watch` 会持续监听配置和附加样式，保存后仍需刷新浏览器。不要删除生成器、监听用的批处理文件或说明文件。

## 接入入口

- Pro：`pm.CustomTheme = "my_theme"`，或 `Web.config` 的 `CustomTheme` 属性。
- Core：`pm.CustomTheme("my_theme")`，或 `appsettings.json` 中的 `FineUI.CustomTheme`。
- Java：在 `FineUIPageManagerInitializer` 中调用 `pm.customTheme("my_theme")`；`pm.theme(...)` 和 `fineui.theme` 用于内置主题。

完整代码、目录基址和 Cookie 覆盖顺序见 [自定义主题接入与验证](../../fineui-custom-theme/references/integration.md)。

## 配置注意事项

配置是每行 `key = value`，`#` 注释必须独占一行，不支持行尾注释。六组核心状态与圆角需要完整保留；不要手改生成的 `theme.css`。

`is-dark-background` 控制生成时的深色派生样式，不会自动设置客户端 `F.isDarkTheme`。`is-dark-active-color` 控制强调式或染色式选中态，两者独立。
