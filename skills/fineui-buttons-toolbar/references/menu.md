# 下拉菜单（Menu / MenuButton）

## 菜单项类型

| 用途 | F.js | C#（Pro / Core） |
|------|------|------------------|
| 可点击菜单项 / 子菜单入口 | `type: 'MenuItem'` / `MenuButton` | `<f:MenuButton>`（带 `<Menu>` 即子菜单） |
| 超链接菜单项 | `type: 'MenuHyperLink'` | `<f:MenuHyperLink NavigateUrl Target>` |
| 可勾选菜单项 | `type: 'MenuCheckBox'` | `<f:MenuCheckBox>` |
| 标题/不可点 | `type: 'MenuText'` | `<f:MenuText HideOnClick="false">` |
| 分隔线 | `type: 'MenuSeparator'` | `<f:MenuSeparator>` |

> **C# 没有 `<f:MenuItem>` 标签**——可点击菜单项用 `<f:MenuButton>`（它带 `<Menu>` 时即成为子菜单入口）。F.js 才有 `type: 'MenuItem'`。

## Button + 下拉菜单（含子菜单）

按钮挂一个 `Menu`；菜单项里再挂 `Menu` 即多级子菜单。

```javascript
// F.js —— menu 挂在按钮上
{ type: 'Button', text: '中国科学技术大学', iconFont: 'bicycle', menu: {
    items: [
        { type: 'MenuHyperLink', text: '化学与材料科学学院', href: 'http://scms.ustc.edu.cn/', hrefTarget: '_blank' },
        { type: 'MenuButton', text: '管理学院', menu: { items: [
            { type: 'MenuHyperLink', text: '工商管理系', href: 'http://is.ustc.edu.cn/', hrefTarget: '_blank' }
        ] } }
    ]
} }
```
```aspx
<%-- Pro / Core-TagHelper —— <Menu> 内嵌 --%>
<f:Button runat="server" ID="btnMenu" Text="中国科学技术大学" IconFont="_Bicycle" EnablePostBack="false">
    <Menu runat="server">
        <f:MenuHyperLink runat="server" Icon="TagGreen" Target="_blank" NavigateUrl="http://scms.ustc.edu.cn/" Text="化学与材料科学学院" />
        <f:MenuButton runat="server" Icon="TagBlue" Text="管理学院">
            <Menu runat="server">
                <f:MenuHyperLink runat="server" Target="_blank" NavigateUrl="http://is.ustc.edu.cn/" Text="工商管理系" />
            </Menu>
        </f:MenuButton>
    </Menu>
</f:Button>
```
```csharp
// Core-MVC（Fluent）
@(F.Button().ID("btnMenu").Text("中国科学技术大学").IconFont(IconFont._Bicycle)
    .Menu(F.Menu().Items(
        F.MenuHyperLink().Icon(Icon.TagGreen).Target("_blank").NavigateUrl("http://scms.ustc.edu.cn/").Text("化学与材料科学学院"),
        F.MenuButton().Icon(Icon.TagBlue).Text("管理学院")
            .Menu(F.Menu().Items(
                F.MenuHyperLink().Target("_blank").NavigateUrl("http://is.ustc.edu.cn/").Text("工商管理系")
            ))
    )))
```
```html
<!-- Core-TagHelper（RazorForms / RazorPages）：结构同 Pro 的 <Menu> 内嵌 -->
<f:Button ID="btnMenu" Text="中国科学技术大学" IconFont="_Bicycle">
    <Menu>
        <f:MenuHyperLink Icon="TagGreen" Target="_blank" NavigateUrl="http://scms.ustc.edu.cn/" Text="化学与材料科学学院"></f:MenuHyperLink>
        <f:MenuButton Icon="TagBlue" Text="管理学院">
            <Menu><f:MenuHyperLink Target="_blank" NavigateUrl="http://is.ustc.edu.cn/" Text="工商管理系"></f:MenuHyperLink></Menu>
        </f:MenuButton>
    </Menu>
</f:Button>
```

## 菜单项点击事件（MenuButton）

```aspx
<%-- Pro / Core-TagHelper：服务端 OnClick，或客户端 Listener --%>
<f:MenuButton runat="server" Text="打开官网" OnClick="menuOpen_Click" />
<f:MenuButton runat="server" Text="反选" EnablePostBack="false">
    <Listeners><f:Listener Event="click" Handler="onSelectInverse" /></Listeners>
</f:MenuButton>
```

## 可勾选菜单项（MenuCheckBox）

`GroupName` 相同即单选组（不同则各自多选）；`AutoPostBack="true"` + `OnCheckedChanged` 勾选即回发。**三种模式事件绑定不同：**

```aspx
<%-- Pro / Core-RazorForms：OnCheckedChanged="方法名" --%>
<f:MenuCheckBox runat="server" Text="English" ID="MenuLangEN" GroupName="MenuLang" AutoPostBack="true"
    OnCheckedChanged="MenuLang_CheckedChanged" Checked="true" />
```
```csharp
// Core-MVC（Fluent）：Url.Action + Parameter
F.MenuCheckBox().ID("MenuLangEN").Text("English").GroupName("MenuLang").Checked(true)
    .OnCheckedChanged(Url.Action("MenuLang_CheckedChanged"), new Parameter("checkedValue", "getMenuChecked('lang')"))
```
```html
<!-- Core-RazorPages：@Url.Handler + OnCheckedChangedParameter1 -->
<f:MenuCheckBox Text="English" ID="MenuLangEN" GroupName="MenuLang"
    OnCheckedChanged="@Url.Handler(&quot;MenuLang_CheckedChanged&quot;)"
    OnCheckedChangedParameter1="@(new Parameter(&quot;checkedValue&quot;, &quot;getMenuChecked('lang')&quot;))" Checked="true"></f:MenuCheckBox>
```
```csharp
// 后台：Pro/RazorForms 事件方法（CheckedEventArgs）；MVC/RazorPages action/OnPost 带参
protected void MenuLang_CheckedChanged(object sender, CheckedEventArgs e) { /* Pro/RazorForms */ }
public IActionResult OnPostMenuLang_CheckedChanged(string checkedValue) {   // RazorPages
    UIHelper.Label("labResult").Text("语言：" + checkedValue); return UIHelper.Result();
}
```

客户端读取勾选项：`F.ui.btnMenu.menu.getCheckedItems()`，逐项 `item.isChecked()`。

## See also

- [button.md](button.md)：Button
- `fineui-layout`：把菜单按钮放进 Toolbar
- `fineui-foundation` 的 rawhtml.md：菜单项文本含 HTML（v15.2 默认转义，用 `F.rawHtml`/`_TextRawHtml`）
