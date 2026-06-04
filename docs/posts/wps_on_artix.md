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
date: 2026-05-31
draft: true
pin: false
readtime: 0
---

# Artix Linux 上使用 WPS

## WPS 依赖 systemd

https://bbs.wps.cn/topic/8060

![bbs post](../assets/bbs.wps.cn_8060_light.png#only-light)
![bbs post](../assets/bbs.wps.cn_8060_dark.png#only-dark)

## Flatpak

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

## 软链接

```
sudo ln -s /usr/lib/libelogind.so /usr/lib/libsystemd.so.0
```
