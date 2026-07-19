# Accordion 手风琴

一组竖直排列的面板，同一时刻展开一个。容器 `<f:Accordion>`（`Title`/`ShowHeader`/`Height`/`ShowBorder`/`ActivePaneIndex`/`EnableCollapse`），面板放 `<Panes>` 里的 `<f:AccordionPane>`（`Title`/`IconUrl`/`BodyPadding`/`Collapsed`，内容用 `<Items>`）。

> **注意**：手风琴面板用 `<Panes>` / `AccordionPane`（不是 TabStrip 的 `<Tabs>`/`Tab`）。

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
<!-- ④ Core-TagHelper（RazorForms / RazorPages，写法相同）-->
<f:Accordion ID="Accordion1" IsFluid="true" Title="手风琴" ShowHeader="false" Height="500" ShowBorder="true" ActivePaneIndex="1">
    <Panes>
        <f:AccordionPane ID="AccordionPane1" Title="面板一" IconUrl="@Url.Content("~/res/images/16/1.png")" BodyPadding="2px 5px">
            <Items><f:Label Text="面板一内容"></f:Label></Items>
        </f:AccordionPane>
    </Panes>
</f:Accordion>
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

## 关键差异

- **面板切换服务端事件（`OnPaneIndexChanged` + `AutoPostBack`）只有 Pro 有**；Core 三模式读客户端 `getActivePaneIndex()`，再用 `UIHelper.Accordion(...)`（MVC/RazorPages）或控件字段（RazorForms）操作。

## See also

- [panel.md](panel.md)：Panel · [tab.md](tab.md)：TabStrip
