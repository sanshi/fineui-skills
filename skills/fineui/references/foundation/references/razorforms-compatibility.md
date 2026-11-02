# Core RazorForms v16.1.1 旧事件兼容

本页旧控件事件继续使用 `Url.Handler(...)`、`OnPost...` 和 `Parameter`，手写 `F.doPostBack` 也可调用本页公开 OnPost；内部仍执行 RazorForms 的状态恢复、模板重跑及后台控件更新。无需为保留这些写法添加 `[NoRazorForms]`。同页可交替使用旧 OnPost 和普通后台控件事件；流式声明本身不代表页面运行在 MVC 模式。

## 请求地址与参数

普通 RF 事件仍请求当前完整 URL；旧 Url.Handler/OnPost 事件使用声明实际生成的地址，不自动追加当前页面的查询参数。需要 id、dialog 等业务参数时，使用显式 URL、Parameter 或提交字段，保持原有绑定来源。

例如页面位于 `/Edit?id=105`，`Url.Handler("Save")` 生成 `/Edit?handler=Save` 时，回发不会自动补入 id。需要编号可显式使用 `Url.Page(null, "Save", new { id = 105 })`，或沿用页面原有 Parameter。

网址由宿主生成后直接使用，框架不会对整个地址重复解码。PathBase 子目录、中文或空格路由值，以及分类值中的编码斜杠 `%2F` 均按实际地址匹配本页 Handler；不要手工解码或再次编码来规避旧版的匹配错误。可在 QueryParameters 的分类路由中验证编码值。

普通 RazorForms 方法名事件只执行当次控件声明的方法，不会因为页面存在 OnPost 方法或 URL 带有 handler 而自动调用它。明确声明的 Url.Handler/Url.Page 事件，以及无控件目标、指向本页公开 OnPost 的手写回发，才由 ASP.NET Core 绑定参数并调用；这些入口共用 RazorForms 状态恢复和响应。

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

严格内容安全策略（CSP）下使用 `Parameter.FromFunction`、`FromData`、`FromEventArgument`。确需按事件位置取附加参数时使用数字下标，例如 `Parameter.FromEventArgument("nodeId", 1)`，不使用字符串形参名称；标准 RF 事件优先从事件对象取值。`readNote` 必须是页面中真实定义的具名函数，每次发送前取当前值。普通策略允许旧字符串表达式；兼容不会自动放宽 CSP。参数名不能以框架保留的 `__` 开头。

## 手写回发调用本页 OnPost

在 `.cshtml` 的页面脚本中，旧字符串重载可继续使用：

```javascript
function onSubmitCompatibilityLogin(event) {
    F.doPostBack('@Url.Handler("Login")', 'SimpleForm1');
    return false;
}
```

也可使用 `F.doPostBack({ url: ..., fields: 'SimpleForm1', params: ... })`。不必伪造控件目标或框架状态字段；服务器仍核对本页公开 POST Handler 与宿主实际选中的方法。有控件目标的旧事件还须匹配本轮控件声明。这里只兼容本页公开 OnPost，不允许把其他页面的 UI 响应应用到当前 RF 页面。OnPost 的身份权限、数据权限与防伪校验继续由宿主和业务代码负责。

## 固定结构的匿名流式控件

固定 Razor 声明中的匿名控件和匿名列可连续回发，无须为纯说明文字或每个固定表格列补 ID。例如：

```csharp
@(F.SimpleForm().ID("SimpleForm1").Items(
    F.Label().Text("填写账号和密码后登录"),
    F.TextBox().ID("tbxUserName").Name("UserName").Label("用户名")))
```

框架按模板中的实际声明位置关联匿名控件，但自动生成的编号不是公开业务接口。需要被脚本、UIHelper 或后台代码引用的控件仍应显式命名；条件、循环和动态增删会改变结构时，使用稳定业务 ID。固定匿名结构与纯动态事件是两项能力：纯动态方法名事件可在首次绑定后按严格签名回退，模板及当轮明确登记仍优先；完整规则见[事件与回发](events-postback.md)。

## 纯动态事件保留首次绑定

Core RF 的 FormDynamic 继续在 Page_Load 的 `!IsPostBack` 分支调用 LoadData，首次为 ddlGender 登记 `new Event("change", "ddlGender_SelectedIndexChanged")`。回发恢复输入后，后台可调用原来的 `protected void ddlGender_SelectedIndexChanged(object sender, EventArgs e)`；不需要特性、重命名或回发重新登记。返回 Task 的异步事件沿用 Core 的 Async 查找规则。

只对本轮没有模板声明、没有明确登记的动态控件启用签名回退。模板事件、服务器明确登记和主动解绑始终优先；模板方法及数据绑定回调、生命周期、框架方法和不兼容签名不能借此调用。条件模板本轮缺席按当前结果判断。符合签名的业务方法成为可请求的入口，权限检查仍写在宿主及业务处理器内；省略重新登记不再表示撤销权限。Java 的对应规则及返回类型区别见[事件与回发](events-postback.md)。

## 回发时追加控件

Core RazorForms/Java 可在按钮事件中向已有容器的 Items 追加控件。Core 使用 `Panel1.Items.Add(control)`，Java 使用 `Panel1.addChild(control, "items")`；为新增按钮绑定事件只需在创建时执行一次，下一轮按纯动态签名规则分派。保持显式稳定 ID，重复点击时先判断控件是否已存在，不重复 new 同名控件。

两端的 FormDynamicPostback 示例在首次点击后创建 tbxRemark 和 btnConfirm，再修改备注连续提交；新控件插入真实面板，其输入和父子归属参与后续状态。已有容器的动态插入目前只支持 Items，不将工具栏、列或菜单等集合套用为相同能力。Java 也支持新根及整棵嵌套子树的创建；数据更新本身不等于创建控件。

## 旧 Listener 与严格 CSP

Core 的 `Listener.Handler="foo()"` 与 `"foo();"` 都表示事件发生时执行的旧函数体，普通策略继续兼容，调用语句不会自动补 `return`。`"return foo();"` 保留返回值；要取消后续处理仍须返回 `false`。严格 CSP 下改成 `Handler="foo"` 或 `"QF.foo"`，并在该具名函数内实现逻辑；不要把调用语句当函数引用。ClickHandler 也只写裸函数名，RendererFunction 的函数工厂能力不因本项兼容改变。

## 业务回调先接管，UI 响应再校验

Core 的顺序是 JSON 语法解析、恢复输入和触发控件、`complete`、`success`、全局成功钩子、完整 UI 准备、应用。页面回调仍以 window 为 `this`，接收原始响应字符串、状态文本和兼容 xhr。

旧请求返回 `{"success":true}` 时，可在现有 `success` 中解析并明确返回 `false`；合法 JSON 数组、标量和 null 也可被 `complete`、`success` 或全局成功钩子接管。未接管的响应必须通过完整 UI 结构和命令校验，非对象根和业务字段仍不能直接当 UI 响应。空正文、HTML 和坏 JSON 在 `complete` 前进入 `parsererror`；晚发现的 UI 格式错误只进入一次错误链，不重跑成功回调。准备失败不应用任何更新，实际 setter 执行失败则使用应用错误提示。回调接管后 RF 丢弃无法安全继续的待发请求，不自动重试保存。

新业务数据接口仍优先使用 `fetch` 或 `jQuery.ajax`，避免混入控件状态流程；无 UI 更新的本页 UI 处理器返回 `UIHelper.Result()`，其空响应为 `{}`，不要返回空正文。

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

仅在 OnGet 查询的列表不会自动重建。模板如果继续循环读取业务列表、决定控件数量或 ID，仍须准备本轮所需数据；不能依赖兜底恢复数据库资料。

旧 RazorPages 的另一种写法也可保留：OnGet 已经把动态字段数组放入 `ViewBag.DynamicItems`，模板只写 `<f:FormRow ID="FormRow1" Items="@ViewBag.DynamicItems" />`。回发该数组为空时只产生空的临时声明，不删除从状态恢复的当前字段；首次绑定的 Url.Handler 事件仍按本页公开 OnPost 核对并执行，不重跑 OnGet，也不要求再创建同名字段。DynamicOnPost 兼容页演示此路径。授权与业务约束仍放在宿主过滤器或处理方法中，不带 RF 字段的独立 POST 沿用原生合同，Page_Init 不是所有接口的统一授权入口。

## 参考示例

Core Examples 的“其他 → 旧写法兼容”提供 FluentLogin、QueryParameters、ModelBinding、SelectionChanged、AsyncUpdate、GridPaging、GridSelection、TreeSelection、TabIndexChanged、MixedEvents、DynamicOnPost 十一页。FluentLogin 使用流式表单、匿名说明标签和手写登录回发；GridPaging 使用流式表格和匿名列；QueryParameters 支持分类路由及编码值；DynamicOnPost 保留 OnGet 动态数组和本页 OnPost。迁移时优先保留项目原有写法，不强制整页改写。

这项兼容仅适用于 Core v16.1.1 及以上。Java 与 Pro 的事件合同不因本文改变。升级时一起替换产品程序集和资源，刷新旧页面；不要自动重试可能已经执行的保存请求。
