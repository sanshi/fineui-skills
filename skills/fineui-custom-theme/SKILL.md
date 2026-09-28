---
name: fineui-custom-theme
description: >
  为 FineUI 应用新增或修改自定义主题：确定配色，在应用 res/themes 中编写 theme.config，
  使用随示例提供的 generate-theme.mjs 生成 theme.css，接入 Pro、Core 或 Java，
  运行应用检查交互状态并按需交付可下载的主题包。适用于品牌配色、深色主题、
  自定义主题创建和主题生成；仅设置或切换已有主题时使用 fineui-theming。
metadata:
  author: FineUI
  version: "16.0.0-rc.3"
  compatibility: FineUI v16；主题变量机制自 v15 起提供
---

# 创建 FineUI 自定义主题

交付能够被真实应用加载的主题文件。先检查当前应用的静态资源目录、已有主题和生成器，再决定如何改配色；不要要求用户获取 FineUI 框架源码。

## 工作流程

1. **确定风格与范围。** 尊重用户给定的配色；没有指定时，对照已有主题提出一个明确方案，说明内容背景、正文、强调色、选中态和圆角。不要把本技能附带的示例配色当作固定要求。
2. **定位应用目录。** Pro 是 `res/themes/`；Core 是 `wwwroot/res/themes/`；Java 是 `src/main/resources/static/res/themes/`；纯 F.js 示例是 `res/themes/`。以项目实际目录为准。
3. **新建主题。** 名称用小写英文、数字和下划线或短横线，如 `plum_mint`。保留已有目录，复制本项目的 `custom_default/theme.config` 或合适的主题配置，再修改六组基础状态和派生项。完整示例见 [assets/plum_mint/theme.config](assets/plum_mint/theme.config)，可选原生深色外观见 [assets/plum_mint/theme-extra.css](assets/plum_mint/theme-extra.css)。
4. **生成样式。** 在应用的 `res/themes` 对应目录执行 `node generate-theme.mjs 主题名`。核对实际生成的文件；生成失败时保留原文件并检查路径和配置。需要边改边预览时用 `node generate-theme.mjs --watch 主题名`，或双击 `生成并监听主题.bat`。监听只负责生成，浏览器仍需刷新。
5. **接入应用。** 按 [references/integration.md](references/integration.md) 的栈差异设置自定义主题。优先使用现有主题初始化入口，避免新增重复的 PageManager 或初始化器。检查旧 `Theme` Cookie 是否覆盖配置。
6. **运行与验证。** 启动真实示例，确认主题样式请求成功、计算样式匹配配置；测试表格选中/取消、表单焦点与校验、下拉浮层、日期面板、主按钮、消息框。截图前等待页面渲染完成，按钮文字必须完整；有 iframe 时分别检查父页与子页。不能运行的项目如实说明，不能用静态拼图代替运行验证。
7. **交付。** 保留 `theme.config`、生成的 `theme.css`，以及存在时的 `theme-extra.css` 和图片。用户要求下载时，打包为单个主题目录并附中文安装说明；不要夹带框架 DLL/JAR、私有源码或整个示例工程。解压后再生成一次并比较样式，确认下载包可复用。

## 配置约束

- `theme.config` 使用 UTF-8，格式是每行 `key = value`；注释以 `#` 开头并独占一行。**不支持行尾注释**：`true # 深色` 会被当作字符串，颜色后追加说明也会污染输出。
- 基础配置完整保留 `content`、`header`、`default`、`hover`、`active`、`error` 六组，每组包含 `border-color`、`background-color`、`text-color`，另加 `border-radius`。
- 用十六进制颜色定义基础色，可让生成器计算深浅派生色；`primary-background-color`、`primary-text-color` 和 `tabstrip-inkbar-color` 应一起考虑。
- `is-dark-background` 控制生成器对深色背景的派生色、阴影等处理；它不是客户端 `F.isDarkTheme` 开关，不保证自动添加 `f-theme-darkbg` 类。
- `is-dark-active-color` 表示选中态是否发生明显的前景/背景对比切换：强调式选中用 `true`，浅色染底、文字基本不变用 `false`。不要仅按主题整体深浅判断。
- `theme.css` 是生成物，不手改。少量附加样式写在 `theme-extra.css`，生成器会并入同一份 `theme.css`；不需要再单独加载。组件颜色优先引用 `var(--f-*)`，避免用大范围选择器覆盖所有控件。
- **保留应用自带的 `generate-theme.mjs`、`生成并监听主题.bat`、`README.txt`。** Java 公开示例也提供生成器。不要将这些客户功能当作临时工具删除，也不要让客户运行仅私有框架仓库才有的构建命令。
- 检查发布是否包含新文件。Pro 旧式项目通常需要将主题文件登记为 `.csproj` 的 `Content`；Core、Java 检查对应静态资源输出。仅本地 HTTP 请求成功不能证明发布包完整。

## 相关技能

- [fineui-theming](../fineui-theming/SKILL.md)：已有主题的全局配置与切换。
