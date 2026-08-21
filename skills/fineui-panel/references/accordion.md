# Accordion 手风琴

一组竖直排列的面板，同一时刻展开一个。容器 `<f:Accordion>`（`Title`/`ShowHeader`/`Height`/`ShowBorder`/`ActivePaneIndex`/`EnableCollapse`），面板放 `<Panes>` 里的 `<f:AccordionPane>`（`Title`/`IconUrl`/`BodyPadding`/`Collapsed`，内容用 `<Items>`）。

> **注意**：手风琴面板用 `<Panes>` / `AccordionPane`（不是 TabStrip 的 `<Tabs>`/`Tab`）。
> **Java（Thymeleaf 方言）**：`<f:accordion active-pane-index= enable-collapse= show-header= show-border= height= is-fluid=>`，面板放 `<f:panes>` 里的 `<f:accordion-pane title= icon-url= body-padding=>`，内容用 `<f:items>`。

## 各写法

```javascript
// ① F.js
F.create({ type: 'Accordion', isFluid: true, id: 'Accordion1', renderTo: '#wrap', height: 500, title: '手风琴', header: false,
    items: [
        { type: 'AccordionPane', title: '面板一', icon: '../res/images/16/1.png', collapsed: true, bodyPadding: '2 5',
          items: [{ type: 'Label', encodeText: false, value: '面板一内容' }] },
        { type: 'AccordionPane', title: '面板二', bodyPadding: '2 5', items: [{ type: 'Label', value: '面板二内容' }] }
    ] });
// F.ui.Accordion1.getActivePaneIndex()；F.ui.Accordion1.activeNextPane()
```
```aspx
<%-- ② Pro（WebForms）：AutoPostBack + OnPaneIndexChanged 服务端切换事件（仅 Pro 有）--%>
<f:Accordion ID="Accordion1" runat="server" IsFluid="true" Title="手风琴" ShowHeader="false" Height="500px"
    ShowBorder="true" ActivePaneIndex="1" EnableCollapse="false" AutoPostBack="true" OnPaneIndexChanged="Accordion1_PaneIndexChanged">
    <Panes>
        <f:AccordionPane ID="AccordionPane1" runat="server" Title="面板一" IconUrl="~/res/images/16/1.png" BodyPadding="2px 5px">
            <Items><f:Label Text="面板一内容" runat="server" /></Items>
        </f:AccordionPane>
        <f:AccordionPane ID="AccordionPane2" runat="server" Title="面板二" BodyPadding="2px 5px">
            <Items><f:Label Text="面板二内容" runat="server" /></Items>
        </f:AccordionPane>
    </Panes>
</f:Accordion>
```
```csharp
// ③ Core-MVC（Fluent API）
@(F.Accordion().IsFluid(true).ID("Accordion1").Title("手风琴").ShowHeader(false).Height(500).ShowBorder(true).ActivePaneIndex(1)
    .Panes(
        F.AccordionPane().ID("AccordionPane1").Title("面板一").IconUrl(Url.Content("~/res/images/16/1.png")).BodyPadding("2 5")
            .Items(F.Label().Text("面板一内容")),
        F.AccordionPane().ID("AccordionPane2").Title("面板二").BodyPadding("2 5").Items(F.Label().Text("面板二内容"))
    ))
```
```html
<!-- ④ Core-TagHelper（RazorForms / RazorPages，写法相同）；RazorForms 可加 OnPaneIndexChanged 服务端事件 -->
<f:Accordion ID="Accordion1" IsFluid="true" Title="手风琴" ShowHeader="false" Height="500" ShowBorder="true" ActivePaneIndex="1"
             OnPaneIndexChanged="Accordion1_PaneIndexChanged">
    <Panes>
        <f:AccordionPane ID="AccordionPane1" Title="面板一" IconUrl="@Url.Content("~/res/images/16/1.png")" BodyPadding="2px 5px">
            <Items><f:Label Text="面板一内容"></f:Label></Items>
        </f:AccordionPane>
    </Panes>
</f:Accordion>
```
```html
<!-- ⑤ FineUIJava（Thymeleaf 方言）：on-pane-index-changed 直接声明服务端事件（客户端 panechange）-->
<f:accordion id="Accordion1" is-fluid="true" title="手风琴控件" show-header="false" height="500" show-border="true" active-pane-index="1"
    enable-collapse="false" on-pane-index-changed="Accordion1_PaneIndexChanged">
    <f:panes>
        <f:accordion-pane id="AccordionPane1" title="面板一" icon-url="~/res/images/16/1.png" body-padding="2px 5px">
            <f:items><f:label id="Label1" text="面板一中的文本"></f:label></f:items>
        </f:accordion-pane>
        <f:accordion-pane id="AccordionPane2" title="面板二" icon-url="~/res/images/16/4.png" body-padding="2px 5px">
            <f:items><f:label id="Label2" text="面板二中的文本"></f:label></f:items>
        </f:accordion-pane>
    </f:panes>
</f:accordion>
```
```java
// FineUIJava 页面类
@FineUIPage("accordion/pane-index-changed")
public class PaneIndexChanged extends FineUIPageBase {
    com.fineui.java.core.controls.Accordion Accordion1;
    public void Page_Load(Object sender, EventArgs e) { }
    public void Accordion1_PaneIndexChanged(Object sender, EventArgs e) {   // 返回 void
        showNotify("当前展开的是第 " + (Accordion1.getActivePaneIndex() + 1) + " 个面板");
    }
    public void Button2_Click(Object sender, EventArgs e) {
        Accordion1.setActivePaneIndex((Accordion1.getActivePaneIndex() + 1) % 3);   // 展开下一个
    }
}
```

## 读取/切换活动面板

```csharp
// Pro / Core-RazorForms —— 控件字段属性
int idx = Accordion1.ActivePaneIndex;
Accordion1.ActivePaneIndex = (idx + 1) % Accordion1.Panes.Count;   // 展开下一个
```
```csharp
// Core-MVC / RazorPages —— 客户端读活动索引回发，UIHelper 设置
// 按钮：.OnClick(Url.Action("btn_Click"), new Parameter("activeIndex", "F.ui.Accordion1.getActivePaneIndex()"))
public IActionResult btn_Click(int activeIndex) {           // RazorPages: OnPostBtn_Click
    UIHelper.Accordion("Accordion1").ActivePaneIndex((activeIndex + 1) % 3);
    return UIHelper.Result();
}
```
```javascript
// F.js
F.ui.Accordion1.getActivePaneIndex();   // 当前展开的面板索引
F.ui.Accordion1.activeNextPane();        // 展开下一个
```
```java
// FineUIJava 页面类 —— Bean 方法读写活动面板
int idx = Accordion1.getActivePaneIndex();
Accordion1.setActivePaneIndex((idx + 1) % 3);   // 展开下一个
```

## 关键差异

- **面板切换服务端事件 `OnPaneIndexChanged`**：**Pro（配 `AutoPostBack="true"`）、Core-RazorForms、Java（`on-pane-index-changed`）都能直接声明**，切换时回发到服务端处理器。Core-MVC / RazorPages 无该声明式事件，改读客户端 `getActivePaneIndex()` 回发，再用 `UIHelper.Accordion(...)` 操作。
- **读/写活动面板**：Pro/RazorForms 用控件字段属性（`Accordion1.ActivePaneIndex`）；**Java 用 Bean 方法 `Accordion1.getActivePaneIndex()` / `Accordion1.setActivePaneIndex(n)`**；MVC/RazorPages 用 `UIHelper.Accordion(...)`。

## See also

- [panel.md](panel.md)：Panel · [tab.md](tab.md)：TabStrip
