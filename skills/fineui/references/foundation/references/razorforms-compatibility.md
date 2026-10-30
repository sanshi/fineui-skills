# Core RazorForms v16.1.1 旧事件兼容

本页旧控件事件继续使用 `Url.Handler(...)`、`OnPost...` 和 `Parameter`，内部仍执行 RazorForms 的状态恢复、模板重跑及后台控件更新。无需为保留这些写法添加 `[NoRazorForms]`。同页可交替使用旧 OnPost 和普通后台控件事件；流式声明本身不代表页面运行在 MVC 模式。

## 请求地址与参数

普通 RF 事件仍请求当前完整 URL；旧 Url.Handler/OnPost 事件使用声明实际生成的地址，不自动追加当前页面的查询参数。需要 id、dialog 等业务参数时，使用显式 URL、Parameter 或提交字段，保持原有绑定来源。

例如页面位于 `/Edit?id=105`，`Url.Handler("Save")` 生成 `/Edit?handler=Save` 时，回发不会自动补入 id。需要编号可显式使用 `Url.Page(null, "Save", new { id = 105 })`，或沿用页面原有 Parameter。

普通 RazorForms 方法名事件只执行当次控件声明的方法，不会因为页面存在 OnPost 方法或 URL 带有 handler 而自动调用它。只有 Url.Handler/Url.Page 等明确声明本页 OnPost 的入口才由 ASP.NET Core 绑定参数并调用；两类入口共用 RazorForms 状态恢复和响应。

## 按事件入口选择签名

```html
<f:Button ID="btnSave" Text="保存" OnClick="@Url.Handler("Save")"
    OnClickParameter1="@(Parameter.FromFunction("note", "readNote"))" />
<f:Button ID="btnAppend" Text="追加" OnClick="btnAppend_Click" />
```

```csharp
public IActionResult OnPostSave(string note)
{
    labResult.Text = note;
    return UIHelper.Result();
}

public void btnAppend_Click(object sender, EventArgs e)
{
    labResult.Text += "，已确认";
}
```

`labResult` 是当前页生成并绑定的控件字段。原生 Handler 保留 ASP.NET Core 参数绑定、ModelState、异步等待及过滤器。使用 `UIHelper.Result()` 返回界面结果；不要把本页 UI 事件改成返回业务 JSON、上传或下载。那些独立接口仍按原有请求方式调用。

严格内容安全策略（CSP）下使用 `Parameter.FromFunction`、`FromData`、`FromEventArgument`。`readNote` 必须是页面中真实定义的具名函数，每次发送前取当前值。普通策略允许旧字符串表达式；兼容不会自动放宽 CSP。参数名不能以框架保留的 `__` 开头。

## UIHelper 的边界

在 RazorForms 中推荐直接操作后台控件实例。为兼容旧页面，允许调用 UIHelper 控件更新方法，但这些调用不保证同步到服务器控件属性或按 RazorForms 持久属性恢复。需要下一轮继续读取的状态，使用后台控件属性赋值；不要用 `UIHelper.Label("labResult").Text(...)` 后再假定 `labResult.Text` 已改变。

## OnGet 和绑定对象

```csharp
[BindProperty]
public OrderInput Data { get; set; }

public void OnGet(int id)
{
    Data = new OrderInput { Name = $"订单{id}" };
}
```

回发不执行 OnGet。宿主先绑定请求输入，模板重跑前只为仍为空、允许绑定的普通复杂对象调用公共无参构造；非空对象原样保留，不清空 ModelState。可写属性及公共无参构造是这项兜底的条件，构造函数不应查询数据库。Uri 等标量、集合、BindNever、服务注入和请求体对象不参与兜底，也不会递归创建子对象。

仅在 OnGet 查询的列表不会自动重建。模板如果用列表循环生成控件、决定事件或 ID，仍须确保每次声明稳定；不能依赖兜底恢复数据库资料。授权与业务约束须放在宿主授权过滤器或实际处理方法中：不带 RF 字段的独立 POST 沿用原生合同，Page_Init 不是所有接口的统一授权入口。

## 参考示例

Core Examples 的“其他 → 旧写法兼容”提供 FluentLogin、QueryParameters、ModelBinding、SelectionChanged、AsyncUpdate、GridPaging、GridSelection、TreeSelection、TabIndexChanged、MixedEvents 十页。只有 FluentLogin 使用流式声明，其余使用标签。迁移时优先保留项目原有写法，不强制整页改写。

这项兼容仅适用于 Core v16.1.1 及以上。Java 与 Pro 的事件合同不因本文改变。升级时一起替换产品程序集和资源，刷新旧页面；不要自动重试可能已经执行的保存请求。
