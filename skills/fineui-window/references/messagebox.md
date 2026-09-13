# 消息框（Alert / Confirm / Notify）

三类：**Alert**（确认对话框，需点击关闭）、**Confirm**（确认/取消）、**Notify**（自动消失的通知）。

> 消息框在 C# 服务端与 JS 客户端调用方式基本一致；跨写法的唯一差异是**触发按钮怎么绑事件**（MVC `OnClick(Url.Action)` / RazorForms `OnClick="方法名"` / RazorPages `OnClick="@Url.Handler(...)"` / **Java `on-click="方法名"`**）。
>
> **FineUIJava 服务端方法名首字母小写**：`Alert.Show(...)`→`showAlert(...)`、`ShowNotify(...)`→`showNotify(...)`、`Confirm`→`showConfirm(...)`；均是 `FineUIPageBase` 上的方法，处理器里直接调用。可信 HTML 变体加 `Raw` 后缀（`showAlertRaw`/`showNotifyRaw`/`showConfirmRaw`）。**客户端 `F.alert`/`F.confirm`/`F.notify` 四栈完全相同**。

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
// FineUIJava —— FineUIPageBase 方法（处理器里直接调用）
showAlert("操作成功！");                                 // 无标题无图标
showAlert("请先选择一行！", null, MessageBoxIcon.Warning); // 带图标
showAlertInTop("保存成功！", null, MessageBoxIcon.Success);// iframe 内推荐：弹到顶层
// 子页「提示 → 确定后关闭窗体并带参回发父页」：showAlertInTopHidePostBack(msg, title, icon, closeArg)
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

### FineUIJava —— 三种「先确认再操作」

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
public void btnOperation1_Click(Object sender, EventArgs e) { showNotify("执行了操作一！"); }
public void Page_CustomEvent(Object sender, CustomEventArgs e) {
    if ("Operation2".equals(e.getEventName())) { showNotify("执行了操作二！"); }
}
// 服务端也可只“显示”确认框：showConfirm("确定要删除吗？")
```

## 三、Notify 通知框（自动消失）

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
// FineUIJava —— FineUIPageBase 方法
showNotify("这是一条通知");
showNotify("成功登录！", MessageBoxIcon.Success);
showNotify("提示", "标题", MessageBoxIcon.Information);  // 带标题头
showNotifyRaw("<ul><li>含 HTML 列表的通知</li></ul>");   // 可信 HTML 变体
// 位置/停留时长/进度条等完整参数用 showNotify(...) 的长参重载
```

### 客户端（F.js）

```javascript
showNotify('这是一条通知');                             // 示例站点封装
F.notify({ message: '添加成功！', messageIcon: 'information', target: '_top',
    displayMilliseconds: 3000, positionX: 'center', positionY: 'top', header: false });
```

## 关键约束

1. **iframe 内的消息要跨到顶层**：Alert 用 `Alert.ShowInTop(...)`（C#）/ `showAlertInTop(...)`（Java）或 `top.F.alert(...)`；Confirm/Notify 用 `target: '_top'`。否则消息只显示在小 iframe 框里。
2. **消息内容含 HTML**：默认转义。要输出可信 HTML 用 `ShowNotify(new RawHtml("..."))`（C#）/ `showNotifyRaw(...)`（Java）或 `F.rawHtml(...)`（JS）——见 `fineui-foundation` 的 rawhtml.md。**用户输入不要声明为可信**。
3. **确认框的“确认后动作”**：普通服务端按钮优先声明 `ConfirmText` / `confirm-text`；需要自定义分支时，用页面具名 `ClickHandler` 调 `F.confirm`，在 `ok` / `cancel` 回调里调用 `F.customEvent(...)`，后台进入 `Page_CustomEvent`。`F.doPostBack(options)` 只留给非 AJAX、loading、命名表单字段或完成回调等完整选项场景。

## See also

- [window.md](window.md)：Window 弹窗
- `fineui-foundation` 的 rawhtml.md：消息里的可信 HTML
