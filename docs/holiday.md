# 假期干啥😈

## 二次元🥰 ACG

* 终将成为你

    夯爆了的必看番剧

* 污秽不堪的你最可爱了

    好像是有些非常污污污黑泥的很夯的漫画，到时候先看看能不能接受

* 超时空辉夜姬

    必看，不解释

<del>woc 百合头子！</del>

## 学习📖 Learn

* 挂的科😭

    数学苦手说是😭

* 乐理，编曲🎶

    （长期任务，假期就当作是开始吧😭）

* 英语六级

    （不要求完成）

* LXC 容器

    （Optional）

* 去 cgroup 化
    
    !!! tip "Whyyyyyyyyyyyyyyyyy?"

        现代 Linux 内核的 cgroup v2 的设计为 systemd 高度妥协，而我目前在使用的 s6 有着一套完善的进程追踪系统且不依赖 cgroup，然后我还在用 elogind，就很......这很 bloat[^1]，对吧🤣，所以我打算给 cgroup 晾着了。假期还可以进一步研究能不能调整内核给 cgroup 扬了。

    * 替代 elogind

        换成 seatd、acpid 等组件。

    * 编译不依赖 elogind 的 polkit 并考虑自动化打包（这就得自己建立仓库了www）

        我目前不确定 polkit 是否强依赖 systemd，如果是的话得抛弃 polkit。

    * 编译不依赖 elogind 的 Fcitx5 并考虑自动化打包
        
        这个貌似可以安全去除依赖，换成 basu 等库。

* 维护 pacman 仓库及自动化

    原因见上。如此，还需要学习更多的东西（服务器维护，Arch 打包）等。

[^1]: 指系统有两套进程追踪机制，一套是 s6 维护的，另一套是图形环境里 elogind 通过 cgroup 维护的

## 开发⌨️ Dev

* 自己的小工具🛠
    * [liblrc](https://github.com/Yulljie/liblrc)
        
        C++ 的歌词解析库

    * Listen!
        
        Waybar 前端的机内听歌识曲

    * Record
        
        Waybar 前端的录屏脚本

