---
date: 
  created: 2026-06-08
draft: false
pin: false
# readtime: 2
tags:
  - Artix
  - Linux
  - elogind
  - systemd
  - Wiki
  - 迁移
  - 翻译
categories:
  - Artix Wiki
---

# Artix 迁移指南

!!! note "翻译状态"

    本文翻译自 Artix Wiki 文章 [Migration](https://wiki.artixlinux.org/Main/Migration)，额外添加了部分注释，所有的脚注和文内“译者注”均为额外添加。

!!! note "注意"

    迁移指南仅为具备相关经验的进阶用户编写。通常，更推荐通过全新安装来上手 Artix。
<!-- more -->

!!! warning "警告"

    下文提到的所有命令均以 root 执行。

## 准备仓库

开始迁移之前，将 Arch 的 `pacman.conf` 和镜像列表替换为 Artix 的。

```bash
mv -vf /etc/pacman.conf /etc/pacman.conf.arch
curl https://gitea.artixlinux.org/packages/pacman/raw/branch/master/pacman.conf -o /etc/pacman.conf
mv -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-arch
curl https://gitea.artixlinux.org/packages/artix-mirrorlist/raw/branch/master/mirrorlist -o /etc/pacman.d/mirrorlist
cp -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.artix
```

若启用了 `multilib`，还需要在 `/etc/pacman.conf` 中取消注释以下行：

```
[lib32]
Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist-arch
```

## 清理软件包缓存

清理所有缓存，因为 Artix 提供的软件包签名不同，pacman 会报错。若您的缓存体积较大，并且想保留这些缓存，那么请不要使用 `-Scc`，但是您依然会被提示删除并重新下载部分损坏的软件包。接下来，强制同步软件包数据库：

    pacman -Scc && pacman -Syy

## 降低 pacman 安全等级

为安装 `artix-keyring`，您需要在 `/etc/pacman.conf` 中暂时降低 pacman 的安全等级。

```
#SigLevel = Required DatabaseOptional
SigLevel = Never
```

先前的选项应当为 `Required DatabaseOptional`。

## 安装 Artix PGP 密钥环。

要安装 `artix-keyring`，您需要手动签名主密钥。

```
pacman -S artix-keyring
pacman-key --populate artix
pacman-key --lsign-key 95AEC5D0C1E294FC9F82B253573A673A53C01BC2
```
!!! warning "警告"

    译者注：

    此处密钥请以原文 [Install the Artix PGP keyring](https://wiki.artixlinux.org/Main/Migration#Install_the_Artix_PGP_keyring) 段落声明为准！

## 恢复 pacman 安全等级

现在立刻恢复，不然您之后很可能会忘了：

```
SigLevel = Required DatabaseOptional
#SigLevel = Never
```

## 备份正在运行的服务列表

保存一份您正在运行的服务列表，之后需要安装对应的服务脚本。

```
systemctl list-units --state=running | grep -v systemd | awk '{print $1}' | grep service > daemon.list
```

现在您可以开始安装 Artix 组件和各服务脚本了。

## 下载 Artix 软件包

您应当从 pacman 缓存安装软件包（不强制，但是如果中途丢失网络连接，很可能会导致无 init 系统）。必须依次重新安装 `base` 和 `base-devel` 软件包组，使用 `[system]` 仓库中的替换原有的。之后，选择一个 Artix 提供的 init（`openrc`、`runit`、`s6` 或 `dinit`）和对应的服务脚本。

```
pacman -Sw base base-devel grub linux linux-headers mkinitcpio \
  rsync lsb-release esysusers etmpfiles artix-branding-base
```

运行以下命令之一，取决于您选择的 init 系统：

```
pacman -Sw openrc elogind-openrc openrc-system
```

或

```
pacman -Sw runit elogind-runit runit-system
```

或

```
pacman -Sw s6-base elogind-s6 s6-system
```

或

```
pacman -Sw dinit elogind-dinit dinit-system
```

## 移除 systemd

!!! danger "危险"

    译者注：

    操作物理机执行本段提供的命令时系统很可能会崩溃，并且可能导致本地软件包数据库（`/var/lib/pacman/local`）损坏[^1]。强烈建议做好备份再进行下一步操作，或者进入 [artixiso](https://artixlinux.org/download.php) 后在 `artix-chroot` 中继续迁移。如果不幸真的坏了，请参考 ArchWiki [pacman/Restore local database](https://wiki.archlinux.org/title/Pacman/Restore_local_database) 或 Arch Linux 中文维基 [Pacman/恢复本地数据库](https://wiki.archlinuxcn.org/wiki/Pacman/恢复本地数据库)（中文维基内容可能略有滞后）恢复软件包数据库。

现在所有的软件包都已缓存，现在可以将 systemd 抛之脑后了（以下在 pacman 的所有问题中回答 `yes`）。

```
pacman -Rdd --noconfirm systemd systemd-libs systemd-sysvcompat pacman-mirrorlist dbus
rm -fv /etc/resolv.conf
```

还原在上一步被删除的 Artix 的镜像源列表：

```
cp -vf /etc/pacman.d/mirrorlist.artix /etc/pacman.d/mirrorlist
```

完成后这一步后**必须**完成下一步，否则会没有 init 系统。如果您在远程迁移（如通过 ssh），请确保开启另一个 ssh 会话，以防第一个会话卡死（这确实发生过。systemd，坏！）。

[^1]: 笨蛋站长是在图形化环境里（swayfx 里开终端）迁移的喵，卸到一半 swayfx 崩了，也切不了其他 tty，幸运的是数据库没坏。不过只好在 artixiso 里继续迁移了（也有可能只是图形环境崩了，至少也要在 tty 里进行迁移呀）。

## 安装您选择的 init

现在您可以安装之前使用 `pacman -Sw` 下载的软件包了。非常简单：

```
pacman -S base base-devel grub linux linux-headers mkinitcpio \
  rsync lsb-release esysusers etmpfiles artix-branding-base
```

以及：

```
 pacman -S openrc elogind-openrc openrc-system
```

或

```
 pacman -S runit elogind-runit runit-system
```

或

```
 pacman -S s6-base elogind-s6 s6-system
```

或

```
 pacman -S dinit elogind-dinit dinit-system
```

!!! note "注意"

    别忘了安装一个[网络管理器](https://wiki.artixlinux.org/Main/Installation#Network_configuration)（如 connman 或 netifrc），文本编辑器和其他必要的软件包。

## 从 Artix 的仓库安装软件包

首先，确保将 `grep(1)` 的 locale 设置为 C：

```
export LC_ALL=C
```

将所有的 Arch 软件包替换为 Artix 的：

```
pacman -Sl system | grep installed | cut -d" " -f2 | pacman -S -
pacman -Sl world | grep installed | cut -d" " -f2 | pacman -S -
pacman -Sl galaxy | grep installed | cut -d" " -f2 | pacman -S -
```

如果还启用了 32 位支持：

```
pacman -Sl lib32 | grep installed | cut -d" " -f2 | pacman -S -
```

## 安装服务脚本

!!! note "注意"

    这些服务脚本均为可选，可以跳过。

安装您正在运行的服务对应的脚本，示例如下（将 `init` 换成 `openrc`、`runit`、`s6` 或 `dinit`）：

```
pacman -S --needed acpid-init alsa-utils-init cronie-init cups-init fuse-init haveged-init hdparm-init openssh-init samba-init syslog-ng-init
```

## 启用服务

#### OpenRC

```
for daemon in acpid alsasound cronie cupsd xdm fuse haveged hdparm smb sshd syslog-ng; do rc-update add $daemon default; done
```

除非您知晓其风险，否则通常必须立刻启用 *udev*；*dbus* 和 *elogind* 不再明确需要，因为这二者是按需启动的（仅限 OpenRC）。

#### Runit

```
for daemon in acpid alsasound cronie cupsd dbus elogind xdm fuse haveged hdparm smb sshd syslog-ng; do ln -s /etc/runit/sv/$daemon /etc/runit/runsvdir/default; done
```

#### s6

```
touch /etc/s6/adminsv/default/contents.d/{acpid,cronie,cupsd,elogind,xdm,fuse,haveged,hdparm,smbd,sshd,syslog-ng}
s6-db-reload
```

#### dinit

```
for daemon in acpid alsasound cronie cupsd xdm fuse haveged hdparm smb sshd syslog-ng; do dinitctl enable $daemon; done
```

## 配置网络

编辑您的网络配置文件 `/etc/conf.d/net`。这非常重要，尤其当是您在迁移一个远程主机时，很可能就失联了[^2]。根据您是否在使用持久化设备命名，您必须将 `/etc/init.d/net.lo` 符号链接到 `net.enp0s3` 或 `net.eth0`。接口名称仅用作示例，请务必确认适用于您的系统。如果不确定（且只有一个以太网接口），您可以使用一个内核命令行（下方提到的 `GRUB_CMDLINE_LINUX`）来禁用持久化设备命名以使用 `net.eth0`。OpenRC 有其自己的网络管理器 netifrc，其默认使用 DHCP 获取有线网卡的 IP。

```
vi /etc/conf.d/net
echo 'GRUB_CMDLINE_LINUX="net.ifnames=0"' >>/etc/default/grub		# disable persistent device naming
ln -s /etc/init.d/net.lo /etc/init.d/net.eth0
rc-update add net.eth0 boot
pacman -S --needed netifrc
```

创建一个基本的 `/etc/resolv.conf`，以 Cloudflare 的 DNS 为例：

```
nano /etc/resolv.conf
```
```
nameserver 1.1.1.1
```

[^2]: 原文写的很有意思：This is especially important if you're converting a remote box, or you may very well be locked out of it. 

## LVM 配置

如果您使用 LVM，则必须安装 `lvm2-init` 和 `device-mapper-init` 软件包，否则逻辑卷在重启后会失效。这些软件包已经包含于各自的 init-system 软件包组，因此您大概已经安装了，只需启用服务即可：

!!! note "注意"

    译者注：

    如同[前文](#_4)，将软件包名中的 `init` 替换为 init 系统的名称（`openrc`、`runit`、`s6` 或 `dinit`）。

#### OpenRC

```
rc-update add lvm boot
rc-update add device-mapper boot
```

#### Runit

```
ln -s /etc/runit/sv/lvm2 /etc/runit/runsvdir/default
ln -s /etc/runit/sv/device-mapper /etc/runit/runsvdir/default
```

#### s6

```
touch /etc/s6/adminsv/default/contents.d/device-mapper
s6-db-reload
```

#### dinit

```
dinitctl enable lvm2
dinitctl enable dmeventd
```

## 移除 systemd 产生的垃圾

可选移除 systemd 的一些残留账户，因为现已不再需要：

```
for user in journal journal-gateway timesync network bus-proxy journal-remote journal-upload resolve coredump; do
  userdel systemd-$user
done
rm -vfr /{etc,var/lib}/systemd
```

确保在您的引导加载程序配置中移除了所有 `init=/usr/lib/systemd/systemd` 或类似的条目，linux 内核默认启动 `/sbin/init`。此外，移除 `/etc/fstab` 中的所有 `x-systemd` 条目。

## 更新引导加载程序和 initramfs

您安装了新的 `mkinitcpio` 和 `grub`，推荐重新配置。将新的 `/etc/mkinitcpio.pacnew` 复制到 `/etc/mkinitcpio`，将 `/etc/default/grub.pacnew` 复制到 `/etc/default/grub`，使用 `mkinitcpio` 生成新的 initramfs，重装 grub，若您修改过相关文件（如在 `/etc/mkinitcpio.conf` 添加 `resume` 钩子，或者给 grub 配置添加自定义的内核参数），应当重新配置。

### 重新生成 initramfs 和 grub 配置文件

```
# 仅为 Artix 默认内核生成 initramfs
mkinitcpio -p linux
# 为所有已安装内核生成 initramfs
mkinitcpio -P

# 更新 GRUB 配置文件
update-grub
```

### 重装 GRUB

* UEFI：
```
# 大部分默认用这个
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
# 部分用户报告这个也可以
grub-install --target=x86_64-efi --efi-directory=esp_mount --bootloader-id=grub
```

* BIOS：
```
# 将 sdX 换成 sda、sdb 或其他磁盘
grub-install /dev/sdX
```

## 重启

由于原先那个臃肿的 PID1 的二进制文件已被删除，因此您现在无法正常重启。请直接使用内核的 **SysRq 触发器**：

```
sync
umount -a
mount -f / -o remount,ro
echo s >| /proc/sysrq-trigger
echo u >| /proc/sysrq-trigger
echo b >| /proc/sysrq-trigger
```

可能还需要一些额外的配置步骤，特别是在桌面功能方面。即便现在使用的是 `elogind` 而非 `consolekit`，[systemd-free.org 的配置章节](https://systemd-free.artixlinux.org/config.php)也依旧能够提供一些思路。

!!! tip "提示"

    译者注：

    前文步骤中创建的备份文件，即 `/etc/pacman.conf.arch`、`/etc/pacman.d/mirrorlist-arch` 和 `/etc/pacman.d/mirrorlist.artix`，以及[从 Artix 的仓库安装软件包](#artix_2)步骤中产生的部分 `.pacnew` 文件（如 `/etc/pacman.conf.pacnew`），在确认不再需要后，也可以给删了。

