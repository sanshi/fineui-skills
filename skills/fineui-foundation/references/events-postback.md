# 客户端事件与回发（FineUI v16.0）

先按意图选入口，不要从历史 API 反推写法。

| 意图 | 推荐入口 | 服务端接收 |
|------|----------|------------|
| 只在浏览器执行 | 页面具名函数 + `ClickHandler` / `click-handler` | 无 |
| 控件的标准服务端事件 | `OnClick`、`OnSelectedIndexChanged`、`OnRowCommand` 等 | 对应控件事件方法 |
| 页面级业务动作或需传自定义数据 | `F.customEvent(name, payload)` | `Page_CustomEvent` |
| 需要非 AJAX、loading、命名表单字段或完成回调 | `F.doPostBack({ ... })` | `Page_CustomEvent` 或指定 MVC/RazorPages 地址 |

## 1. 客户端点击：具名函数 + ClickHandler

函数名采用 `on<语义>Click`，首参固定为 `event`。属性里只写裸函数名，不写调用表达式或内联脚本。

```javascript
function onShowWindowClick(event) {
    F.ui.Window1.show();
}
```

```aspx
<%-- Pro（站点已设置 EnableImplicitPostBack="false"） --%>
<f:Button ID="btnShow" runat="server" Text="显示窗体" ClickHandler="onShowWindowClick" />
```

```html
<!-- Core RazorForms / RazorPages -->
<f:Button ID="btnShow" Text="显示窗体" ClickHandler="onShowWindowClick"></f:Button>

<!-- FineUIJava -->
<f:button id="btnShow" text="显示窗体" click-handler="onShowWindowClick"></f:button>
```

```csharp
// Core MVC
@(F.Button().ID("btnShow").Text("显示窗体").ClickHandler("onShowWindowClick"))
```

F.js 直接传函数对象：

```javascript
F.create({ type: 'Button', text: '显示窗体', handler: onShowWindowClick });
```

规则：

- 新代码不使用 `OnClientClick` / `on-client-click`；FineUIJava 不提供该旧入口，Core/Pro 仅为存量项目兼容。
- 不写 `ClickHandler="doSomething();"`，也不把脚本串写进 `<f:Listener Handler>`；一律写具名函数。
- 一个控件只有一个 click 回调时优先 `ClickHandler`，不要同时再声明 click Listener。
- `Button`、`Tool`、菜单项、`LinkButton`、`HyperLink` 和 `TreeNode` 均支持 `ClickHandler`。`TreeNode` 回调签名是 `(event, nodeId)`，其余通常是 `(event)`。
- `HyperLink` 有真实地址；回调返回 `false` 才阻止导航。`LinkButton` 的占位地址由框架同步阻止默认导航，不要把 `href="javascript:;"` 机械替换成会改写 hash 的 `href="#"`。
- 要取消后续确认框、默认导航或服务端回发，必须显式 `return false`；普通 `return` / `undefined` 表示继续。

## 2. 标准服务端事件

声明服务端事件即可表达“点击后回发”：

```aspx
<%-- Pro --%>
<f:Button ID="btnSave" runat="server" Text="保存" OnClick="btnSave_Click" />
```

```html
<!-- Core RazorForms -->
<f:Button ID="btnSave" Text="保存" OnClick="btnSave_Click"></f:Button>

<!-- Core RazorPages -->
<f:Button ID="btnSave" Text="保存" OnClick="@Url.Handler(&quot;btnSave_Click&quot;)"></f:Button>

<!-- FineUIJava -->
<f:button id="btnSave" text="保存" on-click="btnSave_Click"></f:button>
```

```csharp
// Core MVC
@(F.Button().Text("保存").OnClick(Url.Action("btnSave_Click")))
```

简单确认优先声明 `ConfirmText` / `ConfirmTarget`，不要手写 `F.confirm`：

```aspx
<f:Button ID="btnDelete" runat="server" Text="删除"
    ConfirmText="确定删除？" ConfirmTarget="Top" OnClick="btnDelete_Click" />
```

## 3. FineUIPro 的两个兼容开关

FineUIPro 为老项目保留两个默认值为 `true` 的兼容开关。官方示例、空项目和新应用推荐统一设为 `false`：

```xml
<FineUIPro EnableImplicitPostBack="false"
           EnableImplicitChangeEvents="false"
           AllowDangerousRawTag="false"
           AllowDangerousScriptTag="false" />
```

### EnableImplicitPostBack=false

- 未显式设置控件自身回发属性时，只有声明对应服务端事件才自动回发。
- 纯客户端按钮只写 `ClickHandler`，不用重复写 `EnablePostBack="false"`。
- `OnClick` 等服务端事件会自动推导回发，不用重复写 `EnablePostBack="true"`。
- 显式 `EnablePostBack`、`EnableTrigger1PostBack`、`EnableTrigger2PostBack` 始终优先。
- 作用于 `Button`、`MenuButton`、`Tool`、`LinkButton`、`TriggerBox`、`TwinTriggerBox` 两个触发器和 `LinkButtonField`；不影响变化事件的 `AutoPostBack` 或 Grid 自身功能。

### EnableImplicitChangeEvents=false

- 未显式设置 `AutoPostBack` 时，声明 `TextChanged`、`SelectedIndexChanged`、`CheckedChanged` 等服务端变化事件即可自动回发。
- 其他控件发起回发时仍同步字段当前值，但不会因值与旧状态不同而连带触发该字段的变化事件。
- 显式 `AutoPostBack="true"` / `"false"` 始终优先。
- 涉及文本与选择字段、复选框、单选框、列表、`MenuCheckBox`、`FileUpload`、`Accordion`、`TabStrip` 和 `CheckBoxField`。`HtmlEditor.TextChanged` 只收口连带触发，不自动推导回发。

因此推荐模式下写：

```aspx
<f:DropDownList ID="ddlSchool" runat="server"
    OnSelectedIndexChanged="ddlSchool_SelectedIndexChanged" />
```

不再为了触发已声明事件而重复添加 `AutoPostBack="true"`。

## 4. 页面级自定义回发：F.customEvent + Page_CustomEvent

客户端产生了控件标准事件无法表达的业务动作，或要传对象/数组时，使用：

```javascript
function onDeleteSelectedClick(event) {
    var grid = F.ui.Grid1;
    if (!grid.hasSelection()) {
        F.alert('请先选择要删除的用户');
        return;
    }

    var rowIds = grid.getSelectedRows();
    F.confirm({
        message: F.rawHtml('确定删除选中的 <strong>' + rowIds.length + '</strong> 个用户？'),
        target: '_top',
        ok: function () {
            F.customEvent('DeleteSelectedUsers', { rowIds: rowIds });
        }
    });
}
```

C#（Pro / Core RazorForms）：

```csharp
protected void Page_CustomEvent(object sender, CustomEventArgs e)
{
    if (e.EventName == "DeleteSelectedUsers")
    {
        JArray rowIds = e.EventArgumentsAsJObject.Value<JArray>("rowIds");
        // 按 rowIds 执行业务操作
    }
}
```

Java：

```java
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("DeleteSelectedUsers".equals(e.getEventName())) {
        // 使用项目统一配置的 Jackson ObjectMapper 解析 JSON 参数。
        JsonNode payload = objectMapper.readTree(e.getArgument());
        JsonNode rowIds = payload.path("rowIds");
        // 按 rowIds 执行业务操作
    }
}
```

多参数使用 JSON 对象，批量标识使用数组；不要自己用 `$`、`,`、`#` 拼接后再拆分。业务页面不要直接调用 `__doPostBack`，也不要在 `Page_Load` 中读取原始 `__EVENTARGUMENT` 分派业务。

Pro 的 `PageManager.CustomEvent` / ASPX `OnCustomEvent` 与 `GetCustomEventReference` 已废弃，只为老项目保留运行兼容。新代码只写页面约定方法 `Page_CustomEvent`。

## 5. F.doPostBack 的适用边界

普通页面级业务优先 `F.customEvent`。只有需要完整回发选项时使用 `F.doPostBack(options)`。可用选项随产品不同，下面是 Pro 的加载提示与完成回调示例：

```javascript
F.doPostBack({
    eventName: 'ExportRows',
    eventArgument: { rowIds: F.ui.Grid1.getSelectedRows() },
    enableAjaxLoading: true,
    ajaxLoadingText: '正在导出...',
    complete: function () {
        showNotify('处理完成');
    }
});
```

Core 需要把值绑定到命名表单字段时可用 `fields` / `params`；Core MVC / RazorPages 需要请求明确 action/handler 时可传 `url`。Pro 支持 `enableAjax`、加载提示及 `success` / `error` / `complete`，但不支持 Core 专有的 `url` / `fields` / `params`。Java 推荐直接用 `F.customEvent`；兼容的 `F.doPostBack` 只使用 `eventName` / `eventArgument` / `complete`。不要把 RazorPages 的 `@Url.Handler(...)` 形态套到 RazorForms、Pro 或 Java。

## 6. Grid 行命令

新代码用 `RenderField.Commands`，不要再以 `LinkButtonField.OnClientClick` 或手写行内按钮拼脚本。`Command` 只属于 `RenderField`，`LinkButtonField` / `WindowField` 不支持该子标签。

```aspx
<f:Grid ID="Grid1" runat="server" OnRowCommand="Grid1_RowCommand">
    <Columns>
        <f:RenderField ColumnID="Actions" HeaderText="操作">
            <Commands>
                <f:Command CommandName="Edit" IconFont="_Pencil" ToolTip="编辑" />
                <f:Command CommandName="Delete" IconFont="_Close" ToolTip="删除"
                    ConfirmText="确定删除本行？" ConfirmTarget="Top" />
            </Commands>
        </f:RenderField>
    </Columns>
</f:Grid>
```

Core RazorForms 使用相同标签结构；Java 对应 `<f:render-field><f:commands><f:command ...>`。Pro 与 Java 的服务端参数为 `GridCommandEventArgs`，Core RazorForms 为 `GridRowCommandEventArgs`。

客户端附加逻辑监听 Grid 的 `rowcommand`。监听返回 `false` 可阻止随后默认的服务端 RowCommand 回发。

## 7. 普通控件回发参数

Pro、Core RazorForms 与 Java 的普通控件回发参数都以 F.js 客户端事件名开头，例如 `click`、`change`、`rowcommand$...`。这是框架协议，不是业务自定义事件的替代品。

FineUIJava 普通控件回发不把后台方法名放进 URL；服务端根据本次模板和 `Page_Load` 重建的事件映射分派。回发保留当前页面的查询字符串，因此页面类仍可通过 `getQueryParam()` 读取首次 URL 参数。业务代码不应依赖或自行构造内部控件回发参数。
