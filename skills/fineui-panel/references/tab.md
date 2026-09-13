# TabStrip 选项卡

容器：`Height`、`TabPosition="Top"`、`ActiveTabIndex`（默认激活）、`EnableTabCloseMenu`、`ShowBorder`。每个 `Tab` 有 `Title`/`TitleRawHtml`、`BodyPadding`、`Layout`、`Closable`、`Disabled`、`Icon`；内容用 `<Items>`/`items`，简单文本可用 `content`(F.js)/`Content`。

> **Java（Thymeleaf 方言）**：`<f:tab-strip tab-position= active-tab-index= enable-tab-close-menu= show-border= height= is-fluid=>`，选项卡集合放 `<f:tabs>`，每个 `<f:tab title= title-raw-html= body-padding= layout= enable-iframe= iframe-url= content=>`，内容放 `<f:items>`。自定义属性用 `<f:attributes>`/`<f:attribute key= value=>`。

## 静态选项卡

```javascript
// F.js
F.create({ type: 'TabStrip', isFluid: true, id: 'TabStrip1', renderTo: '#wrap', height: 350, tabPosition: 'top', activeTabIndex: 1,
    defaults: { autoScroll: true, bodyPadding: 10 },
    items: [
        { type: 'Tab', title: '标签一', layout: 'fit', items: [ /* 表单/表格 */ ] },
        { type: 'Tab', title: F.rawHtml('<span class="hot">标签二</span>'), content: '内容', closable: true },
        { type: 'Tab', title: '标签三', disabled: true, content: '内容' }
    ] });
```
```aspx
<%-- Pro / Core-TagHelper --%>
<f:TabStrip ID="TabStrip1" runat="server" IsFluid="true" Height="350px" TabPosition="Top" ActiveTabIndex="1">
    <Tabs>
        <f:Tab Title="标签一" BodyPadding="10px" Layout="Fit" runat="server"><Items> ... </Items></f:Tab>
        <f:Tab TitleRawHtml="<span class='hot'>标签二</span>" BodyPadding="10px" runat="server"><Items>
            <f:Button Text="按钮" runat="server" /></Items></f:Tab>
    </Tabs>
</f:TabStrip>
<%-- Core-TagHelper 富文本标题用下划线便捷形式：_TitleRawHtml="<span ...>标签二</span>" --%>
```
```csharp
// Core-MVC（Fluent）
@(F.TabStrip().IsFluid(true).ID("TabStrip1").Height(350).TabPosition(TabPosition.Top).ActiveTabIndex(1)
    .Tabs(
        F.Tab().Title("标签一").BodyPadding(10).Layout(LayoutType.Fit).Items( /* ... */ ),
        F.Tab().TitleRawHtml(new RawHtml("<span class='hot'>标签二</span>")).BodyPadding(10).Items(F.Button().Text("按钮"))
    ))
```
```html
<!-- FineUIJava（Thymeleaf 方言）-->
<f:tab-strip id="TabStrip1" is-fluid="true" height="350" tab-position="Top" active-tab-index="1" enable-tab-close-menu="false" show-border="true">
    <f:tabs>
        <f:tab title="标签一" body-padding="10" layout="Fit"><f:items> ... </f:items></f:tab>
        <f:tab title-raw-html="<span class='hot'>标签二</span>" body-padding="10">
            <f:items><f:button id="Button1" text="按钮"></f:button></f:items>
        </f:tab>
    </f:tabs>
</f:tab-strip>
```

## Tab 内嵌 iframe

```aspx
<%-- Pro / Core-TagHelper --%>
<f:Tab Title="标签二（IFrame）" EnableIFrame="true" IFrameUrl="~/Panel/Group" runat="server">
    <Listeners><f:Listener Event="iframeload" Handler="onTabIFrameLoad" /></Listeners>
</f:Tab>
```
```csharp
// Core-MVC（Fluent）
F.Tab().Title("标签二（IFrame）").EnableIFrame(true).IFrameUrl(Url.Content("~/Panel/Group")).Listener("iframeload", "onTabIFrameLoad")
```
```html
<!-- FineUIJava（Thymeleaf 方言）：enable-iframe + iframe-url，iframeload 处理器是客户端 JS（同 F.js）-->
<f:tab id="Tab2" title="标签二（IFrame）" enable-iframe="true" iframe-url="/panel/group">
    <f:listeners><f:listener event="iframeload" handler="onTabIFrameLoad" /></f:listeners>
</f:tab>
```

## 动态增删选项卡

### 客户端（各写法一致）

```javascript
F.ui.TabStrip1.addTab({ id: 'tab_x', iframe: true, iframeUrl: 'https://x.com/', title: '新标签', closable: true, icon: '/res/icon/x.png' });
F.ui.TabStrip1.closeTab('tab_x');
F.ui.TabStrip1.activeNextTab();
```

### 服务端

```csharp
// Core-MVC / RazorPages —— UIHelper
UIHelper.TabStrip("TabStrip1").AddTab("tab_x", "https://x.com/", "新标签", IconHelper.GetIconUrl(Icon.Application), true);
UIHelper.TabStrip("TabStrip1").CloseTab("tab_x");
```
```csharp
// Pro / Core-RazorForms —— 注册脚本 + GetAddTabReference
PageContext.RegisterStartupScript(TabStrip1.GetAddTabReference("tab_x", "https://x.com/", "新标签", IconHelper.GetIconUrl(Icon.Application), true));
// RazorForms 用 RegisterStartupScript(TabStrip1.GetAddTabReference(...))
```
```java
// FineUIJava 页面类 —— 直接在控件字段上调 addTab / hideTab（框架下发运行时命令，无需注册脚本）
public void btnAddTab_Click(Object sender, EventArgs e) {
    TabStrip1.addTab("dynamic_tab", "https://deepseek.com/", "新标签",
            IconHelper.namedIconUrl("Application"), true);   // 参数：id, url, title, iconUrl, closable
}
public void btnRemoveTab_Click(Object sender, EventArgs e) {
    TabStrip1.hideTab("dynamic_tab");
}
```

> **相同 `id` 会复用同一个选项卡**（再次打开不新建）。动态 Tab 是客户端加的，服务端控件树取不到——非 Ajax 回发会丢失。

## See also

- [panel.md](panel.md)：Panel
- [accordion.md](accordion.md)：Accordion
- `fineui-window`：Tab 里放 iframe 编辑页
