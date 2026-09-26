# 消息框（Alert / Confirm / Prompt / Notify）

四类：**Alert**（提示对话框，需点击关闭）、**Confirm**（确认/取消）、**Prompt**（输入对话框）、**Notify**（自动消失的通知）。

> 消息框在 C# 服务端与 JS 客户端调用方式基本一致；跨写法的唯一差异是**触发按钮怎么绑事件**（MVC `OnClick(Url.Action)` / RazorForms `OnClick="方法名"` / RazorPages `OnClick="@Url.Handler(...)"` / **Java `on-click="方法名"`**）。
>
> **FineUI.Java 服务端入口在消息类上**：`Alert.Show(...)`→`Alert.show(...)`，`Confirm.Show(...)`→`Confirm.show(...)`，`Prompt.Show(...)`→`Prompt.show(...)`；四类复杂配置都可创建实例再调用 `show()`，`Notify` 与 Core 一样只提供实例显示。Java 的 `FineUIPageBase` 不提供 `showAlert` 等消息方法。官方示例自己的 `PageBase.showNotify` 仅统一通知样式。可信 HTML 用 `RawHtml` 显式声明。**客户端 `F.alert`/`F.confirm`/`F.notify` 四栈完全相同**。

## 图标值（MessageBoxIcon）

`None` / `Information` / `Warning` / `Error` / `Success` / `Question`。F.js 用小写字符串：`'information'`/`'warning'`/`'error'`/`'success'`/`'question'`。Java 用 `MessageBoxIcon` 枚举（`MessageBoxIcon.Warning` 等，同 C#）。

## 一、Alert 对话框

### 服务端（Pro / Core 三模式一致）

```csharp
Alert.Show("操作成功！");                              // 默认 Information 图标
Alert.Show("请先选择一行！", MessageBoxIcon.Warning);   // 带图标
Alert.ShowInTop("保存成功！", MessageBoxIcon.Success);  // iframe 内推荐：显示在顶层页面

// 完整对象形式
Alert alert = new Alert { Message = "内容", Title = "标题",
    MessageBoxIcon = MessageBoxIcon.Information, Target = Target.Top, Width = 300, EnableClose = false };
alert.Show();
```
```java
// FineUI.Java —— 与 Core 对应的类级入口，默认 Information 图标
Alert.show("操作成功！");
Alert.show("请先选择一行！", null, MessageBoxIcon.Warning);
Alert.showInTop("保存成功！", null, MessageBoxIcon.Success);

// 完整属性以及「确定后关闭窗体并回发父页」使用消息实例与结构化后续命令
Alert alert = new Alert();
alert.setMessage("保存成功！");
alert.setTarget(Target.Top);
alert.setOkCommand(ActiveWindow.hidePostBackReference("已保存"));
alert.show();
```

### 客户端（F.js，或 C# 页面内 JS）

```javascript
F.alert('简单提示');
F.alert({ message: '出错了', title: '错误', messageIcon: 'error' });
top.F.alert('在顶层弹出');                 // iframe 内跨到顶层
```

## 二、Confirm 确认框

### 客户端 F.confirm（推荐，带确认回调）

```javascript
F.confirm({
    message: '确定要删除吗？',
    target: '_top',                       // iframe 内在顶层弹
    ok: function () { doDelete(); },       // 点确定
    cancel: function () { /* 点取消（可选） */ }
});
```

### 自定义按钮确认（F.create MessageBox）

```javascript
F.create({
    type: 'MessageBox',
    title: '确认退出', message: '尚未保存，确定退出？',
    buttons: [{ buttonId: 'ok', text: '直接退出' }, { buttonId: 'cancel', text: '不退出' }],
    handler: function (event, buttonId) {
        if (buttonId === 'ok') { F.customEvent('ConfirmOK'); }
    }
});
```

### Pro / Core / Java —— 声明式确认优先

```aspx
<%-- Pro：简单确认直接声明 --%>
<f:Button ID="btnDelete" runat="server" Text="删除"
    ConfirmText="确定删除？" ConfirmTarget="Top" OnClick="btnDelete_Click" />
```

Core RazorForms 使用相同属性名但不写 `runat="server"`；Java 改为 kebab-case：`confirm-text` / `confirm-target` / `on-click`。

需要确认/取消进入不同业务分支时，在 `ClickHandler` 指向的具名函数中调用 `F.confirm`，回调里分别调用 `F.customEvent(...)`；后台统一由 `Page_CustomEvent` 按 `EventName` 分派。不要用 `Confirm.GetShowReference` + `OnClientClick` 生成脚本串。

### FineUI.Java —— 三种「先确认再操作」

```html
<!-- ① 声明式 confirm-text（点击先弹确认，确认后才回发 on-click 处理器）—— 同 Core -->
<f:button text="操作一" confirm-text="确认执行操作一？" confirm-target="Top" on-click="btnOperation1_Click"></f:button>
```
```javascript
// ② 客户端 F.confirm 的 ok 回调里用 F.customEvent 触发后台事件（同 F.js，Java 客户端不改）
F.confirm({ message: '确认执行操作二？', messageIcon: 'question',
    ok: function () { F.customEvent('Operation2'); },
    cancel: function () { F.customEvent('Operation2_cancel'); } });   // 取消也可回发
```
```java
// ③ 后台：确认按钮直接进 on-click 处理器；F.customEvent 统一进 Page_CustomEvent，按事件名分派
public void btnOperation1_Click(Object sender, EventArgs e) {
    Notify notify = new Notify();
    notify.setMessage("执行了操作一！");
    notify.show();
}
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("Operation2".equals(e.getEventName())) {
        Notify notify = new Notify();
        notify.setMessage("执行了操作二！");
        notify.show();
    }
}
// 服务端也可只显示确认框：Confirm.show("确定要删除吗？")
```

## 三、Prompt 输入框

Core 可用 `Prompt.Show(...)`，Java 可用 `Prompt.show(...)`；需要接收输入值时，Java 创建 `Prompt` 实例，设置页面脚本中已定义的全局函数名，确定回调的首参是输入值：

```java
Prompt prompt = new Prompt();
prompt.setMessage("请输入名称");
prompt.setOkFunction("onPromptAccepted");
prompt.show();
```

回调参数是函数名，不传 `onPromptAccepted()` 或任意脚本串。需不同的输入类型、默认值、必填或多行输入时，继续设置实例属性。

## 四、Notify 通知框（自动消失）

### 服务端（Pro / Core，`ShowNotify` 便捷方法）

```csharp
ShowNotify("这是一条通知");
ShowNotify("成功登录！", MessageBoxIcon.Success);
ShowNotify("用户名或密码错误！", MessageBoxIcon.Error);

// 完整对象形式：位置、停留时长、进度条
Notify notify = new Notify { Message = "内容", Title = "标题", ShowHeader = true,
    MessageBoxIcon = MessageBoxIcon.Information, Target = Target.Top,
    DisplayMilliseconds = 3000,          // 0 = 不自动消失
    PositionX = Position.Center, PositionY = Position.Top, IsModal = false };
notify.Show();
```
```java
// FineUI.Java —— 与 Core 一样配置实例后显示
Notify notify = new Notify();
notify.setMessage("成功登录！");
notify.setMessageBoxIcon(MessageBoxIcon.Success);
notify.setTarget(Target.Top);
notify.setPositionX(Position.Center);
notify.setPositionY(Position.Top);
notify.setDisplayMilliseconds(3000);
notify.setShowHeader(false);
notify.show();

// 官方示例的 PageBase 另有 showNotify("文本") 便捷方法，内部按上面的方式配置实例。
// HTML 消息用 notify.setMessageRawHtml(new RawHtml("<b>已保存</b>"))。
```

### 客户端（F.js）

```javascript
showNotify('这是一条通知');                             // 示例站点封装
F.notify({ message: '添加成功！', messageIcon: 'information', target: '_top',
    displayMilliseconds: 3000, positionX: 'center', positionY: 'top', header: false });
```

## 关键约束

1. **iframe 内的消息要跨到顶层**：Alert 用 `Alert.ShowInTop(...)`（C#）/ `Alert.showInTop(...)`（Java）或 `top.F.alert(...)`；Confirm 可用同名类级入口，Notify 实例设 `Target.Top`。否则消息只显示在小 iframe 框里。
2. **消息内容含 HTML**：默认转义。要输出可信 HTML 用 `ShowNotify(new RawHtml("..."))`（C#）/ `setMessageRawHtml(new RawHtml("..."))`（Java 实例）或 `F.rawHtml(...)`（JS）——见 `fineui-foundation` 的 rawhtml.md。**用户输入不要声明为可信**。
3. **确认框的“确认后动作”**：普通服务端按钮优先声明 `ConfirmText` / `confirm-text`；需要自定义分支时，用页面具名 `ClickHandler` 调 `F.confirm`，在 `ok` / `cancel` 回调里调用 `F.customEvent(...)`，后台进入 `Page_CustomEvent`。`F.doPostBack(options)` 只留给非 AJAX、loading、命名表单字段或完成回调等完整选项场景。

## See also

- [window.md](window.md)：Window 弹窗
- `fineui-foundation` 的 rawhtml.md：消息里的可信 HTML
