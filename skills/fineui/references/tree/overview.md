# FineUI 树技能（Tree）

## 何时使用（When to Use）

- 展示层级数据（组织架构、地区、菜单、分类）
- 复选框树 + 级联选中 + 读取选中节点
- 节点点击/展开事件、异步懒加载子节点

## 开始前（Before You Start）

1. **哪种写法？** F.js / Pro / Core-MVC / Core-RazorForms / Core-RazorPages / Java（判定见 `基础`）。
2. **节点从哪来？** 静态写死 / 后台代码构建（C# `Tree.Nodes.Add(new TreeNode{...})` 或 ViewBag；Java `Tree1.addNode(node)` + `node.addChild(child)`）/ 异步懒加载。
3. **要不要复选框？** `checkbox`(F.js) / `EnableCheckBox`(C#) / `enable-check-box`(Java) + 级联 `CascadeCheck` / `cascade-check`。

## 各写法速览（一个静态树）

```javascript
// ① F.js —— rootNode.children 挂节点，节点是 JSON 对象
F.create({
    type: 'Tree', isFluid: true, id: 'Tree1', renderTo: '#wrap', height: 300, title: '树控件',
    rootNode: { expanded: true, children: [
        { text: '中国', id: 'china', expanded: true, children: [
            { text: '河南省', id: 'henan', children: [
                { text: '驻马店市', id: 'zhumadian' },
                { text: '漯河市', id: 'luohe' }
            ] }
        ] }
    ] }
});
```
```aspx
<%-- ② Pro（WebForms）：<Nodes> 里 <f:TreeNode> 嵌套 --%>
<f:Tree ID="Tree1" runat="server" IsFluid="true" ShowHeader="true" Title="树控件">
    <Nodes>
        <f:TreeNode Text="中国" NodeID="china" Expanded="true">
            <f:TreeNode Text="河南省" NodeID="henan" Expanded="true">
                <f:TreeNode Text="驻马店市" NodeID="zhumadian" />
                <f:TreeNode Text="漯河市" NodeID="luohe" />
            </f:TreeNode>
        </f:TreeNode>
    </Nodes>
</f:Tree>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Tree().IsFluid(true).ID("Tree1").ShowHeader(true).Title("树控件")
    .Nodes(
        F.TreeNode().Text("中国").NodeID("china").Expanded(true).Nodes(
            F.TreeNode().Text("河南省").NodeID("henan").Expanded(true).Nodes(
                F.TreeNode().Text("驻马店市").NodeID("zhumadian"),
                F.TreeNode().Text("漯河市").NodeID("luohe")
            )
        )
    ))
```
```html
<!-- ④ Core-TagHelper（RazorForms / RazorPages，写法相同）-->
<f:Tree ID="Tree1" IsFluid="true" ShowHeader="true" Title="树控件">
    <Nodes>
        <f:TreeNode Text="中国" NodeID="china" Expanded="true">
            <f:TreeNode Text="河南省" NodeID="henan" Expanded="true">
                <f:TreeNode Text="驻马店市" NodeID="zhumadian" />
            </f:TreeNode>
        </f:TreeNode>
    </Nodes>
</f:Tree>
```
```html
<!-- ⑤ FineUI.Java（Thymeleaf 方言：标签/属性全 kebab-case；节点用 <f:nodes> 包裹）-->
<f:tree id="Tree1" is-fluid="true" show-header="true" title="树控件">
    <f:nodes>
        <f:tree-node text="中国" node-id="china" expanded="true">
            <f:tree-node text="河南省" node-id="henan" expanded="true">
                <f:tree-node text="驻马店市" node-id="zhumadian"></f:tree-node>
            </f:tree-node>
        </f:tree-node>
    </f:nodes>
</f:tree>
```

> Java 结构与 Core-RazorForms 完全对应：容器 `<f:tree>`、子节点集合标签 `<f:nodes>`（Core 是 `<Nodes>`）、节点 `<f:tree-node>`。属性全 kebab-case（`is-fluid`/`show-header`/`node-id`）；初始选中节点用容器属性 `selected-node-id="..."`。

## 后台切换与清空（v16.1.0）

Core/Java 可动态切换复选框及图标、选中/悬停样式、链接、多选、双击或单击展开、同级单节点展开、右键选中及父子级联；Java 使用相应的 `set...` 方法，Core 可直接给控件属性赋值。切换保留已有节点、勾选、选择和展开结果，从下一次交互采用新规则。清空整棵树使用 `Tree1.ClearData()` / `Tree1.clearData()`，F.js 使用 `clearData()`；它们同时清除选择且不触发选择事件。Pro 的动态能力按其公开属性与对应版本核对，不将 Core/Java 新增接口直接套用。

## 参考文档（Documentation Reference Files）

| 文件 | 何时读 |
|------|--------|
| [references/nodes-data.md](references/nodes-data.md) | 节点属性、图标、后台构建节点、数据绑定（DataSet 递归） |
| [references/checkbox-events.md](references/checkbox-events.md) | 复选框树/级联/读选中、节点点击/展开事件、异步懒加载 |

## 相关分类（Related Skills）

- `基础`：写法判定、命名约定
- `窗口`：树节点点击后弹编辑窗口

## 约束与规则（Constraints & Rules）

1. **先定写法、不混用**：F.js 节点属性 camelCase（`text`/`id`/`expanded`）；C# TreeNode PascalCase（`Text`/`NodeID`/`Expanded`）；**Java 标签属性 kebab-case（`text`/`node-id`/`expanded`）**。F.js 用 `id`，C# 用 `NodeID`，**Java 标签用 `node-id`——但 Java 的 `TreeNode` Bean 用 `getId()/setId(...)`（不是 `NodeID`）**。
2. **级联选中是 `CascadeCheck` / `cascade-check`**：不是 `AutoCheckParent`/`AutoCheckChildNodes`（这些不存在）。
3. **读取选中节点用 `GetCheckedNodes()`**（C# 返回 `TreeNode[]`；**Java `getCheckedNodes()` 返回 `List<TreeNode>`**）；没有 `GetCheckedNodeIDs`。F.js 客户端 `getCheckedNodes(true)`。
4. **读选中的方式随写法不同**：RazorForms / **Java** 用控件字段 `Tree1.GetCheckedNodes()` / `Tree1.getCheckedNodes()`；MVC/RazorPages 用客户端 `getCheckedNodes(true)` 序列化后回发 `JArray`。详见 [references/checkbox-events.md](references/checkbox-events.md)。
5. **懒加载用 `OnNodeLazyLoad` / `on-node-lazy-load` + `AutoLeafIdentification="false"` / `auto-leaf-identification="false"` + 节点 `Leaf` / `leaf`**（不是 `EnableAjax`）；`Leaf=false` 的节点展开时触发懒加载，`Leaf=true` 为叶子。
6. **节点客户端动作使用 `ClickHandler` / `click-handler`**：值只写页面具名函数，签名为 `(event, nodeId)`；显式返回 `false` 可取消后续默认动作与服务端事件。不要在新代码中写 `OnClientClick` / `on-client-click`。
7. **绝不编造 API**：不确定就查官网公开 API 与同模式官方示例。

## 官方资源（Official Resources）

- 在线 API：JS https://fineui.com/js/api/ · Pro https://fineui.com/pro/api/ · Core https://fineui.com/core/api/
