# 富文本编辑器

适用于 FineUI.Pro、Core 三种模式和 Java 的 `HtmlEditor`，F.js 是共同客户端实现。v16.2.0 默认引擎为 `Editor.Default`（JS 为 `editor: 'default'`），编辑内核使用 ProseMirror，工具栏和编辑区域跟随 FineUI 主题。

## 创建与取值

```html
<!-- Core RazorForms/RazorPages；Pro 增加 runat="server"。 -->
<f:HtmlEditor ID="HtmlEditor1" Label="正文" Editor="Default"
    ToolbarSet="Full" Height="320" Text="&lt;p&gt;编辑正文&lt;/p&gt;" />
```

```csharp
// Core MVC 创建控件，枚举名称是 Editor。
F.HtmlEditor().ID("HtmlEditor1").Label("正文").Editor(Editor.Default)
    .ToolbarSet(EditorToolbarSet.Full).Height(320).Text("<p>编辑正文</p>")
```

```html
<f:html-editor id="HtmlEditor1" label="正文" editor="Default"
    toolbar-set="Full" height="320" text="&lt;p&gt;编辑正文&lt;/p&gt;"></f:html-editor>
```

服务端通过 `HtmlEditor1.Text`（Pro/Core）或 `HtmlEditor1.getText()/setText(...)`（Java）读写 HTML。不要把客户端的 `value` 配置名直接写成 C# 的 `Value` 属性。客户端使用 `F('HtmlEditor1').getValue()/setValue(...)`，切换只读用 `setReadonly(true)`。

## 资源与工具栏

服务端默认按需加载 `FineUI.HtmlEditor.css` 和 `FineUI.HtmlEditor.js`；将 `IncludeEditorScripts` 设为 false 时由页面自行引入。纯 JS 在创建前引入这两个文件。

`ToolbarSet` 默认为 `Basic`；`Full` 增加字体、字号、颜色、对齐、引用和分隔线。编辑器引擎、工具栏、资源地址和选项在创建时确定，不把后台赋值当作现有实例的重建操作。已有实例支持更新正文及只读状态。

```javascript
F.create({
    type: 'HtmlEditor',
    id: 'HtmlEditor1',
    renderTo: document.body,
    fieldLabel: '正文',
    height: 320,
    editor: 'default',
    editorOptions: {
        toolbar: ['bold', 'italic', 'underline', '|', 'bulletList', 'link', 'source']
    }
});
```

Pro/Core 用 `Options` 中的 `toolbar` 数组提供同样配置，Java 使用 `setEditorOptions(Map.of("toolbar", List.of(...)))`。显式数组替换预设，`|` 分隔按钮组。`source` 切换原位 HTML 源码，放在数组末尾。内置选项不能当作旧第三方编辑器的选项传递。

## 内容边界与旧引擎

内置编辑器按支持的文档结构规范化 HTML，例如 `<b>` 转为 `<strong>`，不支持的图片、表格结构会被移除。需要这些内容或旧插件的页面继续选择对应旧引擎，并保留原来的 `BasePath`、`Options` 和资源配置。

```html
<!-- Core；Pro 增加 runat="server"。 -->
<f:HtmlEditor ID="HtmlEditor1" Editor="UMEditor"
    BasePath="~/res/third-party/umeditor/" Height="320" />
```

```html
<f:html-editor id="HtmlEditor1" editor="UMEditor"
    base-path="/res/third-party/umeditor/" height="320"></f:html-editor>
```

纯 JS 使用 `editor: 'umeditor'` 和 `editorBasePath`。`getEditorInstance()` 在旧引擎下返回第三方实例，在内置模式返回 FineUI 适配器，不能直接调用旧引擎专用方法。内置编辑器可用于严格内容安全策略（Content Security Policy，CSP）的页面；旧引擎兼容页的策略按其依赖核对。

编辑器内容是用户输入的 HTML。展示到其它页面前仍应由业务服务端按允许的结构和链接规则清洗，不能仅因来自编辑器就声明为可信 HTML。

[Core 完整工具栏](https://fineui.com/core/demo/#/HtmlEditor/FullToolbar) · [Java 基础编辑](https://fineui.com/java/demo/#/html-editor/basic) · [Pro 基础编辑](https://fineui.com/pro/demo/#/htmleditor/basic.aspx)
