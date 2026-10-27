# Calendar 内嵌日历

Calendar 是直接显示在页面中的日期、月份、年份、时间或范围选择面板。需要“输入框 + 弹出日期面板”时使用 `DatePicker`，不要用 Calendar 模拟输入框。

## 基础日期选择

### F.js

```javascript
F.create({
    type: 'Calendar', id: 'Calendar1', renderTo: '#wrap',
    format: 'yyyy-MM-dd', value: new Date(),
    listeners: {
        select: function () {
            showNotify('选择的日期：' + this.getText());
        }
    }
});
```

### FineUI.Pro

```aspx
<f:Calendar ID="Calendar1" runat="server" DateFormatString="yyyy-MM-dd"
    ShowTodayButton="true" EnableDateSelectEvent="true" OnDateSelect="Calendar1_DateSelect" />
```

```csharp
protected void Page_Load(object sender, EventArgs e)
{
    if (!IsPostBack)
    {
        Calendar1.SelectedDate = DateTime.Now;
    }
}

protected void Calendar1_DateSelect(object sender, EventArgs e)
{
    ShowNotify("选择的日期：" + Calendar1.SelectedDate.Value.ToString("yyyy-MM-dd"));
}
```

MVC/RazorPages 的日期参数使用下面的具名函数；在当前页面脚本区声明一次，避免严格 CSP 下编译字符串表达式：

```javascript
function getCalendar1Text() {
    return F.ui.Calendar1.getText();
}
```

### FineUI.Core MVC

```csharp
@(F.Calendar().ID("Calendar1")
    .DateFormatString("yyyy-MM-dd")
    .SelectedDate(DateTime.Now)
    .OnDateSelect(Url.Action("Calendar1_DateSelect"),
        Parameter.FromFunction("selectedDate", "getCalendar1Text")))
```

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Calendar1_DateSelect(string selectedDate)
{
    UIHelper.Label("labResult").Text("选择的日期：" + selectedDate);
    return UIHelper.Result();
}
```

### FineUI.Core RazorForms（推荐）

```html
<f:Calendar ID="Calendar1" DateFormatString="yyyy-MM-dd"
    OnDateSelect="Calendar1_DateSelect"></f:Calendar>
```

```csharp
protected void Page_Load(object sender, EventArgs e)
{
    if (!IsPostBack)
    {
        Calendar1.SelectedDate = DateTime.Now;
    }
}

protected void Calendar1_DateSelect(object sender, EventArgs e)
{
    labResult.Text = "选择的日期：" + Calendar1.SelectedDate.Value.ToString("yyyy-MM-dd");
}
```

### FineUI.Core RazorPages

```html
<f:Calendar ID="Calendar1" DateFormatString="yyyy-MM-dd"
    OnDateSelect="@Url.Handler("Calendar1_DateSelect")"
    OnDateSelectParameter1="@(Parameter.FromFunction("selectedDate", "getCalendar1Text"))">
</f:Calendar>
```

```csharp
public IActionResult OnPostCalendar1_DateSelect(string selectedDate)
{
    UIHelper.Label("labResult").Text("选择的日期：" + selectedDate);
    return UIHelper.Result();
}
```

### FineUI.Java

```html
<f:calendar id="Calendar1" date-format-string="yyyy-MM-dd"
    on-date-select="Calendar1_DateSelect"></f:calendar>
```

```java
protected Calendar Calendar1;
protected Label labResult;

public void Page_Load(Object sender, EventArgs e) {
    if (!isPostBack()) {
        Calendar1.setSelectedDate(LocalDateTime.now());
    }
}

public void Calendar1_DateSelect(Object sender, EventArgs e) {
    DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    labResult.setText("选择的日期：" + format.format(Calendar1.getSelectedDate()));
}
```

Java 页面如果也命名为 `Calendar`，控件字段使用全限定名 `com.fineui.java.core.controls.Calendar`，避免与页面类同名冲突。

## 日期范围

范围面板使用 `DisplayType="DayRange"`，值由开始日期和结束日期组成。月范围与年范围分别使用 `MonthRange`、`YearRange`。

```aspx
<!-- FineUI.Pro -->
<f:Calendar ID="Calendar1" DisplayType="DayRange" DateFormatString="yyyy-MM-dd"
    ShowConfirmButton="true" ConfirmToSelect="true" EnableDateSelectEvent="true"
    OnDateSelect="Calendar1_DateSelect" />
```

```html
<!-- FineUI.Core RazorForms -->
<f:Calendar ID="Calendar1" DisplayType="DayRange" DateFormatString="yyyy-MM-dd"
    ShowConfirmButton="true" ConfirmToSelect="true" OnDateSelect="Calendar1_DateSelect">
</f:Calendar>
```

```html
<!-- FineUI.Core RazorPages -->
<f:Calendar ID="Calendar1" DisplayType="DayRange" DateFormatString="yyyy-MM-dd"
    ShowConfirmButton="true" ConfirmToSelect="true"
    OnDateSelect="@Url.Handler("Calendar1_DateSelect")"
    OnDateSelectParameter1="@(Parameter.FromFunction("selectedRange", "getCalendar1Text"))">
</f:Calendar>
```

```html
<!-- FineUI.Java -->
<f:calendar id="Calendar1" display-type="DayRange" date-format-string="yyyy-MM-dd"
    show-confirm-button="true" confirm-to-select="true"
    on-date-select="Calendar1_DateSelect"></f:calendar>
```

服务端日期范围使用：

| 写法 | 开始值 | 结束值 |
|------|--------|--------|
| Pro / Core RazorForms | `Calendar1.RangeStartDate` | `Calendar1.RangeEndDate` |
| Core MVC / RazorPages | 将 `F.ui.Calendar1.getText()` 作为参数提交 | 在处理器中按 `RangeSeparator` 拆分 |
| FineUI.Java | `Calendar1.getRangeStartDate()` | `Calendar1.getRangeEndDate()` |

`RangeSeparator` / `range-separator` 可以修改范围分隔符。只有需要两个面板互不联动时才设置 `IndependentRangePanels="true"` / `independent-range-panels="true"`。

## 月、年和时间面板

`DisplayType` 的常用值：

| 用途 | `DisplayType` | 建议格式 |
|------|---------------|----------|
| 日期 | `Day` | `yyyy-MM-dd` |
| 月份 | `Month` | `yyyy/MM` |
| 年份 | `Year` | `yyyy` |
| 日期范围 | `DayRange` | `yyyy-MM-dd` |
| 月份范围 | `MonthRange` | `yyyy/MM` |
| 年份范围 | `YearRange` | `yyyy` |
| 时间 | `Time` | `HH:mm:ss` |
| 时间范围 | `TimeRange` | `HH:mm:ss` |

Core/Java 的 `SelectedDate`、`RangeStartDate`、`RangeEndDate` 是当前 `Text/value` 的日期视图，读取按当前格式解析，非法文本返回 `null`。年格式补一月一日，年月补每月第一天，纯时间使用服务器当天；也可直接用 `Text/getText()` 保留格式化文字。Pro 的日期属性仍按对应版本核对，不套用 Core/Java 的内部保存方式。

修改范围一端会保留当前另一端；任一端赋 `null` 清空整个范围。先设置起点、再设置终点时，完整有效后才生成范围文字；未完成的范围只在本请求暂存。修改范围最终记为 `value`，不额外保存两份端点。修改日期格式前先读取当前有效值，再按新格式输出，已被格式舍弃的时分秒不会恢复。

显示日期和时间组合面板时设置 `ShowTime="true"` / `show-time="true"`；`StackDateTime` 控制日期与时间面板是否上下排列。`ShowMinute`、`ShowSecond` 控制面板是否显示分钟和秒，不会截断已有时间值；保存及解析精度由 `DateFormatString` 决定。Core/Java 可在后台动态切换这些开关，以及今天/确定按钮、层叠布局和范围面板独立导航；DatePicker 的弹层同步采用新设置。

## 可选范围与确认按钮

- `MinDate` / `min-date`：最小可选日期。
- `MaxDate` / `max-date`：最大可选日期。
- `ShowTodayButton` / `show-today-button`：显示“今天”按钮。
- `ShowConfirmButton` / `show-confirm-button`：显示确认按钮。
- `ConfirmToSelect` / `confirm-to-select`：只有点击确认按钮才触发最终选择。

日期格式决定显示、提交和服务端解析方式。先设置 `DateFormatString`，再按相同格式处理值，不要混用 .NET、Java 与 F.js 的日期格式 API。
