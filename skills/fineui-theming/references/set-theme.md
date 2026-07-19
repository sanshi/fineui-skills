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

> `IsSystemTheme(name)`：`Enum.GetNames(typeof(Theme))` 里（忽略大小写）匹配到即内置主题，走 `pm.Theme`；否则当自定义主题名走 `pm.CustomTheme`。

## See also

- [custom-theme.md](custom-theme.md)：自定义主题
