# 自定义主题（Custom Theme）

## 主题的三个文件

每个主题一个目录（`F/themes/{名}/`，自定义主题放 `res/themes/{名}/`）：

| 文件 | 作用 |
|------|------|
| `theme.config` | **颜色配置（必须）** —— 手写改这个 |
| `theme-extra.css` | 主题特有样式（可选），如背景图 |
| `theme.css` | **自动生成、勿手改** —— 由 `theme.config` 经 generate-theme 生成，输出 `:root { --f-*: ... }` CSS 变量 |

## 创建自定义主题（步骤）

1. 在 `F/themes/`（或示例项目 `res/themes/`）下新建目录，如 `my_theme/`。
2. 复制一个现成 `theme.config`（如 `pure_black/theme.config` 或 `custom_default/theme.config`），改颜色值。
3. （可选）建 `theme-extra.css` 加背景图等特有样式。
4. 生成：`node generate-theme.mjs my_theme`，或根目录 `npm run theme-gen -- my_theme` / `F\generate-theme.bat my_theme`（不带参 = 生成全部主题）。
5. 完成——用主题名 `my_theme`（走 `CustomTheme`，见 [set-theme.md](set-theme.md)）。

## theme.config 格式

`key = value` 每行一项，`#` 为注释。核心分组：

```ini
# 核心颜色（6 组状态：content / header / default / hover / active / error，各 border/background/text）
content-border-color = #e6e6e6
content-background-color = #ffffff
content-text-color = #444444
default-border-color = #e6e6e6
default-background-color = #ffffff
default-text-color = #444444
hover-border-color = #76b4ac
hover-background-color = #edf9f7
hover-text-color = #007465
active-border-color = #76b4ac
active-background-color = #e5f1ef
active-text-color = #007465
error-border-color = #ffa8a8
error-background-color = #fff8f8
error-text-color = #ff6c6c
border-radius = 6px

# 派生变量
primary-background-color = #007465
primary-text-color = #fff
tabstrip-inkbar-color = #007465

# 标志位
is-dark-background = false      # 深色主题设 true（body 会加 f-theme-darkbg）
is-dark-active-color = false    # 选中行底色深/淡
```

> 深色主题：`is-dark-background = true`，并把 content/header/default 的 background 改深色、text 改浅色；`focus-shadow-alpha` 建议 `.4`。

## 生成脚本原理（了解即可）

- `generate-theme.mjs` 读 `theme.config` → 用类 Sass 的 `color.scale` 预计算派生色 → 输出 `theme.css`（CSS 变量 `:root { --f-*: ... }`）。
- 打包时会把客户端版脚本同步分发到各示例项目的 `res/themes`（`F/examples` + 3 套 Core `wwwroot/res/themes` + Pro `res/themes`）。

## See also

- [set-theme.md](set-theme.md)：用自定义主题名切换（`CustomTheme`）
