# 设置与切换主题

## 一、全局默认主题

```javascript
// F.js —— F.init（小写主题名）
F.init({ theme: 'pure_black' });
```
```xml
<!-- Pro —— Web.config 的 <FineUIPro> 段 -->
<FineUIPro DebugMode="false" Theme="Pure_Black" EnableAnimation="true"
           CustomTheme="" CustomThemeBasePath="~/res/themes/" />
```
```json
// Core（MVC / RazorForms / RazorPages）—— appsettings.json
{
  "FineUI": { "DebugMode": false, "Theme": "Pure_Black", "EnableAnimation": true }
}
```
```properties
# FineUIJava（Spring Boot）—— application.properties（fineui.* 全站默认，主题名小写）
fineui.theme=pure_black
fineui.enable-animation=true
fineui.custom-scrollbar=true
```

### 全局配置入口对照

| 部署栈 | 全局默认入口 | 页面级/按用户 |
|--------|-------------|--------------|
| F.js | `F.init({ theme:'pure_black' })` | 同上（前端） |
| Pro | `Web.config` `<FineUIPro Theme="Pure_Black" .../>` | 页面/基类 `pm.Theme = ...` |
| Core（三模式） | `appsettings.json` 的 `"FineUI":{ "Theme":"Pure_Black" }` | `_InitPageManagerPartial.cshtml` 里 `pm.Theme(...)` |
| **Java（Spring Boot）** | **`application.properties` 的 `fineui.theme=pure_black`（`fineui.*` 键）** | **`FineUIPageManagerInitializer` bean 的 `init(pm, request)` 里 `pm.theme(...)`** |

## 二、PageManager 覆盖（页面级 / 动态）

```csharp
// Pro（WebForms）—— C# 属性赋值
pm.Theme = Theme.Pure_Blue;          // 内置主题
pm.CustomTheme = String.Empty;
// 自定义主题：pm.CustomTheme = "my_theme";
```
```csharp
// Core 三模式 —— PageManager 流式（F = Html.F()），通常写在 _Layout / _InitPageManagerPartial
var pm = F.PageManager;
pm.CustomTheme(String.Empty);
pm.Theme(Theme.Pure_Blue);           // 内置主题
// 自定义主题：pm.CustomTheme("my_theme");
```
```java
// FineUIJava —— 实现 FineUIPageManagerInitializer 的 @Component，渲染前回调、可读 request/cookie
// 注意：Java 不分 Theme / CustomTheme——pm.theme(名) 对内置主题和自定义主题名统一处理
@Component
public class AppPageManagerInitializer implements FineUIPageManagerInitializer {
    @Override
    public void init(PageManager pm, HttpServletRequest request) {
        pm.theme("pure_blue");        // 内置或自定义主题名都走同一个方法
        // pm.language("zh_CN"); pm.displayMode("normal");
    }
}
```

## 三、运行时切换主题（Cookie + 刷新）

**没有纯客户端 `F.setTheme`。** 统一模式：**用户选主题 → 写 `Theme` Cookie → 刷新页面 → 服务端读 Cookie 设 PageManager**。

### 客户端：写 Cookie 并刷新

```javascript
// 用户点击主题项（图墙/下拉）后
F.cookie('Theme', 'Pure_Blue', { expires: 100 });   // expires 单位：天
top.window.location.reload();
```

### 服务端：读 Cookie 设置主题

```csharp
// Pro —— PageBase（页面基类 OnInit 里）
HttpCookie themeCookie = Request.Cookies["Theme"];
if (themeCookie != null) {
    string v = themeCookie.Value;
    if (IsSystemTheme(v)) { pm.CustomTheme = String.Empty; pm.Theme = (Theme)Enum.Parse(typeof(Theme), v, true); }
    else                  { pm.CustomTheme = v; }   // 自定义主题名
}
// IsSystemTheme：用 Enum.GetNames(typeof(Theme)) 判断是否内置主题
```
```csharp
// Core —— _InitPageManagerPartial.cshtml（三套一致）
var pm = F.PageManager;
var themeCookie = Context.Request.Cookies["Theme"];
if (!String.IsNullOrEmpty(themeCookie)) {
    if (IsSystemTheme(themeCookie)) { pm.CustomTheme(String.Empty); pm.Theme((Theme)Enum.Parse(typeof(Theme), themeCookie, true)); }
    else                            { pm.CustomTheme(themeCookie); }
}
```
```java
// FineUIJava —— FineUIPageManagerInitializer bean（渲染前回调，读 cookie 设主题）
@Component
public class AppPageManagerInitializer implements FineUIPageManagerInitializer {
    @Override
    public void init(PageManager pm, HttpServletRequest request) {
        String theme = cookie(request, "Theme");   // Cookie 名：Theme（另有 Language / DisplayMode）
        if (theme != null && !theme.isEmpty()) {
            pm.theme(theme);   // 无需区分内置/自定义——内置名（如 Pure_Blue）与自定义名（如 image_blue_sky）都传给 pm.theme
        }
    }
    // cookie(request, name)：遍历 request.getCookies() 取值
}
```

> **Java 比 Core 简单**：不需要 `IsSystemTheme` 判断、不分 `pm.Theme` / `pm.CustomTheme`——内置主题名和自定义主题名都直接传给 `pm.theme(名)`，框架按 `themes/{名}/theme.css` 解析。客户端写 Cookie + 刷新的那段 JS（`F.cookie('Theme', ...)` + `top.window.location.reload()`）四栈完全相同，见上。

## See also

- [custom-theme.md](custom-theme.md)：自定义主题
