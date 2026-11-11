# 高级表单字段（Advanced Fields）

FileUpload（文件上传）、TriggerBox（触发器输入框）、DropDownBox（下拉框容器）、HtmlEditor（富文本编辑器）。

---

## FileUpload（文件上传）

三栈服务端使用 `AcceptFileTypes` 限制选择器中的扩展名，例如 `.jpg,.jpeg,.png`；F.js 对应配置是 `accept`。`Multiple` 允许多文件，`ButtonOnly` 仅显示按钮。浏览器中的类型限制只帮助选择文件，服务端仍需校验扩展名、大小和实际内容。

v16.2.0 在创建时设置 `EnableDragDrop=true`，即可拖入文件或点击整个区域选择。每次选择替换该字段原来的待提交文件，普通提交模式可逐项移除。这个开关默认为 false，已有实例不支持在回发中切换。

```html
<!-- Core RazorForms 或 RazorPages；Pro 另加 runat="server"。 -->
<f:FileUpload ID="filePhotos" Label="图片" Multiple="true"
    EnableDragDrop="true" AcceptFileTypes=".jpg,.jpeg,.png" />
```

```csharp
// Core MVC 创建控件。
F.FileUpload().ID("filePhotos").Label("图片").Multiple(true)
    .EnableDragDrop(true).AcceptFileTypes(".jpg,.jpeg,.png")
```

```html
<!-- Java 直接使用对应的短横线属性。 -->
<f:file-upload id="filePhotos" label="图片" multiple="true"
    enable-drag-drop="true" accept-file-types=".jpg,.jpeg,.png"></f:file-upload>
```

```javascript
F.create({
    type: 'FileUpload',
    id: 'filePhotos',
    renderTo: document.body,
    fieldLabel: '图片',
    multiple: true,
    enableDragDrop: true,
    accept: '.jpg,.jpeg,.png'
});
```

Pro、Core RazorForms 和 Java 可声明 `FileSelected` 后台事件完成选定即上传。Core MVC/RazorPages 按对应示例的事件地址接线。等待期间显示整批数量，校验失败时保留本次选择并显示原因；只有成功接收后再 `Reset()/reset()`，让用户重试失败文件。

服务端文件来自 Pro 的 `PostedFile/PostedFiles`、Core 的 `PostedFile/PostedFiles`（ASP.NET Core 上传文件对象）和 Java 的 `getPostedFile()/getPostedFiles()`。上传控件不自动保存业务文件。校验后使用服务器生成的存储名，保存到不可直接访问的目录；不能直接用用户文件名拼接保存路径。

[拖放后提交示例](https://fineui.com/core/demo/#/Form/FileUploadDragDrop) · [选定即上传示例](https://fineui.com/core/demo/#/Form/FileUploadDragDropAuto)

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
```html
<!-- FineUI.Java（Thymeleaf 方言）：触发图标用 trigger-icon="Search"（不是 TriggerCls）；示例用客户端 triggerclick 监听打开窗口 -->
<f:trigger-box id="TriggerBox1" show-label="false" trigger-icon="Search" empty-text="打开弹出窗口" enable-click-action="true">
    <f:listeners>
        <f:listener event="triggerclick" handler="onTriggerBox1TriggerClick"></f:listener>
    </f:listeners>
</f:trigger-box>
<!-- 脚本槽：function onTriggerBox1TriggerClick(event) { F.ui.Window1.show(); }（同 F.js）-->
```
```csharp
// Pro / RazorForms 后台
protected void tbx1_TriggerClick(object sender, EventArgs e) {
    // 打开选择窗口等
}
```
```java
// FineUI.Java 页面类：设值 setValue(...)（弹窗选完后回填）
public void btnCloseWindow_Click(Object sender, EventArgs e) {
    Window1.setHidden(true);
    TriggerBox1.setValue("弹出窗口被关闭了");
}
```

---

## DropDownBox（下拉框容器）

同时清空选中值、文字、标签和弹出选择状态使用 `ClearSelection()` / `clearSelection()`；后台读取与 UIHelper 的区别见[统一清空选择](clear-selection.md)。

输入框 + 下拉弹出面板，面板内可挂任意控件（Tree、Grid、CheckBoxList 等）。是"下拉树"、"下拉表格"、"多选下拉"的基础控件。

关键参数：
- `PopPanel`：弹出面板（内嵌 Tree / Grid / CheckBoxList 等）
- `DataControl`：数据控件 ID（多选时用于同步选中值）
- `MultiSelect`：多选模式；`MultiSelectMode="Tags"` 标签形态
- `MatchFieldWidth`：弹出面板宽度是否跟随输入框
- `MaxPopHeight`：弹出面板最大高度（默认 300）

> **FineUI.Java 属性名对照**（kebab-case，个别语义有差异）：`PopPanel`→`<f:pop-panel>`、`MultiSelect`→`enable-multi-select`、`DataControl`→`data-control-id`、单选初值 `Value`→`value`、多选初值 → `values`（逗号分隔）。

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
```html
<!-- FineUI.Java（Thymeleaf 方言）：弹出面板标签是 <f:pop-panel>；单选 enable-multi-select="false" -->
<f:drop-down-box id="DropDownBox1" value="henan" enable-multi-select="false">
    <f:pop-panel>
        <f:tree id="Tree1" show-header="false" hidden="true">
            <f:nodes>
                <f:tree-node text="河南省" node-id="henan" expanded="true"></f:tree-node>
                <f:tree-node text="安徽省" node-id="anhui"></f:tree-node>
            </f:nodes>
        </f:tree>
    </f:pop-panel>
</f:drop-down-box>
```
```csharp
// 服务端读值（Pro / RazorForms）
string nodeId = ddb1.Value;    // 选中节点 id
string text   = ddb1.Text;     // 显示文本
```
```java
// FineUI.Java 页面类：单选读值 getValue()/getText()
String nodeId = DropDownBox1.getValue();
String text   = DropDownBox1.getText();
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
```html
<!-- FineUI.Java（Thymeleaf 方言）：多选 enable-multi-select="true"；data-control-id 指向面板内的 CheckBoxList；初值 values="js,php" -->
<f:drop-down-box id="DropDownBox1" data-control-id="CheckBoxList1" enable-multi-select="true" values="js,php">
    <f:pop-panel>
        <f:simple-form id="SimpleForm2" body-padding="10px" auto-scroll="true" show-header="false" hidden="true">
            <f:items>
                <f:check-box-list id="CheckBoxList1" column-number="3">
                    <f:check-item text="C#" value="csharp"></f:check-item>
                    <f:check-item text="JavaScript" value="js"></f:check-item>
                    <f:check-item text="PHP" value="php"></f:check-item>
                </f:check-box-list>
            </f:items>
        </f:simple-form>
    </f:pop-panel>
</f:drop-down-box>
```
```java
// FineUI.Java 页面类：多选读值 getValues()（List）；后台更新需同时设文本与值
java.util.List<String> vals = DropDownBox1.getValues();          // 如 ["js", "php"]
DropDownBox1.setTexts(java.util.Arrays.asList("PHP", "Basic"));
DropDownBox1.setValues(java.util.Arrays.asList("php", "basic"));
```

---

## HtmlEditor（富文本编辑器）

v16.2.0 默认采用基于 ProseMirror 的内置编辑器，支持主题、工具栏配置及原位 HTML 源码编辑。Pro/Core 使用 `Text`、Java 使用 `getText()/setText()` 读写 HTML，F.js 使用 `getValue()/setValue()`。服务端的旧编辑器路径属性为 `BasePath`，类型枚举为 `Editor`。

创建配置、资源加载和旧引擎兼容见 [富文本编辑器](html-editor.md)。升级后需要继续编辑图片或表格、依赖旧插件或原生实例接口的页面，显式保留原 `Editor` 和对应资源。

## See also

- [fields.md](fields.md)：基础字段（TextBox / NumberBox / DatePicker / DropDownList 等）
- [validation.md](validation.md)：字段校验
- `树`：DropDownBox 内嵌树
- `窗口`：TriggerBox 触发弹窗选择
