

# 系统 Debloat 分析

基于你的需求：保留 SwayFX + i3-wm 双 WM 环境，保留 pywal 壁纸脚本生态（图片/视频壁纸、终端/GTK3 配色、Wayland + X11 双支持）。

---

## 一、功能重合 → 建议移除

### ✂️ 可以直接移除的

| 移除 | 理由 |
|------|------|
| `nano`、`vim` | neovim 完全覆盖 |
| `vinagre` | 已被上游弃用，remmina 功能完全覆盖 |
| `rdesktop` | 同上，remmina 支持 RDP |
| `go-musicfox-git-debug` | debug 包，没有日常用途 |
| `clash` + `clashtui` | mihomo（Clash.Meta 继任）完全覆盖 |
| `river-classic`、`weston`、`cage` | 你已有 SwayFX + i3 两套，这三个是额外的 WM/合成器 |
| `gtk3-demos`、`gtk4-demos` | GTK 示例程序，开发参考看完就可以删 |

### ✂️ 二选一类

**终端模拟器** — `alacritty` / `foot` / `kmscon`
- alacritty 和 foot 都同时支持 Wayland + X11（alacritty 原生双端，foot 是 Wayland 原生但 Xwayland 下也能跑）
- kmscon 是 TTY 级替代品，用途不同，除非你需要 TTY 下显示 CJK 否则可清
- 建议：留你常用的那个，另一个可清

**应用启动器** — `rofi` / `wofi`
- rofi 功能更强且同时支持 X11 + Wayland（rofi-wayland fork），双 WM 都能用
- 建议：**留 rofi，移除 wofi**（除非你 SwayFX 下专门配了 wofi）

**通知** — `dunst` / `swaync`
- 双 WM 场景下两个都保留其实有道理（i3 用 dunst，SwayFX 用 swaync）
- 但 dunst 也能在 Wayland 下工作，如果你愿意统一：**留 dunst 或 swaync 其一**

**亮度控制** — `brightnessctl` / `brillo`
- 功能完全一样，**留一个**

**音频控制** — `pavucontrol-gtk3` / `pwvucontrol`
- 你用 PipeWire，pwvucontrol 是原生方案
- 建议：**留 pwvucontrol**

**浏览器** — `firefox` / `google-chrome` / `ungoogled-chromium`
- chrome 和 ungoogled-chromium 都是 Chromium 内核
- 建议：**留 firefox + ungoogled-chromium，移除 google-chrome**（除非需要 Google 同步）

**PDF 阅读** — `sioyek` / `mupdf`
- sioyek 功能更强（书签、反向搜索、学术优化）
- 建议：**留 sioyek**

**办公** — `onlyoffice-bin` / `wps-office-cn`
- 建议：**留一个**

**登录管理器** — `sddm`（+ s6 + sugar 主题）/ `greetd`
- 两套，**留一套**。greetd 更轻量且 Wayland 友好

**系统监视器** — `htop` / `mate-system-monitor` / `xfce4-taskmanager`
- 后两个分别拉 MATE / XFCE 依赖
- 建议：**留 htop**，图形监视器如果不常用都可清

**Minecraft 启动器** — `minecraft-launcher` / `hmcl-bin` / `prismlauncher`
- 建议：**三选一**。prismlauncher 社区最活跃，hmcl 对国内生态支持好

---

## 二、按用途分类

### 🎮 游戏

**独立游戏：**
`0ad` `0ad-data` `0ad-zh-lang`、`supertuxkart`、`ioq3-bin`

**Steam / Wine 生态：**
`steam` `gamescope` `gamemode` `gamemodeswitch` `lib32-gamemode` `mangohud` `millennium`
`proton-ge-custom-bin` `wine-staging` `wine-gecko` `wine-mono` `dxvk-bin` `lib32-libdxvk` `vkd3d` `lib32-vkd3d` `spritz-wine-bin` `lutris`

**Minecraft：**
`minecraft-launcher` / `hmcl-bin` / `prismlauncher`

**其他：**
`the-honkers-railway-launcher-bin`（崩铁）、`kazumi`（追番）、`piliplus`（B站）

### 🎵 音乐制作 / DAW

**DAW / 合成器（6 个，高度重叠）：**
`lmms-qt6-git`、`rosegarden`、`zrythm`、`musescore`、`yoshimi`、`zynaddsubfx`、`minuet`

**音频插件：**
`lsp-plugins`、`mda.lv2`、`rubberband-ladspa`、`easyeffects`

**SoundFont（5 个）：**
`freepats-salamander-sf2`、`freepats-ydp-grand-piano`、`soundfont-fluid`、`soundfont-sgm`、`soundfont-unison`

> ⚠️ 这是最大的精简空间之一。如果你不再做音乐制作，**整个类别可以清掉**，省下大量空间。即使保留，DAW 留一两个 + SoundFont 留一两个就够。

### 🎵 音乐播放
`mpd` `ncmpcpp` `mpc` `mpd-mpris`（MPD 生态）
`go-musicfox-git`（网易云终端客户端）
`lollypop`（GTK 播放器）
`cava`（可视化）、`zensical`

### 📱 安卓 / 手机 / 刷机
`android-platform-35` `android-tools`（ADB/fastboot）
`scrcpy` `scrcpy-mask-bin`（投屏）
`pmbootstrap`（postmarketOS）
`checkra1n-cli`（iOS 越狱）
`edl-ng` `qdl`（高通 EDL 刷机）
`python-pyusb`

> 如果不再折腾刷机，`checkra1n-cli`、`edl-ng`、`qdl`、`pmbootstrap` 可以清。

### 🖌️ 创作 / 设计
`gimp`、`inkscape`、`krita` — 图像编辑（三个定位不同，gimp 位图/inkscape 矢量/krita 绘画）
`obs-studio`、`wf-recorder` — 录屏（OBS 功能更全，wf-recorder 更轻量脚本化）
`imagemagick` — 命令行图像处理（pywal 可能依赖，**保留**）
`tesseract-data-chi_sim` — OCR

### 💻 开发

**IDE：**
`code`、`pycharm-community-edition`、`qtcreator` `qtcreator-devel`

**JDK（4 个版本）：**
`jdk-openjdk`（最新）、`jdk21-openjdk`、`jdk17-openjdk`、`jdk8-openjdk`
> 只留你项目实际需要的版本。

**静态站点生成：**
`hugo` / `mkdocs-material` + `mkdocs-material-extensions`
> **两套**，留你博客用的那个。

**科学计算：**
`opencv`、`hdf5`、`vtk`、`jupyter-notebook`、`pyside6`
> 如果不做 CV / 科学可视化项目了，可以清。

**其他运行时/工具：**
`dotnet-runtime`、`npm` `yarn`、`python-pip`、`micromamba-bin`、`github-cli`、`jq`

### 🔒 安全 / 逆向 / 网络
`wireshark-qt`、`rkhunter`
`python-capstone`、`python-keystone`、`python-pycryptodome`

### 🔧 虚拟化 / 容器
`virtualbox` `virtualbox-guest-iso` / `qemu-desktop` / `libvirt` / `lxc` `podman`
> 如果只用一种方案，另外的可以清。

### 🌐 通信
`linuxqq`、`fluffychat`（Matrix）、`neomutt`（邮件）、`localsend`

### 📄 办公 / 文档
`onlyoffice-bin` / `wps-office-cn`
`cajviewer`（知网）、`sioyek` / `mupdf`、`vosviewer`（文献可视化）

### 🖥️ 远程桌面
`remmina`、`tigervnc`、`wayvnc`、`sunshine`
> remmina 是客户端（连别人），tigervnc/wayvnc 是服务端（别人连你），sunshine 是游戏串流。用途不完全相同，按需保留。

### 🔤 字体（大量）
`adobe-source-code-pro-fonts`、`adobe-source-han-sans-cn-fonts`、`adobe-source-han-serif-cn-fonts`
`noto-fonts-cjk`、`noto-fonts-emoji`
`ttf-arphic-ukai`、`ttf-arphic-uming`、`ttf-dejavu`、`ttf-maplemononl`
`wqy-bitmapfont`、`wqy-microhei`、`wqy-microhei-lite`、`wqy-zenhei`
`otf-font-awesome`、`woff2-font-awesome`

> CJK 字体有明显重叠：Source Han + Noto CJK 本质同源（Google 和 Adobe 联合开发），文泉驿系列是更早的方案。建议：**留 Noto CJK（覆盖最广）+ Source Code Pro（等宽）+ 你终端用的 Maple Mono + Font Awesome，文泉驿系列和 ttf-arphic 系列可以清**，除非有特定应用硬依赖。

### 🎨 GTK / 光标 / 图标主题（大量）

**GTK 主题：**
`adapta-gtk-theme`、`adw-gtk-theme`、`colloid-gtk-theme-git`、`materia-gtk-theme`、`numix-gtk-theme`、`materia-kde`、`kvantum-theme-materia`、`simplewaita-git`

**光标：**
`colloid-cursors-git`、`graphite-cursor-theme-git`、`lingmo-cursor-themes`、`ani2xcursor-bin`

**图标：**
`colloid-icon-theme-git`、`cosmic-icon-theme`、`tela-circle-icon-theme-all`

> pywal 会动态生成配色，但 GTK 主题本身还是需要一个底。建议：**只留你实际选用的一套 GTK 主题 + 一个光标 + 一个图标主题**，其余全清。

### 🏷️ 大概率可以清的杂项

| 包 | 说明 |
|---|------|
| `activate-linux` | 屏幕水印恶搞 |
| `lolcat` | 彩虹 cat |
| `fuck-u-code-git` | 玩具 |
| `ppet3-bin` | 桌面宠物 |
| `plymouth` | 启动动画，极简党不需要 |
| `gpm-s6` | TTY 鼠标支持 |
| `flite` | TTS 语音合成，确认是否在用 |
| `kali-themes` | Kali 主题，你不跑 Kali |
| `artix-wallpapers`、`artix-dark-theme`、`artix-branding-base` | 发行版默认主题，用自己的了 |
| `arch-wiki-docs`、`arch-wiki-docs-zh-cn` | 离线 ArchWiki，有网就不需要 |
| `archinstall` | 安装器，装完系统后无用 |
| `udisks2-docs` | 文档包 |
| `electron16-bin` | 极老的 Electron 版本，确认哪个应用依赖它 |
| `displaycal` | 显示器校色，确认是否在用 |
| `glmark2` | GPU 基准测试，跑完就可以删 |
| `wev`、`xorg-xev` | 输入事件调试，日常不需要 |
| `os-prober` | 多系统引导检测，单系统不需要 |
| `ventoy-bin` | U 盘启动工具，用完可以删 |
| `llmfit-bin`、`ww-manager`、`kd-bin` | 确认是否还在用 |

---

## 三、快速行动清单

**直接移除（无争议）：**
```bash
sudo pacman -Rns nano vim vinagre rdesktop go-musicfox-git-debug \
  clash clashtui river-classic weston cage gtk3-demos gtk4-demos \
  activate-linux lolcat plymouth gpm-s6 kali-themes archinstall \
  udisks2-docs os-prober artix-wallpapers artix-dark-theme \
  artix-branding-base glmark2
```

**确认后移除（二选一 / 确认不用）：**

- 终端、启动器、通知、亮度、音频控制、浏览器、PDF、办公、登录管理器、Minecraft 启动器 — 各选一个保留

- 字体精简 — 移除文泉驿 + ttf-arphic 系列

- 主题精简 — 只留在用的一套

**按整个类别审视：**

- 音乐制作（最大精简空间）

- 多 JDK 版本

- 刷机工具链

- 科学计算库

- 虚拟化方案

有哪个类别想深入讨论的随时说 🐧


