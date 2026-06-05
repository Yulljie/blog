---
authors:
  - Yulliil
categories:
  - Linux 使用
tags:
  - Artix
  - WPS
  - 办公
  - systemd
  - elogind
  - Linux
date: 2026-06-05
draft: false
pin: false
readtime: 0
---

# Artix Linux 上使用 WPS

从 Arch Linux 迁移到 Artix Linux 后，某次编辑文档时我发现从 AUR 安装的 WPS365 无法运行了，终端运行也不输出任何有效信息，只能<del>算一卦</del>猜测原因所在。而这两个发行版最大的区别就是 init 系统，很容易想到 WPS 可能依赖 systemd<!-- more -->，于是去 bing 了一下，发现还真有相关讨论：

[https://bbs.wps.cn/topic/8060](https://bbs.wps.cn/topic/8060)

<figure markdown="span">
![bbs post](../assets/bbs.wps.cn_8060_light.png#only-light)
![bbs post](../assets/bbs.wps.cn_8060_dark.png#only-dark)
<figcaption>截图</figcaption>
</figure>

我贫瘠的大脑完全无法理解 WPS 为什么依赖 systemd，但是文档还是要写的，然而该帖只提到了存在这个依赖，并没有相关的解决方法，只能自己想办法解决。

## Flatpak

Flatpak 是一个比较简便的解决方法，只需要一行命令即可安装 WPS：

``` console
$ flatpak install cn.wps.wps_365
```

然而 Flatpak 会把所有依赖库都下载下来而不使用系统库，这样做确实保证了兼容性，但是会狠狠肘击寸土寸金的存储空间😢，参见某次更新的输出：

``` console
$ flatpak update cn.wps.wps_365 
Looking for updates…


        ID                                           Branch      Op Remote  Download
 1. [✓] org.freedesktop.Platform.GL.default          25.08       u  flathub    52.4 MB / 142.3 MB
 2. [✓] org.freedesktop.Platform.GL.default          25.08-extra u  flathub    24.4 MB / 142.4 MB
 3. [✓] org.freedesktop.Platform.GL.nvidia-595-58-03 1.4         u  flathub     3.3 kB / 304.1 MB
 4. [—] org.freedesktop.Platform.GL.nvidia-595-71-05 1.4         i  flathub   142.7 MB / 304.3 MB
 5. [ ] org.freedesktop.Platform.Locale              25.08       u  flathub < 379.1 MB
 6. [ ] org.freedesktop.Platform.VAAPI.Intel         25.08       u  flathub  < 13.8 MB
 7. [ ] org.freedesktop.Platform.VAAPI.nvidia        25.08       u  flathub  < 50.3 kB
 8. [ ] org.freedesktop.Platform.codecs-extra        25.08-extra u  flathub  < 14.4 MB
 9. [ ] org.freedesktop.Platform                     25.08       u  flathub < 252.6 MB
10. [ ] cn.wps.wps_365                               stable      u  flathub < 906.4 MB

Updating 10/10… ████████████████████ 100%  2.7 MB/s  00:0009
```

可以看到 Flatpak 甚至拉下来了一套 NVIDIA 驱动。这个做法要求系统驱动程序更新后一同更新 Flatpak 里的驱动，否则就用不了了。。。单在处理 NVIDIA 驱动这一点上，有群友提到 [Snap 做的比 Flatpak 好](https://yoimiyalove.top/2026/05/13/snap/)，但是 Snap 本身问题也比较多，而且除了 Ubuntu，还有哪个发行版会跑去用 Snap 呀......

## 软链接

实际上我最开始使用 Flatpak 的时候并没有注意到拉下来了独立的 NVIDIA 驱动，直到群友提到 Flatpak 的问题，我才发现这几大坨，这下不得不扔掉 Flatpak 了。

由于是从 Flatpak 换成其他方案，我最开始想到的也是容器，有 distrobox 可用。不过我个人还是偏好不要容器，于是去问了万能的 Gemini（AI Mode）有没有其他方案。

Gemini 提到了常规的容器方法，还另外提到了 elogind 继承了 systemd 的大部分接口，可以尝试把 `libelogind.so` 链接到 `libsystemd.so.0`。<del>欸，我之前怎么就没想到呢？</del>

AI 说的[不能全信](https://wiki.archlinuxcn.org/wiki/建议阅读/给新用户的关于如何不去弄坏 Arch Linux 系统的建议#谨慎对待 LLM 给出的内容)，但是用来提供思路还是很好的。我先尝试了链接到 `libsystemd.so`，未果；再尝试了 `libsystemd.so.0`，这次 WPS 成功跑起来了！<del>哈 Gemi 赛高！</del>

```console
# ln -s /usr/lib/libelogind.so /usr/lib/libsystemd.so.0
```

接下来就是愉快清理 Flatpak 啦，命令在此不再赘述。
