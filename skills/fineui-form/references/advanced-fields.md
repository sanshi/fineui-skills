# 高级表单字段（Advanced Fields）

FileUpload（文件上传）、TriggerBox（触发器输入框）、DropDownBox（下拉框容器）、HtmlEditor（富文本编辑器）。

---

## FileUpload（文件上传）

选择文件的输入框 + 浏览按钮。`Accept` 限制文件类型；`Multiple` 允许多文件；`ButtonOnly` 仅显示按钮。

> **服务端保存**：FileUpload 控件本身只负责选择文件，**保存逻辑在服务端**（Pro 用 `filePhoto.PostedFile`；Core 用 `IFormFile` 或 `FileUpload1.PostedFile`）。安全上传范式（白名单扩展名 + 时间戳重命名 + 不可直访目录）见 `fineui-foundation` 的 CLAUDE.md 或官方示例。

```javascript
// F.js
{ type: 'FileUpload', id: 'fu1', fieldLabel: '上传文件', buttonText: '选择文件', accept: 'image/*' }
// 多文件：multiple: true；仅按钮：buttonOnly: true
```
```aspx
<%-- Pro --%>
<f:FileUpload runat="server" ID="fu1" Label="上传头像" Accept="image/*" ButtonText="选择图片" />
```
```csharp
// Core-MVC（Fluent）
F.FileUpload().ID("fu1").Label("上传头像").Accept("image/*").ButtonText("选择图片")
```
```html
<!-- Core-TagHelper -->
<f:FileUpload ID="fu1" Label="上传头像" Accept="image/*" ButtonText="选择图片"></f:FileUpload>
```
```csharp
// Pro 服务端保存（Page_Load 或按钮事件）
if (fu1.PostedFile != null) {
    string ext = Path.GetExtension(fu1.PostedFile.FileName).ToLower();
    if (ext == ".jpg" || ext == ".png") {
        string newName = DateTime.Now.Ticks + ext;
        fu1.PostedFile.SaveAs(Server.MapPath("~/App_Data/upload/" + newName));
    }
}
```

---

## TriggerBox（触发器输入框）

带可点击图标按钮的文本框，是 DropDownList / DatePicker / NumberBox 的基类。常用于"点击图标弹出选择窗口"场景（如弹出树选择、日期范围）。

- `TriggerCls`：自定义触发图标样式类（如 `f-triggericon-search`）
- `TriggerClick` 事件：点击图标时触发（服务端或客户端）
- `EnableClickAction`：点击输入框本身是否也触发默认行为（TriggerBox 默认 `false`，DropDownBox/DatePicker 默认 `true`）

```javascript
// F.js —— triggerclick 事件打开弹窗
{ type: 'TriggerBox', id: 'tbx1', fieldLabel: '搜索', triggerCls: 'f-triggericon-search',
  listeners: { triggerclick: function () { F.ui.Window1.show(); } } }
```
```aspx
<%-- Pro --%>
<f:TriggerBox runat="server" ID="tbx1" Label="选择节点" TriggerCls="f-triggericon-search"
    OnTriggerClick="tbx1_TriggerClick" />
```
```csharp
// Core-MVC（Fluent）
F.TriggerBox().ID("tbx1").Label("选择节点").TriggerCls("f-triggericon-search")
    .OnTriggerClick(Url.Action("tbx1_TriggerClick"))
```
```html
<!-- Core-TagHelper（RazorForms）-->
<f:TriggerBox ID="tbx1" Label="选择节点" TriggerCls="f-triggericon-search" OnTriggerClick="tbx1_TriggerClick"></f:TriggerBox>
```
```csharp
// Pro / RazorForms 后台
protected void tbx1_TriggerClick(object sender, EventArgs e) {
    // 打开选择窗口等
}
```

---

## DropDownBox（下拉框容器）

输入框 + 下拉弹出面板，面板内可挂任意控件（Tree、Grid、CheckBoxList 等）。是"下拉树"、"下拉表格"、"多选下拉"的基础控件。

关键参数：
- `PopPanel`：弹出面板（内嵌 Tree / Grid / CheckBoxList 等）
- `DataControl`：数据控件 ID（多选时用于同步选中值）
- `MultiSelect`：多选模式；`MultiSelectMode="Tags"` 标签形态
- `MatchFieldWidth`：弹出面板宽度是否跟随输入框
- `MaxPopHeight`：弹出面板最大高度（默认 300）

### 下拉树（最常见）

```javascript
// F.js —— 先建隐藏的 Tree，再挂到 DropDownBox
F.create({ type: 'Tree', id: 'tree1', hidden: true, width: 300, height: 250, header: false,
    rootNode: { expanded: true, children: [
        { text: '河南省', id: 'henan', leaf: true },
        { text: '安徽省', id: 'anhui', leaf: true }
    ] }
});
F.create({ type: 'DropDownBox', id: 'ddb1', fieldLabel: '所属省份', popPanel: 'tree1', value: 'henan' });
// 读值：F.ui.ddb1.getValue()（节点 id）；F.ui.ddb1.getText()（显示文本）
```
```aspx
<%-- Pro --%>
<f:DropDownBox runat="server" ID="ddb1" Label="所属省份" MatchFieldWidth="false">
    <PopPanel>
        <f:Tree runat="server" ID="tree1" Width="300px" Height="250px" ShowHeader="false">
            <Nodes>
                <f:TreeNode Text="河南省" NodeID="henan" Leaf="true" />
                <f:TreeNode Text="安徽省" NodeID="anhui" Leaf="true" />
            </Nodes>
        </f:Tree>
    </PopPanel>
</f:DropDownBox>
```
```csharp
// Core-MVC（Fluent）
F.DropDownBox().ID("ddb1").Label("所属省份").MatchFieldWidth(false)
    .PopPanel(F.Tree().ID("tree1").Width(300).Height(250).ShowHeader(false)
        .Nodes(F.TreeNode().Text("河南省").NodeID("henan").Leaf(true),
               F.TreeNode().Text("安徽省").NodeID("anhui").Leaf(true)))
```
```html
<!-- Core-TagHelper -->
<f:DropDownBox ID="ddb1" Label="所属省份" MatchFieldWidth="false">
    <PopPanel>
        <f:Tree ID="tree1" Width="300" Height="250" ShowHeader="false">
            <Nodes>
                <f:TreeNode Text="河南省" NodeID="henan" Leaf="true" />
                <f:TreeNode Text="安徽省" NodeID="anhui" Leaf="true" />
            </Nodes>
        </f:Tree>
    </PopPanel>
</f:DropDownBox>
```
```csharp
// 服务端读值（Pro / RazorForms）
string nodeId = ddb1.Value;    // 选中节点 id
string text   = ddb1.Text;     // 显示文本
```

### 多选下拉（挂 CheckBoxList）

```javascript
// F.js —— multiSelect: true + dataControl 指向 CheckBoxList id
F.create({ type: 'DropDownBox', id: 'ddb2', fieldLabel: '编程语言',
    multiSelect: true, value: ['js', 'php'], dataControl: 'cbl1',
    popPanel: { type: 'Form', hidden: true, layout: 'anchor', bodyPadding: 10, header: false,
        items: [{ type: 'CheckBoxList', id: 'cbl1', columnNumber: 3,
            data: [['csharp','C#'], ['js','JavaScript'], ['java','JAVA'], ['php','PHP']] }] }
});
// 读值：F.ui.ddb2.getValue()  → ['js', 'php']
```

---

## HtmlEditor（富文本编辑器）

集成第三方富文本编辑器（CKEditor / UEditor / UMEditor / TinyMCE）。`Editor` 指定编辑器类型；`EditorBasePath` 指定编辑器资源路径。

> HtmlEditor 是**容器型字段**，需要先在页面引入对应编辑器的 JS/CSS，再创建控件。

```javascript
// F.js —— editor 指定编辑器类型
{ type: 'HtmlEditor', id: 'he1', fieldLabel: '内容', editor: 'ckeditor', height: 300,
  editorBasePath: '/ckeditor/' }
```
```aspx
<%-- Pro --%>
<f:HtmlEditor runat="server" ID="he1" Label="内容" Editor="CKEditor" Height="300px"
    EditorBasePath="~/ckeditor/" />
```
```csharp
// Core-MVC（Fluent）
F.HtmlEditor().ID("he1").Label("内容").Editor(HtmlEditorType.CKEditor).Height(300)
    .EditorBasePath("~/ckeditor/")
```
```html
<!-- Core-TagHelper -->
<f:HtmlEditor ID="he1" Label="内容" Editor="CKEditor" Height="300" EditorBasePath="~/ckeditor/"></f:HtmlEditor>
```
```csharp
// 读值（Pro / RazorForms）
string html = he1.Value;   // 返回 HTML 字符串
// 赋值
he1.Value = "<p>初始内容</p>";
```

> **安全提示**：HtmlEditor 的值是用户输入的 HTML，**不要**直接声明为 RawHtml 输出；如需展示，先做 HTML 净化（白名单过滤）。

## See also

- [fields.md](fields.md)：基础字段（TextBox / NumberBox / DatePicker / DropDownList 等）
- [validation.md](validation.md)：字段校验
- `fineui-tree`：DropDownBox 内嵌树
- `fineui-window`：TriggerBox 触发弹窗选择
