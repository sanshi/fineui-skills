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
// JS: F.doPostBack('@Url.Action("Tree1_NodeClick")', { nodeId: nodeId, nodeText: this.getNodeData(nodeId).text });
public IActionResult Tree1_NodeClick(string nodeId, string nodeText) {
    UIHelper.Label("labResult").Text($"点击了：{nodeId}（{nodeText}）");
    return UIHelper.Result();
}
```
```csharp
// Core-RazorForms —— Listener + __customEvent + Page_CustomEvent
// JS: __customEvent('Tree1_NodeClick', nodeId);
protected void Page_CustomEvent(object sender, CustomEventArgs e) {
    if (e.EventName == "Tree1_NodeClick")
        labResult.Text = $"点击了：{Tree1.FindNode(e.EventArguments).Text}";   // 控件字段 FindNode
}
```

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

## See also

- [nodes-data.md](nodes-data.md)：节点属性、图标、后台建树
- `fineui-window`：点击节点弹编辑窗口
