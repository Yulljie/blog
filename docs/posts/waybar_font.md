---
date:
  created: 2026-06-13
author:
  - Yulliil
---

# 驯服 Waybar 的字体

我的 Waybar 字体之前一直有问题。总是有个图标渲染不对，有一天我实在了看不下去，于是决定修理一下......
<!-- more -->

## 我以太网呢？

最开始是以太网图标显示有问题，会显示为楷体的“口”，如图：
<figure markdown="span">
  ![Image title](../assets/waybar_1.png){ height="20" }
</figure>

我的 `waybar_style.css` 中字体相关部分如下，是拿默认配置改的，我自己都没用到这么多字体，但是这个地方应该是没问题的，只能调 fontconfig 了......

仅为 waybar 调整系统 fontconfig 很明显没必要，并且很可能会搞崩整个系统的字体显示，得不偿失。幸运的是，fontconfig 支持为特定程序设置字体，那就好办啦，在用户侧添加一个仅针对 waybar 的 fontconfig 配置即可：

``` xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <match target="pattern">
        <test name="prgname" compare="eq">
            <string>waybar</string>
        </test>
	<edit name="family" mode="prepend" binding="strong">
		<string>Noto Sans</string>
		<string>Font Awesome 7 Free</string>
	    <string>Font Awesome 7 Brands</string>
        </edit>
    </match>
</fontconfig>
```

因为我的 waybar 实际上只用了 Font Awesome 和 Noto Sans 两个字体，所以 fontconfig 只设置了这两个，写好保存后重启 waybar：
<figure markdown="span">
  ![Image title](../assets/waybar_2.png){ height="20" }
</figure>

## 汉字！！

这个配置看起来没什么问题了，但是实际上并不完善，我用了一段时间后才发现问题：
<figure markdown="span">
  ![Image title](../assets/waybar_3.png){ height="20" }
</figure>

这个是我的本地音乐播放脚本的 waybar 前端，它会把一些汉字显示为异体（日文）字。我的 bar 平时很少显示汉字，只用来显示一些系统监控数据，因此用了一段时间才发现。

由于 Noto Sans 本身只是纯西文字体，不包含 CJK 文字，fontconfig 遇到 CJK 文字就 fallback 了......所以这个配置实际上还需要
<figure markdown="span">
  ![Image title](../assets/waybar_4.png){ height="20" }
</figure>

