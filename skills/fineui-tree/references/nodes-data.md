# 树节点与数据（Nodes & Data）

## 节点属性对照

| 概念 | F.js | C#（Pro / Core TreeNode） | Java（`<f:tree-node>` 标签 / `TreeNode` Bean） |
|------|------|---------------------------|------------------------------------------------|
| 文本 | `text` | `Text` | `text=` / `setText(...)` |
| 节点 ID | `id` | `NodeID` | `node-id=` / `setId(...)`（**Bean 是 `setId`，非 NodeID**） |
| 展开 | `expanded` | `Expanded` | `expanded=` / `setExpanded(...)` |
| 叶子（不可展开） | `leaf` | `Leaf` | `leaf=` / `setLeaf(...)` |
| 子节点 | `children: [...]` | 嵌套 `<f:TreeNode>` / `.Nodes(...)` | 嵌套 `<f:tree-node>`（外层 `<f:nodes>`）/ `node.addChild(child)` |
| 内置图标 | `icon: 'url'` | `Icon="TagBlue"`（内置枚举） | `icon="TagBlue"`（值保持 PascalCase） |
| 自定义图标 | `icon: '../x.png'` | `IconUrl="~/res/x.png"` | `icon-url="~/res/x.png"` |
| 超链接 | `href` / `hrefTarget` | `NavigateUrl` / `Target` | `navigate-url=` / `target=` |
| 提示 | `qtip` | `ToolTip` | `tool-tip=` |
| 初始勾选 | `checked` | `Checked` | `checked=` |

> **命名陷阱（Java）**：模板标签属性叫 `node-id=`，但服务端 `TreeNode` Bean 的方法是 `getId()/setId(...)`（不是 `getNodeID`）。`Tree1.getSelectedNodeId()` 读取当前选中节点 id。

## 节点图标

```javascript
// F.js
{ text: '驻马店市', id: 'zhumadian', icon: '../res/icon/tag_blue.png' }
```
```aspx
<%-- Pro / Core-TagHelper：内置图标 Icon，或自定义 IconUrl，或超链接节点 --%>
<f:TreeNode Text="遂平县" NodeID="suiping" Icon="TagBlue" />
<f:TreeNode Text="漯河市" NodeID="luohe" IconUrl="~/res/icon/tag_orange.png" />
<f:TreeNode Text="科大（链接）" NodeID="ustc" NavigateUrl="http://www.ustc.edu.cn/" Target="_blank" ToolTip="跳转科大" />
```
```csharp
// Core-MVC（Fluent）
F.TreeNode().Text("遂平县").NodeID("suiping").Icon(Icon.TagBlue)
```
```html
<!-- FineUIJava（Thymeleaf 方言）：内置图标 icon（值同 C#）、自定义 icon-url、超链接节点 -->
<f:tree-node text="遂平县" node-id="suiping" icon="TagBlue"></f:tree-node>
<f:tree-node text="漯河市" node-id="luohe" icon-url="~/res/icon/tag_orange.png"></f:tree-node>
<f:tree-node text="科大（链接）" node-id="ustc" navigate-url="http://www.ustc.edu.cn/" target="_blank" tool-tip="跳转科大"></f:tree-node>
```

隐藏所有节点图标：容器 `EnableIcons="false"`（C#）/ `enable-icons="false"`（Java）。

## 后台构建节点（代码建树）

递归把数据（如 `DataSet` + `Relations`）转成 TreeNode：**顶层加到树、子层加到父节点**。

```csharp
// 递归：三种 C# 写法逻辑相同，差别只在"顶层挂到哪"
private void ResolveSubTree(DataRow dataRow, TreeNode treeNode) {
    DataRow[] rows = dataRow.GetChildRows("TreeRelation");
    if (rows.Length > 0) {
        treeNode.Expanded = true;                 // 有子节点默认展开
        foreach (DataRow row in rows) {
            TreeNode node = new TreeNode { Text = row["Text"].ToString(), NodeID = row["Id"].ToString() };
            treeNode.Nodes.Add(node);             // 挂到父节点
            ResolveSubTree(row, node);
        }
    }
}
```

**顶层挂载与回传，三模式不同：**

```csharp
// Pro / Core-RazorForms —— 直接操作控件字段（顶层加到 Tree1）
protected void Page_Load(object sender, EventArgs e) {
    if (!IsPostBack) {
        foreach (DataRow row in ds.Tables[0].Rows)
            if (row.IsNull("ParentId")) {
                TreeNode node = new TreeNode { Text = row["Text"].ToString() };
                Tree1.Nodes.Add(node);            // 顶层加到控件
                ResolveSubTree(row, node);
            }
    }
}
```
```csharp
// Core-MVC / RazorPages —— 建 List<TreeNode>，放 ViewBag
List<TreeNode> nodes = new List<TreeNode>();
// ... nodes.Add(node); ResolveSubTree(row, node);
ViewBag.Tree1Nodes = nodes.ToArray();
```
```csharp
// Core-MVC / RazorPages 视图 —— 从 ViewBag 灌入
@(F.Tree().ID("Tree1").ShowHeader(true).Title("树").Nodes((TreeNode[])ViewBag.Tree1Nodes))
```

**FineUIJava —— 顶层 `Tree1.addNode(node)`、子层 `parent.addChild(child)`：**

```java
// FineUIJava 页面类：把"自引用关系"扁平数据（Id/Text/ParentId）按父子关系建成层级树
@FineUIPage("tree/data-bind-data-table")
public class DataBindDataTable extends FineUIPageBase {

    Tree Tree1;   // 控件字段：类型 com.fineui.java.core.controls.Tree，同名 = 标签 id

    public void Page_Load(Object sender, EventArgs e) {
        if (!isPostBack()) {
            for (String[] row : ROWS) {
                TreeNode node = new TreeNode();
                node.setText(row[1]);
                node.setId(row[0]);
                if (row[2] == null) Tree1.addNode(node);          // 顶层挂到树
                else                findParent(row[2]).addChild(node);  // 子层挂到父节点
            }
        }
    }
    // 有子节点的目录：parent.setExpanded(true) 默认展开
}
```

> Java 建树 API：`Tree1.addNode(node)`（顶层）、`node.addChild(child)`（子层）、`node.setText/setId/setExpanded/setLeaf(...)`。与 Core 的 `Tree1.Nodes.Add(...)` / `treeNode.Nodes.Add(...)` 语义一一对应，只是换成 Bean 方法。

## 声明式数据源（Pro）

```csharp
Tree1.DataSource = XmlDataSource1;
Tree1.DataBind();
```

## See also

- [checkbox-events.md](checkbox-events.md)：复选框、事件、懒加载
