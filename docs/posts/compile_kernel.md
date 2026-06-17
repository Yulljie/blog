---
authors:
  - Yulliil
categories:
  - Linux 使用
  - 学习
tags:
  - Linux
  - 内核
  - 编译
date:
  created: 2026-06-17
draft: false
---

# 编译最小化 Linux 内核

之前尝试手动编译最小化 Linux 7.1-rc5 的时候没有记录，最近正好 Linux 主线发布了 7.1，来更新一下内核，顺便做个笔记。
<!-- more -->

## 获取源码

从 [kernel.org](https://kernel.org) 下载并解压内核源码包，或者克隆 GitHub 上的内核源码仓库。

Linus Torvalds 的主线内核：

    $ git clone https://github.com/torvalds/linux.git

Greg Kroah-Hartman 的稳定分支：

    $ git clone https://github.com/gregkh/linux.git

准备好源码后，进入源码目录，继续接下来的流程。

## 配置选项

可以直接使用 `make localmodconfig` 来扫描所有已加载的内核模块，剔除其余未加载的模块。然而这样做的问题在于会排除一些**目前没有加载而将来可能会加载**的内核模块，导致未来无法正确识别某些设备（比如执行的时候没插 U 盘，编译好的内核可能会无法识别 U 盘）。如果计划长期使用自行编译的内核，建议使用 [modprobed-db](https://wiki.archlinux.org/title/Modprobed-db) 来记录一段时间内（比如一周，足以覆盖大部分需要用到的模块）系统加载过的内核模块。如果只是尝尝鲜，可以在接入常用设备后直接 `make localmodconfig`。

由于咱已经有一个最小化内核，所以可以直接重启到这个内核然后复制配置（笑）：

    $ zcat /proc/config.gz > .config

!!! note "注意"

    如果你使用的是发行版提供的内核，这条命令会复制发行版的内核配置。Anyway，这条命令只是复制当前正在运行的内核的配置。

    此外部分配置选项会随内核更新而更改/移除，编译时会要求手动指定。如果不想一直按敲键盘，可以在复制配置后执行 `make olddefconfig` 使用默认值。

!!! warning "警告"

    [ArchWiki](https://wiki.archlinux.org/title/Kernel/Traditional_compilation) 提到需要检查 `.config` 的 `CONFIG_LOCALVERSION`，手动指定版本，否则可能会覆盖已有的内核：

    > If you are compiling a kernel using your current `.config` file, do not forget to rename your kernel version "`CONFIG_LOCALVERSION`" in the new `.config` or in the *General Setup*> *Local version* - *append to kernel release* option using one of the user interfaces listed under [#Advanced configuration](https://wiki.archlinux.org/title/Kernel/Traditional_compilation#Advanced_configuration). If you skip this, there is the risk of overwriting one of your existing kernels by mistake.

    这似乎没有必要，我的选项如下：

    ``` console
    $ zcat /proc/config.gz | grep LOCALVERSION
    CONFIG_LOCALVERSION=""
    CONFIG_LOCALVERSION_AUTO=y
    ```

## 开始编译！

### 编译内核

直接执行 `make`。如果要加速编译，可以指定一下核心数量：

```
# 使用当前最大核心数
$ make -j$(nproc)
```

此外还可以计个时：

```
$ time make -j$(nproc)
```

### 构建模块

    $ make modules

## 安装内核和模块

### 安装构建完成的模块

    # make modules_install

#### NVIDIA

使用 dkms 安装 Nvidia 驱动。So f\*\* you Nvidia.

     # dkms install nvidia/595.71.05 -k 7.1.0

此处按实际情况替换驱动版本。

### 复制内核

    将内核复制到 `/boot` 并重命名，本文编译的是 Linux7.1，因此重命名为 `vmlinuz-linux71`：

    # cp -v arch/x86_64/boot/bzImage /boot/vmlinuz-linux71

## 生成 initramfs

    # mkinitcpio -k 7.1.0 -g /boot/initramfs-linux71.img

`-k` 选项指定的版本名称需要和 `/usr/lib/modules/` 下的目录名称一致。

!!! tip "提示"

    此时你可能会发现生成的 initramfs 镜像体积极大：

    ```console
    $ ls -hl /boot/
    total 435M
    ...
    -rwxr-xr-x 1 root root 155M Jun 17 16:19 initramfs-linux71.img*
    -rwxr-xr-x 1 root root  22M May 28 12:56 initramfs-linux71rc5.img*
    -rwxr-xr-x 1 root root  66M Mar 21 09:17 initramfs-linux-fallback.img*
    -rwxr-xr-x 1 root root  23M Jun  2 13:56 initramfs-linux.img*
    ...
    ```
    
    可以看到 `initramfs-linux71.img` 的体积来到了 155M，这是因为安装的模块文件们包含了大量的调试符号，不需要的话可以去除，能够有效减小体积：

    ```console
    # 解压模块
    # find /lib/modules/7.1.0/ -name "*.ko.zst" -exec zstd -d --rm {} +
    # 移除调试符号
    # find /lib/modules/7.1.0/ -name "*.ko" -exec strip --strip-debug {} +
    # 压回去
    # find /lib/modules/7.1.0/ -name "*.ko" -exec zstd --rm {} +
    ```
    
    完成后还需要重新生成 initramfs。实践中模块压缩方式可能与这里有所不同，按需修改第一条和第三条命令。

## 重新生成 bootloader 配置

根据你的系统配置重新生成 bootloader 配置。我的方案是 EFISTUB（用于主内核 `linux-zen`）+ UEFI Shell（用于测试和其他临时需求），只需要创建一个 `.nsh` 脚本，写入启动命令即可。

