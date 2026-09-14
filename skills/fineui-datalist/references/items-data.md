# DataList 列表项与数据

## 静态列表项

DataListItem 的常用属性：`Text`、`Value`、`EnableSelect`、`Selected`、`Group`、`NavigateUrl`、`Target`、`ShowArrow`。Java 标签将属性名转为 kebab-case。

```html
<!-- FineUICore RazorForms / RazorPages -->
<f:DataList ID="DataList1" EnableSelect="true">
    <f:DataListItem Text="中国" Value="cn" EnableSelect="true" />
    <f:DataListItem Text="FineUI 官网" NavigateUrl="https://fineui.com/" Target="_blank" ShowArrow="true" />
</f:DataList>
```

```html
<!-- FineUIJava -->
<f:data-list id="DataList1" enable-select="true">
    <f:data-list-item text="中国" value="cn" enable-select="true"></f:data-list-item>
    <f:data-list-item text="FineUI 官网" navigate-url="https://fineui.com/" target="_blank" show-arrow="true"></f:data-list-item>
</f:data-list>
```

## F.js 数据结构

F.js 先声明字段，再按相同顺序提供数组数据：

```javascript
F.create({
    type: 'DataList', id: 'DataList1', renderTo: '#wrap',
    fields: ['text', 'value', 'enabled', 'group', 'href', 'hrefTarget', 'arrow'],
    data: [
        ['中国', 'cn', true, '亚洲', null, null, false],
        ['FineUI 官网', null, false, '链接', 'https://fineui.com/', '_blank', true]
    ]
});
```

字段名必须与 DataList 的真实数据键一致，不要把 C# 的 PascalCase 属性名直接写进 `fields`。

## C# 服务端数据绑定

Pro、Core RazorForms 可以设置 `DataSource` 和字段映射后调用 `DataBind()`：

```csharp
DataList1.DataTextField = "Name";
DataList1.DataValueField = "Id";
DataList1.DataGroupField = "Region";
DataList1.DataEnableSelectField = "Enabled";
DataList1.DataSource = rows;
DataList1.DataBind();
```

Core MVC / RazorPages 优先在首次渲染时通过 Fluent API 或标签的 `DataSource` 提供数据；回发处理器只返回需要更新的控件状态。

## FineUIJava 服务端添加列表项

Java 使用控件字段清空并添加列表项：

```java
protected DataList DataList1;

private void loadData() {
    DataList1.clearData();
    DataList1.addItem("中国", "cn", true, false, "亚洲", null, null, false);
    DataList1.addItem("美国", "us", true, false, "美洲", null, null, false);
}
```

参数依次为文本、值、是否可选、是否选中、分组、导航地址、目标窗口、是否显示箭头。没有的可选值传 `null`。

## 图文列表与 RawHtml

普通 `Text` 会做 HTML 编码。图片、标题和说明组合成一个列表项时，使用各栈的可信 HTML 类型：

```csharp
// Pro / Core
item.TextRawHtml = new RawHtml(
    "<img class='item-img' src='{0}'><span>{1}</span>",
    System.Net.WebUtility.HtmlEncode(imageUrl),
    System.Net.WebUtility.HtmlEncode(name));
```

```java
// FineUIJava
DataList1.addItem(
    new RawHtml("<img class='item-img' src='%s'><span>%s</span>",
        htmlEncode(imageUrl), htmlEncode(name)),
    id, true, false, null, null, null, false);
```

RawHtml 只声明外层模板可信，不会自动保证插入值安全。来自数据库、请求或用户输入的文字必须先做 HTML 编码。

## 加载更多

首次进入页面时清空并加载第一批；按钮回发时只追加下一批：

```csharp
// FineUIPro：参数是与初始绑定相同结构的数据源
DataList1.AppendData(nextDataSource);
```

```csharp
// FineUICore RazorForms：参数是这一批 DataListItem
DataList1.AppendData(nextItems);
```

```java
List<Map<String, Object>> items = new ArrayList<>();
String template = "<img class='item-img' src='%s'><span>%s</span>";
items.add(DataList1.createItem(new RawHtml(template, htmlEncode(imageUrl), htmlEncode(name)),
        id, true, null, null, null, false));
DataList1.appendData(items);
```

Java 的 `RawHtml` 格式模板使用 `String.format` 语法，因此占位符写 `%s`；`template` 必须是代码中的可信常量，动态插入的属性值和文字仍需先编码。

F.js 客户端使用 `F.ui.DataList1.appendData(data)`。追加的数据字段结构必须与初始数据一致。
