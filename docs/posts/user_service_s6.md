---
date:
  created: 2026-08-24
draft: true
pin: false
# readtime: 2
tags:
  - Artix
  - Linux
  - s6
  - Wiki
  - 翻译
  - 服务
categories:
  - Artix Wiki
---

# 使用 s6/s6-rc 管理用户服务

!!! note "翻译状态"

    本文翻译自 Artix Wiki 文章 [Local User Services on s6/s6-rc](https://wiki.artixlinux.org/Main/LocalUserServicesOns6)。

只需进行少量配置，即可以本地用户创建和运行 s6/s6-rc 服务，且无需提权。

<!-- more -->

!!! note "注意"

    这并不局限于 s6，其他初始化系统上也可以这么做。用户可以根据需要在一个运行着 OpenRC 的系统上配置 s6/s6-rc 用户进程。如果您的初始化系统支持管理用户服务（如 dinit），那么您应当不需要这份指南。

## 初始配置

若 s6/s6-rc 不是您的初始化系统，请先安装 `s6-rc`。

```
pacman -S s6-rc
```

这条命令会拉取全部所需依赖，不会替换您目前的初始化系统或其他类似物。

接下来，选择一个目录用于存储服务文件。本文假定 `~/.local/share/s6` (XDG_DATA_DIR)。创建 `rc` 和 `sv` 目录。

```
~/.local/share
└── s6
    ├── rc
    └── sv
```

## 示例服务

我们以 udiskie————一个用于自动挂载 usb 设备的用户服务为例。首先，创建目录：

```
mkdir ~/.local/share/s6/sv/udiskie
```

然后在目录内创建一个 `run` 和 `type` 文件。如下：

``` title="~/.local/share/s6/sv/udiskie/run"
#!/bin/execlineb -P
exec udiskie
```

``` title="~/.local/share/s6/sv/udiskie/type"
longrun
```

execline 是一个轻量化、非交互式的脚本语言，与 s6 和 s6-rc 集成优秀[^1]，因此本文将其作为 `run` 脚本的解释器。实际上任意脚本语言都可以作为解释器，只要 shebang 有效就能正常运行。唯一的例外是一次性脚本（`oneshot`），由于 s6-rc 的内部机制，一次性脚本必须使用 execline。不过这个脚本可以是简单到调用另一个脚本（如 `exec sh /path/to/shell/script），因此实际上没什么限制。

接下来，创建一个名为 `default` 的 bundle，包含 udiskie 和其他服务，以便使用一行命令拉起它们。

```
mkdir -p ~/.local/share/s6/sv/default/contents.d
touch ~/.local/share/s6/sv/default/contents.d/udiskie
echo "bundle" > ~/.local/share/s6/sv/default/type
```

!!! note "注意"

    阅读上游关于[源目录](https://skarnet.org/software/s6-rc/s6-rc-compile.html)和[服务目录](https://skarnet.org/software/s6/servicedir.html)的文档以便知晓您可以在这些地方放什么。

[^1]: 实际上他们都出自 [skarnet](https://skarnet.org/) 之手。

## 创建 s6-rc 数据库

!!! note "注意"

    Artix 的 `s6-base` 包提供了执行以下流程的封装脚本。您只需执行 `s6-db-reload -u` 即可更新用户数据库。其假设您将服务存储在 `~/.local/share/s6/sv`。下文仅供参考[^2]。

现在服务已经写成，可以创建 s6-rc 数据库了：

```
s6-rc-compile ~/.local/share/s6/rc/compiled-$(date +%s) ~/.local/share/s6/sv
```

该命令接受两个参数。第一个是数据库路径，第二个是源目录路径。为确保数据库名称唯一性，其格式为 `compiled-$(date +%s)`，即将当前 unix 时间戳作为数据库名称。可以取任意名称，但强烈推荐使用工具为每次编译数据库生成独立的名称。

接下来创建符号链接。其必须是绝对路径。将以下的 `compiled-timestamp` 替换为实际的数据库名称：

```
ln -sf /home/${USER}/.local/share/s6/rc/compiled-timestamp /home/${USER}/.local/share/s6/rc/compiled
```

!!! warning "警告"

    如果 `compiled` 符号链接已经存在，则必须以原子方式将其替换，而 `ln` 提供的覆盖是非原子的。本文省略此部分，更多信息参见上游关于[数据库管理](https://skarnet.org/software/s6-rc/faq.html#compiledmanagement)的文档。

手动执行这个命令极为繁琐，您可以使用脚本来简化这一流程。Artix 在使用类似的脚本来管理仓库中的 s6 服务。您可以按需修改：

```
#!/bin/sh

DATAPATH="/home/${USER}/.local/share/s6"
RCPATH="${DATAPATH}/rc"
DBPATH="${RCPATH}/compiled"
SVPATH="${DATAPATH}/sv"
SVDIRS="/run/${USER}/s6-rc/servicedirs"
TIMESTAMP=$(date +%s)

if ! s6-rc-compile "${DBPATH}"-"${TIMESTAMP}" "${SVPATH}"; then
    echo "Error compiling database. Please double check the ${SVPATH} directories."
    exit 1
fi

if [ -e "/run/${USER}/s6-rc" ]; then
    for dir in "${SVDIRS}"/*; do
        if [ -e "${dir}/down" ]; then
            s6-svc -x "${dir}"
        fi
    done
   s6-rc-update -l "/run/${USER}/s6-rc" "${DBPATH}"-"${TIMESTAMP}"
fi

if [ -d "${DBPATH}" ]; then
    ln -sf "${DBPATH}"-"${TIMESTAMP}" "${DBPATH}"/compiled && mv -f "${DBPATH}"/compiled "${RCPATH}"
else
    ln -sf "${DBPATH}"-"${TIMESTAMP}" "${DBPATH}"
fi

echo "==> Switched to a new database for ${USER}."
echo "    Remove any old unwanted/unneeded database directories in ${RCPATH}."
```

以本地用户运行该脚本即可自动更新数据库。

## （可选）完整 s6 监督

若 s6/s6-rc 被用作初始化系统，那么用户服务可以被纳入系统全局监督树。这可以确保用户服务*完全*被 s6 监督。


[^2]: Artix 从 2026-05-14 起[转移到了新的 s6 前端](https://artixlinux.org/news.php#New_s6_service_manager)，`s6-db-reload` 已被弃用，因此你现在只能参考下文。
