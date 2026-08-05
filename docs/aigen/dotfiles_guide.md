# Yulliil's Artix Linux 桌面配置食用指南

> **系统环境：** Artix Linux (s6 init) + SwayFX (Wayland)
>
> **仓库地址：**
> - Dotfiles：[Yulliil/My_Artix_Configuration](https://codeberg.org/Yulliil/My_Artix_Configuration)
> - 壁纸脚本：[Yulliil/wallpaper.sh](https://codeberg.org/Yulliil/wallpaper.sh)
> - GTK 主题：[Yulliil/Materia-dark-trans-compact-wal](https://codeberg.org/Yulliil/Materia-dark-trans-compact-wal)

---

## 目录

1. [整体架构概览](#1-整体架构概览)
2. [依赖清单](#2-依赖清单)
3. [安装与部署](#3-安装与部署)
4. [wallpaper.sh 壁纸与配色引擎](#4-wallpapersh-壁纸与配色引擎)
5. [GTK 主题：Materia-dark-trans-compact-wal](#5-gtk-主题materia-dark-trans-compact-wal)
6. [SwayFX 窗口管理器配置](#6-swayfx-窗口管理器配置)
7. [终端：Alacritty（默认）& foot（备用）](#7-终端alacritty默认--foot备用)
8. [Waybar 状态栏](#8-waybar-状态栏)
9. [Rofi 应用启动器](#9-rofi-应用启动器)
10. [Dunst 通知守护进程](#10-dunst-通知守护进程)
11. [电池监控与自动刷新率切换](#11-电池监控与自动刷新率切换)
12. [MPD 音乐子系统](#12-mpd-音乐子系统)
13. [其他组件](#13-其他组件)
14. [快捷键速查表](#14-快捷键速查表)
15. [已弃用组件说明](#15-已弃用组件说明)
16. [常见问题 / 需要手动修改的地方](#16-常见问题--需要手动修改的地方)

---

## 1. 整体架构概览

整套配置的核心思路是 **壁纸驱动的全局动态配色**——换一张壁纸，从窗口管理器边框、GTK 应用背景、终端配色、启动器主题到通知弹窗，全部自动跟着壁纸色调变化。

### 配色管线流程

```
wallpaper.sh <壁纸图片/视频/目录>
    │
    ├─ 1. 检测后端 (Wayland / X11)
    ├─ 2. 清理旧壁纸进程 (swaybg / mpvpaper)
    ├─ 3. 视频？→ ffmpeg 提取缩略帧
    ├─ 4. matugen 从图片提取 Material Design 3 配色
    │       ├─ 生成 colors-sway          → ~/.cache/wallpaper.sh.d/
    │       ├─ 生成 colors-alacritty.toml → ~/.cache/wallpaper.sh.d/
    │       ├─ 生成 colors-foot.ini      → ~/.cache/wallpaper.sh.d/
    │       ├─ 生成 rofi_config.rasi     → ~/.cache/wallpaper.sh.d/
    │       └─ 生成 GTK CSS 变量文件      → /tmp/Materia-dark-trans-compact-wal-color.css
    │
    ├─ 5. matugen post_hook 触发
    │       ├─ pkill -USR1 foot   (foot 终端热重载配色)
    │       └─ swaymsg reload     (SwayFX 重载，读取新的 colors-sway)
    │
    ├─ 6. 设置壁纸 (swaybg / mpvpaper)
    ├─ 7. gsettings 设置深浅色模式 + 图标主题
    └─ 8. wallpaper-reload.sh
            └─ 重启 waybar（加载新配色的 CSS）
```

**关键设计决策：**
- 使用 **matugen**（非 pywal）作为配色引擎，基于 Material Design 3 色彩科学提取壁纸配色
- 配色文件统一缓存在 `~/.cache/wallpaper.sh.d/`，GTK 色值文件放 `/tmp/`
- 各应用通过 `@import` / `import` / `include` 机制引用缓存中的配色文件，实现运行时动态换色
- 支持深色模式（默认）和浅色模式（`-l` 参数）

---

## 2. 依赖清单

### 核心依赖

| 组件 | 用途 | 安装 |
|------|------|------|
| **SwayFX** | 窗口管理器（Sway 的 fork，支持模糊/动画） | `swayfx` |
| **matugen** | 从壁纸提取 Material Design 3 配色 | AUR: `matugen-bin` |
| **swaybg** | 静态壁纸 | `swaybg` |
| **mpvpaper** | 视频壁纸 | `mpvpaper` |
| **ffmpeg** | 视频缩略帧提取 | `ffmpeg` |
| **waybar** | 状态栏 | `waybar` |
| **alacritty** | 默认终端 | `alacritty` |
| **fish** | Shell | `fish` |
| **rofi** | 应用启动器（Wayland 版用 `rofi-wayland`） | `rofi-wayland` |
| **dunst** | 通知守护进程 | `dunst` |
| **grim** | 截图 | `grim` |
| **slurp** | 区域选择（配合截图） | `slurp` |
| **jq** | JSON 处理（截图脚本用） | `jq` |
| **playerctl** | 媒体控制 | `playerctl` |
| **brightnessctl** | 亮度控制 | `brightnessctl` |
| **wob** | 音量/亮度浮层条 | `wob` |
| **fcitx5** | 输入法 | `fcitx5` |

### 音频栈

| 组件 | 用途 |
|------|------|
| **PipeWire** | 音频服务 |
| **pipewire-pulse** | PulseAudio 兼容层 |
| **WirePlumber** | 会话管理 |

### 外观依赖

| 组件 | 用途 |
|------|------|
| **Materia-dark-trans-compact-wal** | GTK 主题（本仓库） |
| **Colloid-Pink-Dark / Colloid-Pink** | 图标主题（深/浅色各一套） |
| **lingmo-dark** | 鼠标光标主题 |
| **Source Code Pro** | 终端/编辑器字体 |
| **FontAwesome** | Waybar 图标字体 |
| **Noto Sans / Noto Sans CJK SC** | UI 字体 |

### 可选依赖

| 组件 | 用途 |
|------|------|
| **foot** | 备用终端（支持 SIGUSR1 热重载配色） |
| **pcmanfm** | 文件管理器 |
| **pavucontrol** | 音量控制面板 |
| **nwg-displays** | 显示器配置工具 |
| **swayimg** | 图片查看（evernight GIF 特效用） |
| **mpd + mpd-mpris** | 音乐播放 |
| **thefuck** | 终端命令纠错 |
| **fastfetch** | 系统信息展示 |

---

## 3. 安装与部署

### 3.1 克隆仓库

```bash
# 1. Dotfiles
git clone https://codeberg.org/Yulliil/My_Artix_Configuration.git

# 2. 壁纸脚本（克隆到你喜欢的位置，脚本会自动检测自身路径）
git clone https://codeberg.org/Yulliil/wallpaper.sh.git ~/Projects/wallpaper

# 3. GTK 主题
git clone https://codeberg.org/Yulliil/Materia-dark-trans-compact-wal.git
```

### 3.2 部署 Dotfiles

将 `My_Artix_Configuration` 中的各目录复制到 `~/.config/`：

```bash
cd My_Artix_Configuration

# 核心配置
cp -r sway/ ~/.config/sway/
cp -r alacritty/ ~/.config/alacritty/
cp -r waybar/ ~/.config/waybar/           # 备用的 Hyprland/独立 waybar 配置
cp -r dunst/ ~/.config/dunst/
cp -r rofi/ ~/.config/rofi/               # 静态回退 rofi 配置
cp -r fish/ ~/.config/fish/
cp -r wob/ ~/.config/wob/

# 可选
cp -r foot/ ~/.config/foot/               # 备用终端
cp -r aria2/ ~/.config/aria2/             # 下载管理器
cp -r swayimg/ ~/.config/swayimg/         # GIF 特效
```

### 3.3 安装 GTK 主题

```bash
cd Materia-dark-trans-compact-wal

# 复制主题文件夹到系统主题目录（需要 root 权限）
sudo cp -r Materia-dark-trans-compact-wal/ /usr/share/themes/
```

> **注意：** 主题放在 `/usr/share/themes/` 下，修改 CSS 需要 `sudo`。如果你想无 root 编辑，可以放到 `~/.local/share/themes/` 下，但需要确认你的 GTK 设置能从该路径加载主题。

### 3.4 初始化配色缓存

首次使用前，需要运行一次 wallpaper.sh 来生成配色文件：

```bash
# 创建缓存目录
mkdir -p ~/.cache/wallpaper.sh.d

# 运行壁纸脚本（会自动生成所有配色文件）
~/Projects/wallpaper/wallpaper.sh ~/Pictures/你的壁纸.png
```

### 3.5 应用 GTK 主题

```bash
# 通过 gsettings 设置 GTK 主题
gsettings set org.gnome.desktop.interface gtk-theme "Materia-dark-trans-compact-wal"

# 或者使用 nwg-look 等图形工具来选择主题
```

---

## 4. wallpaper.sh 壁纸与配色引擎

**仓库：** [Yulliil/wallpaper.sh](https://codeberg.org/Yulliil/wallpaper.sh)

### 4.1 基本用法

```bash
# 设置单张图片壁纸（深色模式，默认）
wallpaper.sh ~/Pictures/wallpaper.png

# 从目录随机选择壁纸
wallpaper.sh ~/Pictures/wallpapers/

# 浅色模式
wallpaper.sh -l ~/Pictures/wallpaper.png

# 视频壁纸
wallpaper.sh ~/Videos/loop.mp4

# 帮助
wallpaper.sh -h
```

**支持的格式：** `png`、`jpg`、`jpeg`、`mp4`、`mkv`

### 4.2 深色 / 浅色模式

| 参数 | 背景透明度 | 图标主题 | GTK 配色方案 |
|------|-----------|---------|-------------|
| 默认（深色） | 0.7 | Colloid-Pink-Dark | `prefer-dark` |
| `-l`（浅色） | 0.5 | Colloid-Pink | `prefer-light` |

切换模式时，wallpaper.sh 会自动通过 `gsettings` 更新系统级的 `color-scheme` 和 `icon-theme`。

### 4.3 matugen 配色引擎

wallpaper.sh 使用 [matugen](https://github.com/InioX/matugen) 替代了之前的 pywal。matugen 基于 Material Design 3 的色彩算法从壁纸中提取配色方案。

**matugen.toml 模板配置：**

```toml
[templates.foot]
input_path = 'templates/colors-foot.ini'
output_path = '~/.cache/wallpaper.sh.d/colors-foot.ini'
post_hook = 'pkill -USR1 foot'         # foot 终端热重载

[templates.alacritty]
input_path = 'templates/colors-alacritty.toml'
output_path = '~/.cache/wallpaper.sh.d/colors-alacritty.toml'

[templates.gtk]
input_path = 'templates/Materia-dark-trans-compact-wal-color.css'
output_path = '/tmp/Materia-dark-trans-compact-wal-color.css'

[templates.sway]
input_path = 'templates/colors-sway'
output_path = '~/.cache/wallpaper.sh.d/colors-sway'
post_hook = "swaymsg reload"            # SwayFX 重载配置

[templates.rofi]
input_path = 'templates/rofi_config.rasi'
output_path = '~/.cache/wallpaper.sh.d/rofi_config.rasi'
```

**生成的配色文件与消费者对应关系：**

| 生成文件 | 路径 | 消费者 | 重载方式 |
|---------|------|--------|---------|
| `colors-sway` | `~/.cache/wallpaper.sh.d/` | SwayFX (窗口边框颜色) | `swaymsg reload` (post_hook) |
| `colors-alacritty.toml` | `~/.cache/wallpaper.sh.d/` | Alacritty 终端 | `live_config_reload = true` |
| `colors-foot.ini` | `~/.cache/wallpaper.sh.d/` | foot 终端 | `pkill -USR1 foot` (post_hook) |
| `rofi_config.rasi` | `~/.cache/wallpaper.sh.d/` | Rofi 启动器 | 下次启动时生效 |
| `Materia-...-color.css` | `/tmp/` | GTK 主题 | GTK 应用窗口重绘时生效 |

### 4.4 模板变量体系

matugen 模板使用两套变量命名空间：

**Material Design 3 语义色：**
- `{{colors.primary.default.hex}}` — 主色调
- `{{colors.surface_container.default.rgba}}` — 容器表面色（带 alpha，用于透明背景）
- `{{colors.tertiary.default.hex}}` — 第三色调
- `{{colors.on_surface.default.hex}}` — 表面上的文字色
- 等等……

**Base16 调色板：**
- `{{base16.base00.default.hex}}` 到 `{{base16.base0f.default.hex}}` — 16 色调色板

### 4.5 与 dotfiles 的集成

dotfiles 中的 Sway 主配置文件通过以下机制与 wallpaper.sh 联动：

```bash
# sway/config 中：
# 1. 引入 wallpaper.sh 生成的 SwayFX 配色
include ~/.cache/wallpaper.sh.d/colors-sway

# 2. 窗口边框颜色使用生成的变量
client.focused          $primary_color $background $foreground $primary_color $primary_color
client.unfocused        $background $background $foreground $background $background
# ...

# 3. 启动时运行 wallpaper.sh
exec ~/Projects/wallpaper/wallpaper.sh ~/Pictures/wallpaper/cycle/
```

---

## 5. GTK 主题：Materia-dark-trans-compact-wal

**仓库：** [Yulliil/Materia-dark-trans-compact-wal](https://codeberg.org/Yulliil/Materia-dark-trans-compact-wal)

### 5.1 主题特性

这是一个基于 [Materia](https://github.com/nana-4/materia-theme) (Materia-dark-compact 变体) 深度定制的 GTK 主题，主要特性：

- **半透明背景**：窗口背景使用 RGBA 颜色，壁纸可透过窗口显示（"trans" 的含义）
- **紧凑布局**：按钮 24px、输入框 32px、标题栏 40px（"compact" 的含义）
- **动态配色**：通过 `@import` 引用 wallpaper.sh 生成的 CSS 变量文件，配色跟随壁纸变化（"wal" 的含义）
- **Backdrop 状态响应**：失焦窗口背景自动变暗，带 0.25s 过渡动画
- **Material Design 动效**：点击涟漪、阴影层级、标准贝塞尔曲线

### 5.2 配色变量

主题 CSS 通过 `@import url("/tmp/Materia-dark-trans-compact-wal-color.css")` 引入以下变量：

```css
/* 背景色 */
@define-color background <surface_container RGBA>;   /* 带 alpha 透明 */
@define-color background_no_alpha <surface_container RGB>;  /* 不带 alpha */

/* Base16 调色板 */
@define-color color0  到  @define-color color15;

/* Material Design 语义色 */
@define-color primary ...;
@define-color primary_container ...;
@define-color tertiary ...;
@define-color tertiary_container ...;
@define-color surface ...;
```

**GTK 3.0 vs GTK 4.0 的差异：**

| 特性 | GTK 3.0 | GTK 4.0 |
|------|---------|---------|
| 动态变量使用 | 丰富（@color7, @primary, @background 等） | 有限（仅 @color4） |
| Backdrop 响应 | 完整实现，背景动态变化 | 无 `.background:backdrop` 规则 |
| 其他颜色 | 大量使用动态变量 | 多数为硬编码值 |

因此，**GTK 3.0 应用**（如 Thunar、pcmanfm）的动态配色效果远好于 GTK 4.0 应用。

### 5.3 Backdrop 状态（失焦窗口响应）

这是最新加入的特性。当窗口失去焦点时：

```css
.background {
  background-color: @background;             /* 正常状态：matugen 生成的 RGBA 背景 */
  color: @color7;
  transition: background-color 0.25s;        /* 0.25 秒过渡动画 */
}

.background:backdrop {
  background-color: alpha(@background_no_alpha, 0.8);  /* 失焦：80% 不透明度 */
}
```

**效果：**
- 焦点窗口：半透明背景（透明度由 matugen 的 `--opacity` 控制，默认 0.7）
- 失焦窗口：背景变为 80% 不透明度，视觉上变暗
- 切换时有 0.25 秒的平滑过渡
- 文字颜色始终不变，保持可读性

> **注意：** 这不是 bug，而是 feature。如果你在使用中看到后面的窗口比当前窗口暗，那就是 backdrop 状态在正常工作。

### 5.4 GTK Inspector 调试方法

修改主题时使用 GTK Inspector：

```bash
# 启动应用并打开 Inspector
GTK_THEME=Materia-dark-trans-compact-wal GTK_DEBUG=interactive <应用名>

# 推荐组合：Inspector + 重绘区域高亮
GTK_THEME=Materia-dark-trans-compact-wal GTK_DEBUG=interactive:updates <应用名>
```

在 Inspector 左上角点击「选择元素」按钮 → 在应用窗口点击目标 widget → 切到 CSS Nodes 选项卡查看该元素的 CSS node 路径和当前匹配的规则 → 编辑 `gtk.css` → 重启应用查看效果。

---

## 6. SwayFX 窗口管理器配置

### 6.1 基本设定

| 项目 | 值 |
|------|-----|
| Mod 键 | `Super`（Mod4） |
| 方向键映射 | **WASD**（非 HJKL） |
| 默认终端 | `alacritty -e fish` |
| 文件管理器 | `pcmanfm` |
| 启动器 | `rofi`（使用 wallpaper.sh 生成的动态主题） |
| Xwayland | 启用 |
| 窗口边框 | 1px pixel（无标题栏） |
| 间距 | 内间距 5px，顶部 0 |
| 光标主题 | `lingmo-dark`，大小 20 |

### 6.2 SwayFX 视觉效果

```
blur: 3 passes, radius 10
├─ noise: 0.05
├─ brightness: 1
├─ contrast: 1.2
├─ saturation: 1.1
└─ xray: 启用

动画时长: 250ms
圆角: 0（无圆角）
```

**图层效果（layer_effects）：**

| 图层 | 效果 |
|------|------|
| waybar | `blur_ignore_transparent`（透明区域不模糊） |
| wob | `blur` |
| notifications | `blur` + `blur_ignore_transparent` |
| rofi | `blur` + `xray` + `shadows` |
| gtk-layer-shell | `blur` + `blur_ignore_transparent` |
| fcitx5 | `blur` |

### 6.3 自启动服务

Sway 配置中 `exec` 启动的后台服务：

```
dbus-update-activation-environment    (D-Bus 环境变量)
pipewire → pipewire-pulse → wireplumber  (音频栈)
fcitx5                                  (输入法)
dunst                                   (通知)
wob (通过 FIFO ~/.config/wob/wobpipe)   (音量/亮度浮层)
BatteryMonitorC                         (电池监控 + 自动刷新率)
mate-polkit                             (权限提升代理)
gnome-keyring-daemon                    (密钥管理)
wallpaper.sh ~/Pictures/wallpaper/cycle/ (壁纸 + 配色初始化)
```

### 6.4 窗口规则（部分）

SwayFX 配置中定义了大量 per-app 窗口规则：

| 应用 | 规则 |
|------|------|
| pavucontrol | 浮动，固定 663×405 |
| Thunar 对话框 | 浮动（重命名/进度/替换） |
| QQ 各子窗口 | 浮动（图片查看器/文件管理/设置等） |
| galculator | 浮动 |
| `.exe` 应用 | 浮动（Wine/Proton 程序） |
| 游戏/启动器 | 关闭模糊（性能优化） |
| PPet3, bongo-cat | 固定尺寸、粘滞、关闭模糊（桌面宠物） |

---

## 7. 终端：Alacritty（默认）& foot（备用）

### 7.1 Alacritty 配置

**配置文件：** `~/.config/alacritty/alacritty.toml`

```toml
[general]
import = ["~/.cache/wallpaper.sh.d/colors-alacritty.toml"]  # 动态配色
live_config_reload = true  # 配色文件变化时自动重载

[font]
normal = { family = "SourceCodePro", style = "LightIt" }
bold   = { family = "SourceCodePro", style = "BoldIt" }

[scrolling]
multiplier = 3
```

**配色重载机制：** Alacritty 内建 `live_config_reload`，当 `~/.cache/wallpaper.sh.d/colors-alacritty.toml` 被 matugen 重写时，终端配色自动更新，无需重启。

### 7.2 foot 配置（备用）

**配置文件：** `~/.config/foot/foot.ini`

- Shell：fish
- 字体：Source Code Pro 12，Noto Sans CJK SC 回退
- 初始窗口：900×600
- 配色：从 `~/.cache/wallpaper.sh.d/colors-foot.ini` 导入
- 快捷键：`Ctrl+P/N` 半页滚动，`Ctrl+Shift+O` 打开 URL

**配色重载机制：** matugen 的 post_hook 发送 `SIGUSR1` 信号，foot 收到后立即热重载配色，无需重启终端。

---

## 8. Waybar 状态栏

### 8.1 配置路径

SwayFX 专用的 Waybar 配置不在标准的 `~/.config/waybar/` 下，而是放在：

- 配置：`~/.config/sway/waybar_config.jsonc`
- 样式：`~/.config/sway/waybar_style.css`

启动命令：
```bash
waybar -c ~/.config/sway/waybar_config.jsonc -s ~/.config/sway/waybar_style.css -l off
```

### 8.2 模块布局

```
┌─────────────────────────────────────────────────────────────────────┐
│ [工作区] [模式] [暂存区] [窗口标题]     音量|网络|CPU|内存|温度|电源|亮度|电池|时钟|托盘 │
└─────────────────────────────────────────────────────────────────────┘
```

- 高度 25px，`reload_style_on_change` 启用
- 模块间使用 `|` 分隔符
- 时钟：`%H:%M`，点击切换为 `%Y-%m-%d`
- 音量点击打开 `pavucontrol`（自动应用 GTK 主题）

### 8.3 样式特性

```css
@import url("/tmp/Materia-dark-trans-compact-wal-color.css");
```

- 字体：FontAwesome + Noto Sans，16px
- 左右模块容器：pill 形状（border-radius 20px），matugen 背景色
- 工作区：透明背景 + 微弱的 `@color4` 底色，焦点工作区使用 `@primary` 色 20% 透明度
- 所有模块背景透明（颜色由 pill 容器提供）
- 电池 critical 状态：红色闪烁动画

---

## 9. Rofi 应用启动器

### 9.1 启动方式

Sway 配置中的启动命令：
```bash
rofi -show drun -theme ~/.cache/wallpaper.sh.d/rofi_config.rasi
```
快捷键：`Super+R`

### 9.2 动态主题

Rofi 主题由 wallpaper.sh 通过 matugen 自动生成到 `~/.cache/wallpaper.sh.d/rofi_config.rasi`。

- 窗口背景：壁纸色带 alpha 透明
- 文字颜色：base16 base07
- 选中项：`@primary` 色 + 30% 透明度（`{{colors.primary.default.hex}}30`）
- 字体：Source Code Pro 12
- 12 行列表 + 滚动条 + 2em 图标

### 9.3 静态回退

如果动态主题文件不存在（比如首次使用前），`~/.config/rofi/config.rasi` 提供一个静态的深色透明主题作为回退。

---

## 10. Dunst 通知守护进程

### 10.1 基本配置

**配置文件：** `~/.config/dunst/dunstrc`

- 宽度：0-400px，高度 0-100px
- 位置：右上角，偏移 10,10
- 透明度：80%
- 边框：1px
- 字体：NotoSans 8
- 图标主题：Colloid-Pink-Dark
- 间距：10px

### 10.2 动态配色

`~/.config/dunst/dunstrc.d/00-look.conf` 存放 matugen 生成的配色覆盖。

> **注意：** 当前 `wallpaper-reload.sh` 中 dunst 的重启已被注释掉。dunst 的配色更新需要手动重启 dunst，或者取消 `wallpaper-reload.sh` 中相关行的注释。

---

## 11. 电池监控与自动刷新率切换

### 11.1 BatteryMonitorC

一个用 C 编写的轻量级守护进程，在 SwayFX 启动时自动运行。

**功能：**
- 每秒轮询 `/sys/class/power_supply/BAT0/` 和 `/sys/class/power_supply/ADP1/online`
- **插入 AC 电源** → 自动切换到 144Hz 刷新率 + 发送通知
- **拔出 AC 电源** → 自动切换到 60Hz 刷新率 + 发送通知
- **电量低于 20%** → 发送 critical 通知（去重，恢复后重置）

**源码位置：** `~/.config/sway/scripts/BattertyMonitor/main.c`

> 如果你对预编译二进制不放心，可以自行编译：
> ```bash
> cd ~/.config/sway/scripts/BattertyMonitor/
> gcc main.c -o BatteryMonitorC
> ```

---

## 12. MPD 音乐子系统

### 12.1 架构

位于 `~/.config/sway/scripts/music/` 的独立音乐播放子系统：

```
music-init.sh
  ├─ 生成 MPD 临时配置（随机端口）
  ├─ 启动 MPD 实例
  ├─ 启动 mpd-mpris（MPRIS 协议桥接）
  └─ 构建全曲目播放列表

music-waybar.sh
  ├─ 生成底部 Waybar 配置
  └─ 启动带音乐控件的 Waybar（上一首/暂停/下一首/当前曲目）
```

### 12.2 使用

```bash
# 初始化并启动 MPD
~/.config/sway/scripts/music/music-init.sh

# 启动音乐控制条（底部 Waybar）
~/.config/sway/scripts/music/music-waybar.sh
```

---

## 13. 其他组件

### 13.1 WOB（Wayland Overlay Bar）

**配置：** `~/.config/wob/wob.ini`

- 宽度 384px，高度 5px
- 顶部居中
- 半透明深色背景，白色进度条，红色溢出

通过 FIFO 管道 `~/.config/wob/wobpipe` 接收数值输入（音量/亮度快捷键写入）。

### 13.2 Evernight 桌面 GIF 特效

从 `~/Pictures/Evernight_Shake/` 随机选取一个 GIF，通过 swayimg 以透明浮层形式显示在桌面上。

```bash
~/.config/sway/scripts/evernight.sh
```

### 13.3 Fish Shell

- 启动时运行 `fastfetch`
- 集成 `thefuck` 命令纠错
- 自定义提示符：`user@host PWD [status] >`
- `Ctrl+Backspace` 删除前一个词

---

## 14. 快捷键速查表

### 窗口管理

| 快捷键 | 功能 |
|--------|------|
| `Super+Return` | 打开 Alacritty 终端 |
| `Super+Shift+Return` | 打开浮动终端 |
| `Super+E` | 打开文件管理器（pcmanfm） |
| `Super+R` | Rofi 应用启动器 |
| `Super+Q` | 关闭当前窗口 |
| `Super+F` | 全屏切换 |
| `Super+Space` | 浮动/平铺切换 |
| `Super+L` | 锁屏 |
| `Super+M` | 退出 SwayFX（需确认） |

### 焦点与移动（WASD）

| 快捷键 | 功能 |
|--------|------|
| `Super+W/A/S/D` | 切换焦点（上/左/下/右） |
| `Super+Shift+W/A/S/D` | 移动窗口 |
| `Super+Z` → `W/A/S/D` | 进入 resize 模式后调整大小 |

### 工作区

| 快捷键 | 功能 |
|--------|------|
| `Super+1~0` | 切换到工作区 1-10 |
| `Super+Shift+1~0` | 将窗口移到工作区 1-10 |
| `Super+Shift+/` | 移到暂存区 |
| `Super+/` | 显示暂存区 |

### 媒体与亮度

| 快捷键 | 功能 |
|--------|------|
| `XF86AudioPlay/Prev/Next` | playerctl 控制 |
| `XF86AudioRaiseVolume/Lower/Mute` | wpctl + wob 反馈 |
| `XF86MonBrightnessUp/Down` | brightnessctl + wob 反馈 |

### 截图

| 快捷键 | 功能 |
|--------|------|
| `Super+P` | 全屏截图 |
| `Super+Shift+P` | 活动窗口截图 |
| `Super+Ctrl+P` | 区域截图 |

### 配置编辑

| 快捷键 | 功能 |
|--------|------|
| `Super+B` | 隐藏/显示 Waybar |
| `Super+Shift+B` | 重载 Waybar |
| `Super+I` | 编辑 Sway 主配置 |
| `Super+Shift+I` | 编辑 SwayFX 配置 |

---

## 15. 已弃用组件说明

以下组件的配置文件仍保留在仓库中，但**已弃用，不再使用**：

### wofi

- **状态：** 已弃用
- **文件：** `wofi/config`、`wofi/style.css`
- **替代：** Rofi（`rofi-wayland`）
- **说明：** wofi 的配置和 Catppuccin Macchiato 主题仍在仓库中但不再维护。启动器已统一使用 Rofi + wallpaper.sh 动态配色。

### swaylock

- **状态：** 已弃用
- **文件：** `sway/swaylock`
- **说明：** swaylock 配置仍在仓库中但不再作为默认锁屏方案维护。配置指向固定壁纸 `StelleResized.png`，不与动态壁纸联动。

### foot（作为默认终端）

- **状态：** 降级为备用终端
- **文件：** `foot/foot.ini`
- **替代：** Alacritty
- **说明：** foot 的配置仍然完整可用，matugen 仍然会为 foot 生成配色文件并通过 `SIGUSR1` 热重载。如果你更喜欢 foot，只需修改 Sway 配置中的 `$term` 变量。

### Hyprland 配置

- **状态：** 不再维护
- **文件：** `hypr/hypr.conf`
- **说明：** 旧格式的 Hyprland 配置，使用 dmenu 和 kitty，不与当前配色系统联动。

---

## 16. 常见问题 / 需要手动修改的地方

### 16.1 视频壁纸显示器名称

wallpaper.sh 中 `mpvpaper` 的输出显示器硬编码为 `eDP-1`（笔记本内置屏幕）。如果你使用外接显示器，需要修改：

```bash
# wallpaper.sh 中找到这行
mpvpaper -o '--loop' eDP-1 "$wallpaper" -f
# 改为你的显示器名称，例如
mpvpaper -o '--loop' HDMI-A-1 "$wallpaper" -f
```

查看显示器名称：`swaymsg -t get_outputs | jq '.[].name'`

### 16.2 colors-sway 中的硬编码壁纸路径

`wallpaper.sh/templates/colors-sway` 模板中有一行硬编码的壁纸路径：

```
set $wallpaper /home/Yulliil/Pictures/wallpaper/cycle/1203650158_p0_resized.png
```

这是作者的个人路径，不会动态更新为你当前使用的壁纸。如果你的 Sway 配置依赖 `$wallpaper` 变量来设置壁纸背景，需要修改这行为你自己的路径，或者将其改为动态生成（目前壁纸设置是由 wallpaper.sh 直接调用 swaybg 完成的，不依赖这个变量）。

### 16.3 电池监控的硬件路径

BatteryMonitorC 读取的硬件路径：
- 电池：`/sys/class/power_supply/BAT0/`
- AC 适配器：`/sys/class/power_supply/ADP1/`

不同硬件上这些路径可能不同（如 `BAT1`、`AC0`）。请检查 `ls /sys/class/power_supply/` 并修改源码后重新编译。

### 16.4 刷新率切换

BatteryMonitorC 默认在 AC 电源插入时切换到 144Hz，拔出时切换到 60Hz。如果你的屏幕不支持 144Hz，需要修改源码中的刷新率值。

### 16.5 /tmp 下的 GTK 配色文件

GTK 配色文件生成到 `/tmp/`，该目录在重启后会被清空。因此每次开机后 wallpaper.sh 需要运行一次来重新生成配色文件。在 Sway 配置中已通过 `exec wallpaper.sh` 处理了这个问题。

### 16.6 dunst 配色不自动更新

当前 `wallpaper-reload.sh` 中 dunst 的重启代码被注释掉了。如果你想让 dunst 配色跟随壁纸变化，取消注释以下行：

```bash
# wallpaper-reload.sh 中取消注释：
pkill dunst
cp ~/.cache/wal/00-look.conf ~/.config/dunst/dunstrc.d/00-look.conf
dunst &
```

> **注意：** dunst 的模板（`templates/00-look.conf`）仍使用旧版 pywal 语法（`{background}`、`{color7}`），尚未迁移到 matugen 语法。如需启用 dunst 动态配色，需要将模板改为 matugen 的 `{{...}}` 语法并在 `matugen.toml` 中注册。

---

*最后更新：2026-08-05*
*基于 dotfiles commit 26+、wallpaper.sh commit 11、GTK 主题 commit 9*
