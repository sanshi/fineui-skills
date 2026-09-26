# 自定义主题接入与验证

以下用 `plum_mint` 举例。主题目录必须包含生成后的 `theme.css`；单有 `theme.config` 不会改变页面。目录名与设置值保持一致。

## FineUI.Pro

资源放在 Web 项目的 `res/themes/plum_mint/`。在现有 `Web.config` 的 `FineUI.Pro` 元素上增加属性，保留原有属性，不要另建第二个元素：

```xml
<FineUI.Pro CustomTheme="plum_mint" CustomThemeBasePath="~/res/themes/" />
```

页面级或用户偏好设置采用已有 PageManager：

```csharp
pm.CustomTheme = "plum_mint";
```

官方 Pro 示例的 `PageBase` 读取 `Theme` Cookie：内置名设置 `Theme` 并清空 `CustomTheme`，其它名设置 `CustomTheme`。因此调试时可在已加载 FineUI 的示例页控制台运行：

```javascript
F.cookie('Theme', 'plum_mint', { expires: 30, path: '/' });
window.location.reload();
```

这是**该示例页面基类实现的约定**，不是所有 FineUI 应用都自动读取 Cookie。使用应用默认值时，先清除旧偏好：

```javascript
F.cookie('Theme', null, { path: '/' });
window.location.reload();
```

用 Visual Studio 的 MSBuild 还原和构建，通过 IIS Express 启动；Pro 是 .NET Framework WebForms 项目，不能用 `dotnet run`。浏览器应请求 `/res/themes/plum_mint/theme.css`（可能带版本参数）。

Pro 项目若显式列出内容文件，需要登记新增的三个文件；没有附加样式时省略对应项：

```xml
<Content Include="res\themes\plum_mint\theme.config" />
<Content Include="res\themes\plum_mint\theme-extra.css" />
<Content Include="res\themes\plum_mint\theme.css" />
```

## FineUI.Core

三种开发模式的资源都放在 `wwwroot/res/themes/plum_mint/`。在现有 `appsettings.json` 的 `FineUI` 对象中合并：

```json
{
  "FineUI": {
    "CustomTheme": "plum_mint",
    "CustomThemeBasePath": "~/res/themes/"
  }
}
```

或者在布局中已有的 PageManager 初始化位置使用流式写法：

```csharp
var pm = F.PageManager;
pm.CustomTheme("plum_mint");
```

这里的 `F` 是视图中已有的 `Html.F()`。官方示例的 `_InitPageManagerPartial.cshtml` 可能读取 `Theme` Cookie 覆盖默认值，排查时一并检查。

## FineUI.Java

资源放在 `src/main/resources/static/res/themes/plum_mint/`；该目录的上一级 `res/themes/` 自带 `generate-theme.mjs`。

在已有 `FineUIPageManagerInitializer` 中合并设置；应用尚未提供初始化器时可以创建：

```java
import com.fineui.java.core.PageManager;
import com.fineui.java.web.FineUIPageManagerInitializer;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.stereotype.Component;

@Component
public class AppPageManagerInitializer implements FineUIPageManagerInitializer {
    @Override
    public void init(PageManager pm, HttpServletRequest request) {
        // 在 head 渲染前选择应用静态资源里的自定义主题。
        pm.customTheme("plum_mint");
        pm.customThemeBasePath("/res/themes");
    }
}
```

`pm.theme("pure_blue")` 和 `fineui.theme=pure_blue` 用于**内置主题**。应用自带的主题走 `pm.customTheme(...)`，不要把它写成 `fineui.theme=plum_mint`。恢复内置主题时先 `pm.customTheme(null)`，再 `pm.theme("pure_blue")`。

不要笼统声称 Java 与 Pro 的 `Theme` Cookie 处理相同：检查项目初始化器，只有它明确区分内置和自定义主题时，Cookie 才能用于自定义主题切换。固定使用本主题时，将设置放在已有 Cookie 处理之后，或者按应用需求增加明确的主题名白名单。

## 纯 F.js 应用

先加载项目已有的 FineUI 基础样式，再加载应用的自定义主题：

```html
<link rel="stylesheet" href="res/themes/plum_mint/theme.css">
```

在原有 `F.init` 参数中合并以下设置，避免按主题名再去内置目录加载：

```javascript
F.init({
    theme: 'plum_mint',
    addThemeTag: false,
    isDarkTheme: true
});
```

`isDarkTheme` 只在本例这种深色主题设为 `true`；浅色主题不照抄。服务端栈的主题生成开关与客户端深色标志各有职责，不能据此假设服务端会自动下发这个值。

## 验证与常见问题

| 现象 | 检查位置 |
|------|----------|
| 命令提示未找到配置 | 当前路径是否为带生成器的 `res/themes`，主题目录名是否拼对 |
| 生成成功但颜色未变 | 页面是否请求了该主题的 `theme.css`；Cookie、PageManager 是否覆盖全局值；是否刷新了缓存 |
| 样式请求返回 404 | 目录是否放在当前 Web 项目的静态资源根；发布文件是否包含新主题 |
| 只有外框换色 | iframe 中的页面是否也应用同一主题；子页面是否重新设置 PageManager |
| 某个控件仍是固定颜色 | 页面是否存在硬编码样式或特定语义色，不要为了覆盖它而重写全部组件样式 |
| 深色配置不生效 | 布尔值是否严格为 `true` / `false`；值后是否误加行尾注释 |

浏览器检查以样式请求和计算值为准。例如查看 `--f-content-background-color`，再查看表格选中行、输入框和浮层的实际颜色。检查文字、勾选标记与焦点轮廓是否清晰；测试一次真实服务端回发，确认操作后的状态仍正确。
