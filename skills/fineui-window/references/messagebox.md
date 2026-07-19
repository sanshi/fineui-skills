# 消息框（Alert / Confirm / Notify）

三类：**Alert**（确认对话框，需点击关闭）、**Confirm**（确认/取消）、**Notify**（自动消失的通知）。

> 消息框在 C# 服务端与 JS 客户端调用方式基本一致；跨写法的唯一差异是**触发按钮怎么绑事件**（MVC `OnClick(Url.Action)` / RazorForms `OnClick="方法名"` / RazorPages `OnClick="@Url.Handler(...)"`）。

## 图标值（MessageBoxIcon）

`None` / `Information` / `Warning` / `Error` / `Success` / `Question`。F.js 用小写字符串：`'information'`/`'warning'`/`'error'`/`'success'`/`'question'`。

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
        if (buttonId === 'ok') { F.doPostBack('/Xxx/ConfirmOK'); }   // 回发到服务端
    }
});
```

### Pro —— Confirm.GetShowReference（绑在按钮 OnClientClick，确认后回发）

```csharp
// Page_Load 内：确认后回发按钮事件，取消不操作
btn.OnClientClick = Confirm.GetShowReference("确认执行？", String.Empty, MessageBoxIcon.Question,
    btn.GetPostBackEventReference(), String.Empty);
// 确认/取消分别回发不同参数：cancelScript 传 btn.GetPostBackEventReference("Cancel")
// 后台用 GetRequestEventArgument() 区分：if (arg == "Cancel") ...
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

### 客户端（F.js）

```javascript
showNotify('这是一条通知');                             // 示例站点封装
F.notify({ message: '添加成功！', messageIcon: 'information', target: '_top',
    displayMilliseconds: 3000, positionX: 'center', positionY: 'top', header: false });
```

## 关键约束

1. **iframe 内的消息要跨到顶层**：Alert 用 `Alert.ShowInTop(...)`（C#）或 `top.F.alert(...)`；Confirm/Notify 用 `target: '_top'`。否则消息只显示在小 iframe 框里。
2. **消息内容含 HTML**：默认转义。要输出可信 HTML 用 `ShowNotify(new RawHtml("..."))`（C#）或 `F.rawHtml(...)`（JS）——见 `fineui-foundation` 的 rawhtml.md。**用户输入不要声明为可信**。
3. **确认框的“确认后动作”**：F.js/客户端用 `ok` 回调或按钮 handler 里 `F.doPostBack`；Pro 用 `Confirm.GetShowReference` 把确认脚本设为按钮回发。

## See also

- [window.md](window.md)：Window 弹窗
- `fineui-foundation` 的 rawhtml.md：消息里的可信 HTML
