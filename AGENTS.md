# AGENTS.md

## 语言与写作

- 文档、技能说明和代码注释使用中文；代码标识符遵循对应语言惯例。
- 面向用户生成的示例应通俗、可直接运行，优先展示 FineUIPro、FineUICore 与 FineUIJava 三栈共有的控件和推荐写法。

## 以源码确认公开 API

- 不得根据文件名、类名或搜索结果推断某个类型是用户可直接使用的控件、表格列或推荐 API。把握不准时，必须查看类声明、继承关系、公开 API 和官方示例；本机开发时以相邻 `D:\FineUI` 仓库的源码为准。
- 抽象基类、内部辅助类、TagHelper、Extension、AjaxHelper 和事件参数类不能当作独立控件介绍。例如 `SelectionListField` 仅在 FineUIJava 中作为 `CheckBoxList` 与 `RadioButtonList` 的抽象父类，FineUIPro 与 FineUICore 的这两个控件直接继承 `Field`；`ToolTipField` 是 `Label`、`LinkButton`、`HyperLink`、`Image` 等控件的抽象父类。两者都不是用户直接声明的控件，更不是表格列。写继承关系时必须注明适用栈，不能把某一栈的实现扩大为三栈共同结论。
- 新增或修改技能前，至少核对 FineUIPro、FineUICore 与 FineUIJava 的对应实现。只有部分栈具备的能力必须明确标注边界，不能写成三栈共有能力。

## 技能覆盖边界

- 技能以 FineUIPro、FineUICore、FineUIJava 三栈共同的控件和特性为主，F.js 作为共同的客户端运行时同步说明。
- `PageLoading`、`Timer`、`UserControlConnector` 等仅 FineUIPro 提供的控件不纳入通用技能。除非任务明确要求 Pro 专属兼容说明，否则不要添加相关内容。
- FineUIPro 的服务端渲染表格列，如 `BoundField`、`TemplateField`、`CheckBoxField`、`HyperLinkField`、`ImageField`、`LinkButtonField`、`WindowField`，在 FineUICore 和 FineUIJava 中没有对应列，不再推荐用于新代码。通用技能应使用三栈共有的 `RenderField`、`RenderCheckField`、`RowNumberField`、`GroupField` 及 `RenderField.Commands`。旧列名只能在升级或兼容说明中出现，并明确标注“不推荐用于新代码”。
- 不以“控件数量齐全”为目标机械添加内容。先判断它是否是公开、可直接使用、仍推荐的控件，再根据真实任务边界决定并入现有技能还是建立新技能。

## 技能组织

- 一个技能对应一类清晰的用户任务，不按源码继承层次拆分，也不为了减少目录数把无关控件拼在一起。
- `SKILL.md` 保留触发条件、必要判断和关键约束；详细示例放入 `references/`，并从 `SKILL.md` 明确链接。
- 技能名称和 `description` 必须准确反映实际内容。重命名技能时同步检查 README、相关技能引用、目录名和 frontmatter 的 `name`。
- 修改完成后，对每个受影响技能运行结构校验，并检查仓库中是否残留旧技能名或不推荐 API。

## Git

- 提交消息使用中文，不添加 `Co-Authored-By` 尾行。
- 不主动提交；只有用户明确要求提交时才提交。
- 仓库可能存在其他会话的未提交改动。暂存时逐个点名文件，禁止使用 `git add .` 或 `git add -A`。
