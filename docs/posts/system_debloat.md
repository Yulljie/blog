---
date:
  created: 2026-06-19
authors:
  - Yulliil
categories:
  - Linux 使用
draft: false
---

# Debloat 你的系统

!!! tip "本文无关 systemd，请放心食用。"

对于 Arch 这种命令式发行版，系统状态会随时间交织、积累。如果平时不加以维护，用的时间越长，通常会变得越笨重。篇幅有限，本文不会涵盖所有的 debloat 路径，仅从几个角度提供一些帮助。
<!-- more -->

## 移除不常用的软件包

有时兴起想体验个新玩具，然后体验完就忘记卸了；或者是对比几个同类工具，然后比完就忘记卸了……电脑上总会产生一些用不到的软件，如果时间比较久远的话可能自己都不知道哪个包是干什么的了。。。不过幸运的是，在 LLM 的帮助下，这个问题如今可以很容易地解决。

简单来讲就是把自己安装的软件包列表甩给 LLM，然后讲清楚需求，让 LLM 给卸载建议即可。

!!! warning "警告"

    **永远**不要让 LLM 直接操控你的系统！这种放权导致的惨剧已经屡见不鲜了<del>，你也不想你的系统被删完，对吧</del>。

单行列出所有显式安装的包（可以不用考虑作为依赖被安装的包，最后 `-Rns` 即可带着卸载依赖）：

```console
$ pacman -Q --explicit | sed 's/ .*//' | tr '\n' ' '
0ad 0ad-data 0ad-zh-lang 7zip activate-linux adapta-gtk-theme adobe-source-code-pro-fonts adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts adw-gtk-theme alacritty android-platform-35 android-tools ani2xcursor-bin arch-wiki-docs arch-wiki-docs-zh-cn archinstall archlinuxcn-keyring aria2 artix-archlinux-support artix-branding-base artix-dark-theme artix-keyring artix-wallpapers audit-s6 baidunetdisk-bin base base-devel basu bat bat-extras bc blueman bluez bluez-s6 bluez-utils brightnessctl brillo cage cajviewer cava checkra1n-cli clash clashtui cliphist clipit code colloid-cursors-git colloid-gtk-theme-git colloid-icon-theme-git cosmic-icon-theme cronie-s6 cryptsetup-s6 dash dbus-s6 device-mapper-s6 dhcpcd-s6 displaycal dosfstools dotnet-runtime dunst dxvk-bin easyeffects edl-ng efibootmgr electron16-bin elogind-s6 engrampa envycontrol esysusers etmpfiles fastfetch fbv fcitx5 fcitx5-anthy fcitx5-breeze fcitx5-configtool fcitx5-gtk fcitx5-lua fcitx5-material-color fcitx5-nord fcitx5-rime fcitx5-skin-fluentdark-git fcitx5-skin-materia-yanli feh firefox fish flite fluffychat foot freepats-salamander-sf2 freepats-ydp-grand-piano fuck-u-code-git galculator gamemode gamemodeswitch gamescope gdu gimp github-cli glfw glmark2 gnome-keyring go-musicfox-git go-musicfox-git-debug google-chrome gparted gpm-s6 graphite-cursor-theme-git greetd grim grub gtk-engine-murrine gtk3-demos gtk4-demos gvfs hdf5 hdparm-s6 hmcl-bin htop hugo hyprpicker i3-wm i3status imagemagick inkscape inotify-tools intel-ucode ioq3-bin iptables-s6 jdk-openjdk jdk17-openjdk jdk21-openjdk jdk8-openjdk jq jupyter-notebook kali-themes kazumi kd-bin kmscon krb5-s6 krita kvantum-theme-materia lib32-artix-archlinux-support lib32-gamemode lib32-libdxvk lib32-vkd3d libnotify libvirt lingmo-cursor-themes linux-firmware-intel linux-firmware-other linux-zen linux-zen-headers linuxqq llmfit-bin lmms-qt6-git localsend lolcat lollypop lsp-plugins lutris lvm2-s6 lxc man-db mangohud mate-polkit mate-system-monitor mate-utils materia-gtk-theme materia-kde matugen mda.lv2 mdadm-s6 mesa-utils micromamba-bin mihomo millennium minecraft-launcher minuet mkdocs-material mkdocs-material-extensions modemmanager modemmanager-s6 mpc mpd mpd-mpris mpv mpvpaper mupdf musescore nano ncmpcpp neomutt neovim network-manager-applet networkmanager networkmanager-s6 nfs-utils-s6 noto-fonts-cjk noto-fonts-emoji npm ntfs-3g numix-gtk-theme nvidia-open-dkms nvidia-prime nvidia-utils-s6 nwg-look obs-studio onlyoffice-bin opencv opendoas openldap-s6 openssh-s6 os-prober otf-font-awesome pavucontrol-gtk3 picom piliplus pipewire pipewire-alsa pipewire-audio pipewire-pulse plymouth pmbootstrap podman polybar power-profiles-daemon power-profiles-daemon-s6 powertop ppet3-bin prismlauncher proton-ge-custom-bin pv pwvucontrol pycharm-community-edition pyside6 python-capstone python-docopt python-exscript python-keystone python-paginate python-passlib python-pip python-pyclip python-pycryptodome python-pyperclip python-pyusb python-pywal python-qrcode qdl qemu-desktop qt5-graphicaleffects qt5-quickcontrols2 qt5-svg qt5-wayland qt5ct qt6ct qtcreator qtcreator-devel quickshell rdesktop refind remmina rime-ice-git river-classic rkhunter rofi rosegarden rpcbind-s6 rsync rubberband-ladspa scrcpy scrcpy-mask-bin sddm sddm-s6 sddm-sugar-dark sddm-sugar-light seahorse simplewaita-git sioyek slurp sof-firmware soundfont-fluid soundfont-sgm soundfont-unison sov spritz-wine-bin starship steam sudo sunshine supertuxkart swaybg swayfx-git swayfx-nvidia swayimg swaylock swaync syslog-ng syslog-ng-s6 tela-circle-icon-theme-all tesseract-data-chi_sim the-honkers-railway-launcher-bin thefuck thunar thunar-archive-plugin thunar-media-tags-plugin tigervnc tk ttf-arphic-ukai ttf-arphic-uming ttf-dejavu ttf-maplemononl turbostat udisks2-docs ungoogled-chromium unrar ventoy-bin vim vinagre virtualbox virtualbox-guest-iso vkd3d vosviewer vtk waybar wayvnc weston wev wf-recorder wget whois wine-gecko wine-mono wine-staging wireplumber wireshark-qt wlroots0.19 wmenu wob woff2-font-awesome wofi wpa_supplicant-s6 wps-office-cn wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei ww-manager xdg-desktop-portal-gtk xdg-desktop-portal-wlr xfce4-taskmanager xlibre-video-vesa xlibre-xserver xlibre-xserver-common xlibre-xserver-devel xlibre-xserver-src xlibre-xserver-xephyr xlibre-xserver-xnest xlibre-xserver-xvfb xorg-xdpyinfo xorg-xev xorg-xinput xwallpaper xwayland-satellite xwinwrap-git yarn yay yazi yoshimi yt-dlp zenity-gtk3 zensical zrythm zsh zynaddsubfx
```

包列表带需求甩给 LLM，我使用的 Migoo 回答[见此](../extra/debloat_migoo.md)。此外，卸载某个包之前看看还有没有其他包依赖此包，正确评估卸载这些包可能带来的影响<del>，炸了别来轰我www</del>。

## 停用不必要的服务

减少启动时间和运行时的资源占用。另外如果你按照前一段删除了某些服务，应当检查你的 init 是否需要再次手动配置更改[^1]。

[^1]: 我们 s6 是这样的 qwq

## 清理不必要的缓存和日志

Pacman 的软件包缓存和数据库，时间一长会很占空间，如果不需要可以清理了：

    # sudo pacman -Scc

用户缓存 `~/.cache/`，Arch 系操作系统下这个地方可能 AUR Helper 构建缓存会比较大，没用的可以删了。

长久积累的日志（`/var/log/`）也会占用大量存储空间。可以考虑限制日志大小。

## 定制内核

手动编译适用于你的设备的最小化内核，参考[编译最小化 Linux 内核](compile_kernel.md)。


