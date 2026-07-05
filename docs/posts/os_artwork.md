---
date:
  created: 2026-07-01
  updated: 2026-07-05
author: Yulliil
tags:
  - Linux
  - Arch
  - Artix
categories:
  - Linux 使用
draft: false
---

# Your OS is Your Artwork

<del>好吧我承认这标题是有点那啥😂以及这篇只是我的碎碎念，不认同请无视</del>

起初我是看到群友系统一炸就重装，不太理解这种行为。很多问题是可以比较轻易地手动排查的，完全犯不着重装。而且重装后折腾同样的东西八成还是会炸，有这试错的时间还不如去好好看文档搜 report 呢......
<!-- more -->

## 双标（？）的咱

咱应该是在高中的时候开始玩 Arch 的，当时是听说 Arch 很难装啦......不过看着[安装指南](https://wiki.archlinuxcn.org/wiki/安装指南)依葫芦画瓢倒是也没花多少工夫就在自己的移动硬盘里（是家里的电脑，因为家里人还要用所以装到了自己的移动硬盘）装上了（Arch 的难度确实被过分夸大了呢😂），然后就是体验各种 DE 咯......

在这之前我玩过 Fedora（但也只是玩过，连 dnf 都不会用😂），觉得 GNOME 的风格比较顺眼，于是愉快安装 GNOME 咯。后来发现 GNOME 比较重，而且道听途说 GNOME 会污染系统配置，又正好看到 B 站上一个展示 Hyperland[^1] 的[视频](https://www.bilibili.com/video/BV1VF411r7Pv/)......嘛，那咋办？重装！

<del>（果然人类就是这样无法理解过去的自己的生物呐~（虽然现在咱知道 `-Rns` 一下也没什么😂</del>

Hyprland 是挺好，但是当时好像还没有进官方仓库，安装不了。我又不会用 AUR，于是克隆了 GitHub 上 Hyprland 的源码，手动处理好 submodules 后[^2]，磕磕绊绊稀里糊涂编译了一个能用的 Hyprland[^3]。

[^1]: 是故意写错的啦。当时咱第一次见到时确实看成了“Hyperland”<del>（其实后面也一直看错，直到两三天后才发现没有 e🤣</del>

[^2]: 指手动克隆仓库再一一复制到对应路径。

[^3]: 为什么你的启动命令首字母大写，为什么啊😡

## bloat（？）的系统

Hyprland 固然不错，但是咱也并没有玩的很出色，一是没有抄别人 dotfiles 的意识，二是不能耐着性子读 Hyprland Wiki 以及 eww 等小工具的文档。结果就是咱配的 Hyprland 几乎和默认配置一样😂[^4]。

加之当时还折腾其他东西，久而久之咱的 DM 菜单变得又臭又长。据不完全回忆，那时的 DM 菜单至少包含以下条目：

* Hyprland

    没错天天用的就是这个

* Sway

    其实是几乎没配置过的 SwayFX。听说 sway 比较轻量化，实测玩 mc 确实比 Hyprland 流畅[^5]<del>（于是恶堕为 mc 专属 wm😂（那你干嘛不去用 cage</del>

* GNOME \*

    包括 GNOME (Wayland)、GNOME、GNOME (X11)等条目，装个 GDM 就拉下来了<del>，高耦合，很神奇吧</del>。

* Steam

    直接启动 Steam 的大屏幕模式（其实根本用不到😅），参考 [Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Steam#无窗口管理器的大屏幕模式)

除 WM/DE 外，DM 本身咱也装了好几个，至少有 GDM、SDDM 和 Greetd+wlgreet......嘛，总之就是一堆东西也不知道装了干嘛就一直在那堆着......

再之后，咱发现 SwayFX 确实挺合胃口（？），于是折腾 SwayFX 的配置，经过大概几个晚上的努力，终于是弄好了 SwayFX + Waybar[^6]，不过其他用不到的 wm/de/dm/menu/etc 还是在那堆着😂完全没删，想着有时候可能会用到<del>，骗你的其实再也用不到了</del>。

这应该算是咱最初始的 artwork 了，是按照我的想法配出来的，不顺眼，但还算顺手。

[^4]: 仅针对“能用”作最小配置，这又何尝不是一种 [The Arch Way](https://wiki.archlinux.org/title/Arch_Linux#Simplicity)（大雾

[^5]: 家里那台的显卡是 NVIDIA GeForce GTX 960M，开个光影能明显地感受到性能差距

[^6]: 也是咱现在使用的配置的前身呐~

## One last reinstall（？？？

移动硬盘里的 Arch 一直用到我高考结束买新电脑，因为这个系统在新电脑上运行的效果并不理想，而且没用的东西太多，与其手动迁移，不如装个新的。

这个系统到如今才变成了真正意义上的我的 artwork。我学习了更多的东西来完善和维护系统。那么，我对她做过什么？

* 写了一个脚本，用于实现终端和 GTK 的动态配色

    好看喵~

* Arch -> Artix

    见[Artix 迁移指南](artix_migration.md)，systemd -> s6。btw 有些东西离开 systemd 后会出问题，需要手动调整<del>，人生苦短，干嘛不用 systemd（笑）</del>。

* 写了另一个脚本，以 waybar 为前端 mpd 为后端来听歌

    我需要的本地音乐播放工具不需要复杂大而全的 ui，只需要 prev/play-pause/next + 随机播放。

* grub -> efistub

    用不到那么多功能，而且拖慢启动时间。

* 更美观的 wm 和 bar 样式

    虽然但是药丸样式的 bar 搭配方角窗口确实不太符合大多数人的审美（？

* 自行编译的简化内核

    作为下游的 Artix 更新软件包比 Arch 更迟

* more

    特化配置太多记不清了😂以上是几个主要的

以及迁移到 Artix 后，由于混用 Arch 仓库，所以有些工具更容易挂😂<del>，这也是维护 artwork 不得不品的一环</del>。

## So...?

写了半天也不知道自己讲的和标题有什么关系。维护 artwork 就是个不断折腾的过程，出问题重装显然不是一个好的做法，尝试探索这个问题，对其机制有更深入的了解才能帮你创造更好的作品。

<del>不知所云的一篇 post，就当作是我的胡言乱语吧</del>
