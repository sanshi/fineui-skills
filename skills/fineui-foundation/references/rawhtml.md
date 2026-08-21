# 可信 HTML（RawHtml）—— 让文本按 HTML 原样输出

## 为什么需要它（v15.2 安全模型）

v15.2 起，FineUI 控件的文本类属性（按钮/标签/菜单项文本、面板标题、列头、提示、消息框等）**默认一律 HTML 转义**。要让某段文本按 HTML **原样输出**（不转义），必须由开发者**显式声明其为“可信 HTML”**。

底层机制：值 + 类型标记。`new RawHtml(html)` 构造出的对象被视为可信；普通用户输入永远是 `string`，**伪造不出这个类型**，从根上防 XSS。

> **书写偏好（强约定）：优先用「单一便捷属性」声明可信 HTML**，命名规律是「属性名 + `RawHtml`」（`Text`→`TextRawHtml`、`Title`→`TitleRawHtml`、`ToolTip`→`ToolTipRawHtml`、`HeaderText`→`HeaderTextRawHtml`、消息 `Message`→`MessageRawHtml` …）。**不要**手写「值 + 独立布尔标记」两个属性。

## 各写法用法对照

以“菜单项/标签文本”与“消息框”为例。

### F.js —— `F.rawHtml('<...>')`

```javascript
{ type: 'MenuItem', text: F.rawHtml('<span class="hot">新</span> 报表') }
{ type: 'Tab', title: F.rawHtml('<span class="highlight">标签二</span>') }
{ type: 'Button', tooltip: F.rawHtml('第一行<br>第二行') }

// 消息 / 通知
showNotify(F.rawHtml('已选择：<b>' + name + '</b>'));
F.alert({ message: F.rawHtml(errors.join('<br/>')), title: '错误', messageIcon: 'error' });
```

### Pro（WebForms，aspx）—— 声明式 `XxxRawHtml="..."` / 后置 `new RawHtml(...)`

```aspx
<f:MenuHyperLink runat="server" TextRawHtml="<span class='hot'>新</span> 报表" NavigateUrl="..." />
<f:Tab runat="server" TitleRawHtml="<span class='highlight'>标签二</span>" />
<f:Button runat="server" Text="LeftStart" ToolTipRawHtml="第一行<br>第二行" ToolTipType="Qtip" />
```
```csharp
// 后置代码
node.TextRawHtml = new RawHtml("<span class=\"highlight\">{0}</span>", node.Text);
ShowNotify(new RawHtml("已选择：" + sb.ToString()));   // 消息框
```

### Core-MVC（Fluent API）—— `.XxxRawHtml(new RawHtml(...))`

```csharp
F.MenuHyperLink().TextRawHtml(new RawHtml("<span class='hot'>新</span> 报表"))
F.Tab().TitleRawHtml(new RawHtml("<span class='highlight'>标签二</span>"))
@(F.Button().Text("LeftStart").ToolTipRawHtml(new RawHtml("第一行<br>第二行")))

// Controller / 后置
ShowNotify(new RawHtml("已选择：" + sb.ToString()), MessageBoxIcon.None);
```

### Core-RazorForms / RazorPages（TagHelper）—— 下划线便捷 `_XxxRawHtml="..."`

```html
<%-- 字符串便捷形式（推荐）：直接写 HTML 字符串 --%>
<f:MenuHyperLink _TextRawHtml="<span class='hot'>新</span> 报表" NavigateUrl="..." />
<f:TreeNode _TextRawHtml="<span class='nodetitle'>驻马店市</span>，位于……" NodeID="zmd" />
<f:RenderField _HeaderTextRawHtml="姓名<br>（换行）" DataField="Name" />

<%-- 含 Razor 变量时用表达式形式 --%>
<f:RenderField _HeaderTextRawHtml="@("入学年份<i class=\"f-icon custom-filter\"></i>")" DataField="EntranceYear" />
<f:CheckBox _SwitchOnTextRawHtml="@ViewBag.OnText" _SwitchOffTextRawHtml="@ViewBag.OffText" />
```
```csharp
// 后置代码（.cshtml.cs）——与 Fluent/Pro 相同，用 new RawHtml(...)
ShowNotify(new RawHtml("已选择：" + sb.ToString()), MessageBoxIcon.None);
notify.MessageRawHtml = new RawHtml("<div class=\"box\">...</div>");
CheckBox5.SwitchOnTextRawHtml = new RawHtml("<i class=\"f-icon check\"></i>");
```

### FineUIJava（Thymeleaf 方言）—— 单属性便捷 `xxx-raw-html="..."`（直接写 HTML，无下划线前缀）

```html
<!-- markup 里直接写 HTML 字符串（不像 Core 需要下划线前缀） -->
<f:tree-node text-raw-html="<span class='nodetitle'>驻马店市</span>，位于……" node-id="zmd" />
<f:render-field header-text-raw-html="入学年份<i class='f-icon custom-filter'></i>" data-field="EntranceYear" />
<f:grid empty-text-raw-html="<div class='grid-empty-text'>没有数据</div>"></f:grid>
```
```java
// 页面类（.java）——用 new RawHtml(...)，setter 形式；支持 %s/%d 格式化参数（同 C#）
import com.fineui.java.core.RawHtml;

labResult.setTextRawHtml(new RawHtml("<span class=\"highlight\">%s</span>", node.getText()));
CheckBox5.setSwitchOnTextRawHtml(new RawHtml("<i class=\"f-icon f-iconfont f-iconfont-check\"></i>"));
showNotifyRaw(new RawHtml("已选择：<b>%s</b>", name));   // ← 消息框用 showNotifyRaw（区别于 Core 的 ShowNotify(new RawHtml(...))）
```

## 概念 → 各写法对照

| | F.js | Pro (aspx) | Core-MVC (Fluent) | Core-TagHelper (标签) | Java (Thymeleaf) |
|--|------|-----------|-------------------|------------------------|------------------|
| 声明可信 HTML | `F.rawHtml('...')` | `XxxRawHtml="..."` | `.XxxRawHtml(new RawHtml("..."))` | `_XxxRawHtml="..."` 或 `XxxRawHtml="@(new RawHtml("..."))"` | `xxx-raw-html="..."`（直接写 HTML） |
| 后置代码赋值 | — | `x.XxxRawHtml = new RawHtml(...)` | `x.XxxRawHtml = new RawHtml(...)` | `x.XxxRawHtml = new RawHtml(...)` | `x.setXxxRawHtml(new RawHtml(...))` |
| 消息框 | `showNotify(F.rawHtml(...))` / `F.alert({message: F.rawHtml(...)})` | `ShowNotify(new RawHtml(...))` | `ShowNotify(new RawHtml(...))` | `ShowNotify(new RawHtml(...))` | `showNotifyRaw(new RawHtml(...))` |

## 关键约束

1. **优先单属性便捷写法**，不要写「`Text` + `TextRaw` 两个属性」。命名一律 `Xxx` → `XxxRawHtml`。
2. **只对开发者确信可信的 HTML 用它**；来自**用户输入 / 数据库**的内容**不要**声明为 RawHtml，让其保持默认转义。可信内容里若拼接了不可信片段，先 `HttpUtility.HtmlEncode(...)`（C#）再拼。
3. **消息框用 `ShowNotify(new RawHtml(...))`**；`Alert.Show(new RawHtml(...))` 虽存在但示例中不用它。F.js 用 `F.alert({ message: F.rawHtml(...) })` 对象配置形式，**不是** `F.alert(F.rawHtml(...))` 直传。
4. **Core TagHelper 便捷形式**：markup 里直接写 HTML 字符串用 `_XxxRawHtml="..."`（下划线前缀）；含 Razor 变量/表达式用 `_XxxRawHtml="@(...)"` 或 `XxxRawHtml="@(new RawHtml(...))"`。**Java 便捷形式**：`xxx-raw-html="..."`（kebab-case，直接写 HTML 字符串，**无**下划线前缀）；服务端 `x.setXxxRawHtml(new RawHtml(...))`、消息框 `showNotifyRaw(new RawHtml(...))`。
5. **（Pro 大坑）`XxxRawHtml`（RawHtml 类型）便捷属性绝不能加 `[Browsable(false)]` 或 `[DesignerSerializationVisibility(Hidden)]`**——否则 aspx 里 `TextRawHtml="..."` 声明式用法会直接抛分析器错误、页面打不开。（这是控件开发者约束；使用者只要按上面写法即可。）
6. **旧的内联 `<raw>...</raw>` 写法**仍兼容但**不推荐**（可被全局开关 `AllowDangerousRawTag=false` 关闭），一律改用上面的声明式写法。

## See also

- [stacks.md](stacks.md)：各写法总览与命名规律
- `fineui-grid` 技能：列头/单元格的可信 HTML（`RendererFunction` 返回的 HTML 需自行保证可信）
