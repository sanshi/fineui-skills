# 树节点与数据（Nodes & Data）

## 节点属性对照

| 概念 | F.js | C#（Pro / Core TreeNode） |
|------|------|---------------------------|
| 文本 | `text` | `Text` |
| 节点 ID | `id` | `NodeID` |
| 展开 | `expanded` | `Expanded` |
| 叶子（不可展开） | `leaf` | `Leaf` |
| 子节点 | `children: [...]` | 嵌套 `<f:TreeNode>` / `.Nodes(...)` |
| 内置图标 | `icon: 'url'` | `Icon="TagBlue"`（内置枚举） |
| 自定义图标 | `icon: '../x.png'` | `IconUrl="~/res/x.png"` |
| 超链接 | `href` / `hrefTarget` | `NavigateUrl` / `Target` |
| 提示 | `qtip` | `ToolTip` |
| 初始勾选 | `checked` | `Checked` |

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

隐藏所有节点图标：容器 `EnableIcons="false"`（C#）。

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

## 声明式数据源（Pro）

```csharp
Tree1.DataSource = XmlDataSource1;
Tree1.DataBind();
```

## See also

- [checkbox-events.md](checkbox-events.md)：复选框、事件、懒加载
