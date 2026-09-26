# 设置与切换主题

## 全局默认主题

```javascript
// 纯 F.js 应用在已有初始化参数中设置内置主题。
F.init({ theme: 'pure_black' });
```

```xml
<!-- Pro：修改 Web.config 中已有的 FineUI.Pro 元素。 -->
<FineUI.Pro Theme="Pure_Black" CustomTheme="" CustomThemeBasePath="~/res/themes/" />
```

```json
{
  "FineUI": {
    "Theme": "Pure_Black",
    "CustomTheme": ""
  }
}
```

Core 三种模式都使用上面的 `appsettings.json` 配置，合并到现有 `FineUI` 对象。

```properties
# Java：application.properties 中设置内置主题。
fineui.theme=pure_black
```

## 页面级覆盖

在项目已有的初始化位置，选用对应栈的写法：

```csharp
// Pro：恢复内置主题时，先清空自定义主题。
pm.CustomTheme = String.Empty;
pm.Theme = Theme.Pure_Blue;
```

```csharp
// Core：视图中的 F 来自 Html.F()。
var pm = F.PageManager;
pm.CustomTheme(String.Empty);
pm.Theme(Theme.Pure_Blue);
```

```java
// Java：在已有 FineUIPageManagerInitializer 的 init 方法内设置，早于 head 渲染。
pm.customTheme(null);
pm.theme("pure_blue");
```

应用静态资源中的自定义主题分别使用 `pm.CustomTheme = "my_theme"`、`pm.CustomTheme("my_theme")`、`pm.customTheme("my_theme")`。Java 的 `theme(...)` 与 `customTheme(...)` 也需要区分，不能混用。

## 运行时切换

官方示例常用“保存 Cookie → 刷新 → 服务端初始化 PageManager”的方式；没有 `F.setTheme` 这个公开方法。

```javascript
// Cookie 只是保存偏好，下一次请求仍需由应用代码读取并应用。
F.cookie('Theme', 'pure_blue', { expires: 100, path: '/' });
window.location.reload();
```

- Pro 示例的 `PageBase` 判断名称是否属于 `Theme` 枚举，内置名设置 `Theme` 并清空 `CustomTheme`，其它名设置 `CustomTheme`。
- Core 示例的 `_InitPageManagerPartial.cshtml` 使用相同的区分逻辑，采用流式调用。
- Java 示例的 `AppPageManagerInitializer` 目前读取 Cookie 后调用 `pm.theme(...)`，不能直接用这段逻辑加载应用自定义主题。需要扩展现有初始化器，以允许的自定义主题名清单区分两类入口。
- 纯 F.js 应用需要自己读取偏好，在加载样式和初始化时使用；写 Cookie 本身不会换肤。

已有 Cookie 可能覆盖全局默认。排查时清除偏好后刷新：

```javascript
F.cookie('Theme', null, { path: '/' });
window.location.reload();
```

新应用处理外部传入的主题名时使用允许列表，不把任意 Cookie 值直接拼进资源路径。

## 自定义主题

创建文件与运行验证见 [fineui-custom-theme](../../fineui-custom-theme/SKILL.md)，完整接入代码见 [自定义主题接入与验证](../../fineui-custom-theme/references/integration.md)。
