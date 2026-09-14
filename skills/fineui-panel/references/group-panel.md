# GroupPanel 分组面板

GroupPanel 用带标题的边框组织一组相关控件，常见于较长表单的分区。它与 Panel 一样可以包含 `Items`、`Toolbars`，也支持折叠；仅展示一段普通内容时仍优先使用 ContentPanel。

## 各写法

### F.js

```javascript
F.create({
    type: 'GroupPanel', id: 'GroupPanel1', renderTo: '#wrap',
    title: '基本信息', layout: 'anchor', bodyPadding: 10,
    collapsible: true, collapsed: false,
    items: [
        { type: 'TextBox', id: 'TextBox1', fieldLabel: '名称' },
        { type: 'TextArea', id: 'TextArea1', fieldLabel: '备注' }
    ]
});
```

### FineUIPro

```aspx
<f:GroupPanel ID="GroupPanel1" runat="server" Title="基本信息"
    BodyPadding="10" EnableCollapse="true">
    <Items>
        <f:SimpleForm ID="SimpleForm1" runat="server" ShowBorder="false" ShowHeader="false">
            <Items>
                <f:TextBox ID="TextBox1" runat="server" Label="名称" />
                <f:TextArea ID="TextArea1" runat="server" Label="备注" />
            </Items>
        </f:SimpleForm>
    </Items>
</f:GroupPanel>
```

### FineUICore MVC

```csharp
@(F.GroupPanel().ID("GroupPanel1").Title("基本信息").BodyPadding(10).EnableCollapse(true)
    .Items(
        F.SimpleForm().ShowBorder(false).ShowHeader(false).Items(
            F.TextBox().ID("TextBox1").Label("名称"),
            F.TextArea().ID("TextArea1").Label("备注"))))
```

### FineUICore RazorForms / RazorPages

```html
<f:GroupPanel ID="GroupPanel1" Title="基本信息" BodyPadding="10" EnableCollapse="true">
    <Items>
        <f:SimpleForm ID="SimpleForm1" ShowBorder="false" ShowHeader="false">
            <Items>
                <f:TextBox ID="TextBox1" Label="名称"></f:TextBox>
                <f:TextArea ID="TextArea1" Label="备注"></f:TextArea>
            </Items>
        </f:SimpleForm>
    </Items>
</f:GroupPanel>
```

### FineUIJava

```html
<f:group-panel id="GroupPanel1" title="基本信息" body-padding="10" enable-collapse="true">
    <f:items>
        <f:simple-form id="SimpleForm1" show-border="false" show-header="false">
            <f:items>
                <f:text-box id="TextBox1" label="名称"></f:text-box>
                <f:text-area id="TextArea1" label="备注"></f:text-area>
            </f:items>
        </f:simple-form>
    </f:items>
</f:group-panel>
```

## 折叠与服务端控制

| 用途 | F.js | Pro / Core | FineUIJava |
|------|------|------------|------------|
| 允许折叠 | `collapsible: true` | `EnableCollapse="true"` | `enable-collapse="true"` |
| 初始折叠 | `collapsed: true` | `Collapsed="true"` | `collapsed="true"` |
| 判断是否折叠 | `isCollapsed()` | `Collapsed` | `isCollapsed()` |
| 展开/折叠 | `expand()` / `collapse()` | Pro / Core RazorForms 设置 `Collapsed`；Core MVC / RazorPages 使用 `UIHelper.GroupPanel(id).Collapsed(...)` | `setCollapsed(false/true)` |
| 切换状态 | `toggleCollapse()` | 读取当前状态后按上一行方式设置相反值 | 根据 `isCollapsed()` 设置 |

GroupPanel 的工具栏写法与 Panel 相同：C# 放在 `<Toolbars>`，Java 放在 `<f:toolbars>`，F.js 使用 `bars`。工具栏详细结构见 [panel.md](panel.md)。
