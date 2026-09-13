# 复选框、事件与懒加载（CheckBox / Events / Lazy Load）

## 一、复选框树

容器开 `EnableCheckBox`；级联勾选 `CascadeCheck`；节点 `Checked` 设初始勾选。

```javascript
// F.js —— checkbox:true + nodecheck 事件
F.create({ type: 'Tree', id: 'Tree1', checkbox: true, rootNode: { ... },
    listeners: { nodecheck: function (event, nodeId, checked) {
        showNotify('节点 ' + nodeId + ' 勾选=' + checked);
    } } });
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:Tree ID="Tree1" runat="server" EnableCheckBox="true" CascadeCheck="true" ShowHeader="true" Title="树">
    <Nodes><f:TreeNode Text="河南省" Checked="true"><f:TreeNode Text="遂平县" NodeID="suiping" Checked="true" /></f:TreeNode></Nodes>
</f:Tree>
```
```csharp
// Core-MVC（Fluent）
@(F.Tree().ID("Tree1").EnableCheckBox(true).CascadeCheck(true).Nodes( /* ... */ ))
```
```html
<!-- FineUIJava（Thymeleaf 方言）：enable-check-box + cascade-check；节点 checked 设初始勾选 -->
<f:tree id="Tree1" is-fluid="true" show-header="true" title="树" enable-check-box="true" cascade-check="true">
    <f:nodes>
        <f:tree-node text="河南省" checked="true">
            <f:tree-node text="遂平县" node-id="suiping" checked="true"></f:tree-node>
        </f:tree-node>
    </f:nodes>
</f:tree>
```

> 复选框相关容器属性（Java kebab-case）：`enable-check-box`、`cascade-check`（级联）、`only-leaf-check`（只叶子可勾）、`only-folder-check`（只目录可勾）。

## 二、读取选中的复选框节点

### F.js（客户端）

```javascript
var nodes = F.ui.Tree1.getCheckedNodes(true);   // [{id, text, ...}]
F.ui.Tree1.checkNodes(['zhumadian', 'luohe']);   // 勾选
F.ui.Tree1.uncheckNodes(['zhumadian']);          // 取消
```

### Pro / Core-RazorForms —— 服务端控件字段 `GetCheckedNodes()`

```csharp
protected void btnGetChecked_Click(object sender, EventArgs e) {
    TreeNode[] nodes = Tree1.GetCheckedNodes();          // TreeNode[]
    foreach (TreeNode node in nodes)
        sb.AppendFormat("<li>{0}（{1}）</li>", node.Text, node.NodeID);
    labResult.Text = sb.ToString();
}
```

### FineUIJava —— 控件字段 `getCheckedNodes()`（同 RazorForms，返回 `List<TreeNode>`）

```java
// FineUIJava 页面类
com.fineui.java.core.controls.Tree Tree1;   // 页面类同名与 Tree 重名时用全限定
Label labResult;

public void btnGetCheckedValues_Click(Object sender, EventArgs e) {
    List<TreeNode> nodes = Tree1.getCheckedNodes();   // List<TreeNode>（不是数组）
    StringBuilder sb = new StringBuilder("<ul>");
    for (TreeNode node : nodes)
        sb.append(String.format("<li>%s（%s）</li>", node.getText(), node.getId()));  // getId 非 getNodeID
    sb.append("</ul>");
    labResult.setTextRawHtml(new RawHtml(sb.toString()));   // 仅可信 HTML 使用 RawHtml
}
```

> Label 输出 HTML：模板可保持 `<f:label id="labResult">`，服务端调用 `setTextRawHtml(new RawHtml(...))`。不要再用 `encode-text="false"` 作为通用写法；用户输入与数据库内容必须按普通文本输出。

### Core-MVC / RazorPages —— 客户端收集回发 `JArray`

```javascript
// 页面内 JS
function getCheckedNodes() {
    var result = [];
    $.each(F.ui.Tree1.getCheckedNodes(true), function (i, n) { result.push({ id: n.id, text: n.text }); });
    return F.toJSON(result);
}
```
```csharp
// 按钮：MVC  .OnClick(Url.Action("btnGetChecked_Click"), new Parameter("checkedNodes", "getCheckedNodes()"))
//       RazorPages  OnClick="@Url.Handler("btnGetChecked_Click")" OnClickParameter1="@(new Parameter("checkedNodes","getCheckedNodes()"))"
public IActionResult btnGetChecked_Click(JArray checkedNodes) {              // RazorPages: OnPostBtnGetChecked_Click
    foreach (JObject n in checkedNodes)
        sb.AppendFormat("<li>{0}（{1}）</li>", n.Value<string>("text"), n.Value<string>("id"));
    UIHelper.Label("labResult").Text(sb.ToString());
    return UIHelper.Result();
}
```

## 三、节点点击事件

```javascript
// F.js —— nodeclick 监听
listeners: { nodeclick: function (event, nodeId) { showNotify('点击了 ' + nodeId); } }
```

C# 端点击事件三模式绑定不同：

```csharp
// Pro —— OnNodeCommand + 节点 EnableClickEvent="true"
// <f:Tree OnNodeCommand="Tree1_NodeCommand"><f:TreeNode EnableClickEvent="true" .../>
protected void Tree1_NodeCommand(object sender, TreeCommandEventArgs e) {
    labResult.Text = "点击了：" + e.Node.Text;
}
```
```csharp
// Core-MVC —— Listener("nodeclick") + F.doPostBack
// @(F.Tree().Listener("nodeclick", "onTree1NodeClick"))
// JS: F.doPostBack({ url: '@Url.Action("Tree1_NodeClick")', params: { nodeId: nodeId, nodeText: this.getNodeData(nodeId).text } });
public IActionResult Tree1_NodeClick(string nodeId, string nodeText) {
    UIHelper.Label("labResult").Text($"点击了：{nodeId}（{nodeText}）");
    return UIHelper.Result();
}
```
```csharp
// Core-RazorForms —— Listener + F.customEvent + Page_CustomEvent
// JS: F.customEvent('Tree1_NodeClick', nodeId);
protected void Page_CustomEvent(object sender, CustomEventArgs e) {
    if (e.EventName == "Tree1_NodeClick")
        labResult.Text = $"点击了：{Tree1.FindNode(e.EventArguments).Text}";   // 控件字段 FindNode
}
```
```java
// FineUIJava 页面类 —— 与 Core-RazorForms 同构：Listener + F.customEvent + Page_CustomEvent
// 模板：<f:listeners><f:listener event="nodeclick" handler="onTree1NodeClick"></f:listener></f:listeners>
// 页面脚本（同 F.js，不重复）：function onTree1NodeClick(event, nodeId){ ... F.customEvent('Tree1_NodeClick', nodeId); }
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("Tree1_NodeClick".equals(e.getEventName())) {
        String nodeId = e.getArgument();
        labResult.setText(String.format("点击了：%s（%s）", nodeId, Tree1.findNode(nodeId).getText()));
    }
}
```

> 客户端 `nodeclick` 处理函数（`F.customEvent(...)` 触发后台）四栈完全相同，**不重复**——见上面 F.js 段。Java 侧只有后台入口不同：`Page_CustomEvent(Object, CustomEventArgs)`，读值用 `e.getEventName()` / `e.getArgument()`，节点查找 `Tree1.findNode(id).getText()`。

## 四、展开 / 折叠事件

```csharp
// Pro / Core-RazorForms —— OnNodeExpand="方法名" + 节点 EnableExpandEvent="true"
protected void Tree1_NodeExpand(object sender, TreeNodeEventArgs e) {
    labResult.Text = "展开：" + e.Node.Text;   // e.NodeID / e.Node
}
```
```csharp
// Core-MVC —— .OnNodeExpand(Url.Action("Tree1_NodeExpand"), new Parameter("nodeInfo", "getNodeInfo(arguments[1])"))
public IActionResult Tree1_NodeExpand(JObject nodeInfo) {
    UIHelper.Label("labResult").Text($"展开：{nodeInfo.Value<string>("text")}");
    return UIHelper.Result();
}
// Core-RazorPages —— OnNodeExpand="@Url.Handler("Tree1_NodeExpand")" → OnPostTree1_NodeExpand(...)
```
```java
// FineUIJava 页面类 —— 容器 on-node-expand / on-node-collapse 直接指向同名方法（同 RazorForms）
// 模板：<f:tree ... on-node-expand="Tree1_NodeExpand" on-node-collapse="Tree1_NodeCollapse">
public void Tree1_NodeExpand(Object sender, TreeNodeEventArgs e) {
    labResult.setText(String.format("展开节点：%s（%s）", e.getNodeID(), e.getNode().getText()));
}
public void Tree1_NodeCollapse(Object sender, TreeNodeEventArgs e) {
    labResult.setText(String.format("折叠节点：%s（%s）", e.getNodeID(), e.getNode().getText()));
}
```

## 五、异步懒加载（展开时动态加子节点）

容器 `AutoLeafIdentification="false"` + `OnNodeLazyLoad`；节点 `Leaf=false`（可展开→触发懒加载）/ `Leaf=true`（叶子）。展开时后台按 `nodeId` 生成子节点回灌。

```csharp
// 生成子节点（三模式相同）
private List<TreeNode> DynamicAppendNode(string nodeId) {
    var nodes = new List<TreeNode>();
    switch (nodeId) {
        case "zhumadian":
            nodes.Add(new TreeNode { Text = "遂平县", NodeID = "suiping", Leaf = false }); // 还能展开
            nodes.Add(new TreeNode { Text = "西平县", NodeID = "xiping", Leaf = true });   // 叶子
            break;
    }
    return nodes;
}
```
```csharp
// Core-MVC —— .OnNodeLazyLoad(Url.Action("Tree1_NodeLazyLoad"), new Parameter("nodeId", "arguments[1]"))
public IActionResult Tree1_NodeLazyLoad(string nodeId) {
    UIHelper.Tree("Tree1").LoadData(nodeId, DynamicAppendNode(nodeId));   // UIHelper.Tree(...).LoadData
    return UIHelper.Result();
}
```
```csharp
// Core-RazorForms —— OnNodeLazyLoad="Tree1_NodeLazyLoad"
protected void Tree1_NodeLazyLoad(object sender, TreeNodeEventArgs e) {
    RegisterStartupScript(Tree1.GetLoadDataReference(e.NodeID, DynamicAppendNode(e.NodeID)));  // 控件字段
}
// Core-RazorPages —— OnNodeLazyLoad="@Url.Handler("Tree1_NodeLazyLoad")" + Parameter → OnPostTree1_NodeLazyLoad(string nodeId) + UIHelper.Tree(...).LoadData
```
```csharp
// Pro —— OnNodeLazyLoad="Tree1_NodeLazyLoad"，在 e.Node.Nodes 上加子节点
protected void Tree1_NodeLazyLoad(object sender, TreeNodeEventArgs e) {
    e.Node.Nodes.Clear();
    foreach (var n in DynamicAppendNode(e.Node.NodeID)) e.Node.Nodes.Add(n);
}
```
```java
// FineUIJava 页面类 —— 容器 auto-leaf-identification="false" + on-node-lazy-load="Tree1_NodeLazyLoad"
// 用 Tree1.loadData(nodeId, 子节点列表) 回灌（对应 RazorForms 的 GetLoadDataReference / MVC 的 UIHelper.Tree().LoadData）
public void Tree1_NodeLazyLoad(Object sender, TreeNodeEventArgs e) {
    Tree1.loadData(e.getNodeID(), dynamicAppendNode(e.getNodeID()));
}

// 生成子节点：TreeNode setter；leaf=false 还能展开（再次懒加载），leaf=true 为叶子
static List<TreeNode> dynamicAppendNode(String nodeId) {
    List<TreeNode> nodes = new ArrayList<>();
    if ("zhumadian".equals(nodeId)) {
        TreeNode n1 = new TreeNode(); n1.setText("遂平县"); n1.setId("suiping"); n1.setLeaf(false);
        TreeNode n2 = new TreeNode(); n2.setText("西平县"); n2.setId("xiping");  n2.setLeaf(true);
        nodes.add(n1); nodes.add(n2);
    }
    return nodes;
}
```

## See also

- [nodes-data.md](nodes-data.md)：节点属性、图标、后台建树
- `fineui-window`：点击节点弹编辑窗口
