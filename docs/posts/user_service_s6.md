---
date:
  created: 2026-08-24
draft: false
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

[^2]: Artix 从 2026-05-14 起[转移到了新的 s6 前端](https://artixlinux.org/news.php#New_s6_service_manager)，`s6-db-reload` 已被弃用，因此你现在只能参考下文。

## （可选）完整 s6 监督

若 s6/s6-rc 被用作初始化系统，那么用户服务可以被纳入系统全局监督树，确保用户服务*完全*被 s6 监督。

本节所述命令需要 root 权限。首先创建一个配置文件以便于使用。

``` title="/etc/s6/config/user-services.conf"
# username for the user-services bundle
USER=<用户名>
```

创建一个名为 user-services 的 bundle。

```
mkdir -p /etc/s6/adminsv/user-services/contents.d
touch /etc/s6/adminsv/user-services/contents.d/local-s6-user
touch /etc/s6/adminsv/user-services/contents.d/local-s6-rc-user
echo "bundle" > /etc/s6/adminsv/user-services/type
````

要使 s6-rc 正常运行，需要运行一个 s6-svscan 进程。由于此脚本面向本地用户，请确保脚本中的所有命令都以本地用户身份运行。此外，还需要为 s6-svscan 选择一个扫描目录。该目录必须是本地用户拥有完整读写权限的目录。在本例中，我们使用 `/run/${USER}/service`。上游建议将其设置为 RAM 文件系统（如 tmpfs），这样与 s6-rc 的兼容性最佳。如下：

```
mkdir -p /etc/s6/adminsv/local-s6-user/dependencies.d
touch /etc/s6/adminsv/local-s6-user/dependencies.d/mount-filesystems
echo "3" > /etc/s6/adminsv/local-s6-user/notification-fd
echo "longrun" > /etc/s6/adminsv/local-s6-user/type
```

``` title="/etc/s6/adminsv/local-s6-user/run"
#!/bin/execlineb -P
envfile /etc/s6/config/user-services.conf
importas -uD "username" USER USER
foreground { install -d -o ${USER} -g ${USER} /run/${USER} }
foreground { install -d -o ${USER} -g ${USER} /run/${USER}/service }
s6-setuidgid ${USER} exec s6-svscan -d 3 /run/${USER}/service
```

该脚本会自动解析配置文件中的 `USER` 变量。但是注意您可以在 `username` 部分设置一个备用用户，以防环境变量文件出错。可以利用这一点添加所需用户。

接下来是 `local-s6-rc-user` 的部分。它是一个一次性脚本，在 `local-s6-user` 启动后运行。

```
mkdir -p /etc/s6/adminsv/local-s6-rc-user/dependencies.d
touch /etc/s6/adminsv/local-s6-rc-user/dependencies.d/mount-filesystems
touch /etc/s6/adminsv/local-s6-rc-user/dependencies.d/local-s6-user
echo "oneshot" > /etc/s6/adminsv/local-s6-rc-user/type
```

``` title="/etc/s6/adminsv/local-s6-rc-user/down"
#!/bin/execlineb -P
envfile /etc/s6/config/user-services.conf
importas -uD "username" USER USER
foreground { s6-setuidgid ${USER} s6-rc -l /run/${USER}/s6-rc -bDa change }
foreground { s6-setuidgid ${USER} rm -r /run/${USER}/service }
s6-setuidgid ${USER}
elglob -0 dirs /run/${USER}/s6-rc*
forx -E dir { ${dirs} }
	rm -r ${dir}
```

``` title="/etc/s6/adminsv/local-s6-rc-user/up"
#!/bin/execlineb -P
envfile /etc/s6/config/user-services.conf
importas -uD "username" USER USER
foreground { s6-setuidgid ${USER}
s6-rc-init -c /home/${USER}/.local/share/s6/rc/compiled -l /run/${USER}/s6-rc /run/${USER}/service }
s6-setuidgid ${USER}
exec s6-rc -l /run/${USER}/s6-rc -up change default
```

同样如前文注意此处的 `username`。

现在可以更新系统数据库了。

```
s6-db-reload
```

大功告成。现在本地 s6-rc 数据库 bundle 可以像其他服务一样被启动。

这种配置的妙处在于用户服务完全受监控，除非您指定其中止，否则其会在设备的整个声明周期内持续运行。如果直接 kill 进程，s6-supervise 会立刻将其重新拉起。您的用户可以像往常一样使用 s6-rc 和 s6 命令。这些脚本相当通用，若想添加更多用户，基本只需要复制粘贴和修改修改几个路径和变量名称，再创建一个 user-services bundle。欲使这些服务在开机时启动，只需将 user-services 添加到系统数据库的 default bundle 即可。

## 独立 s6 监督进程

若用户进程未能被纳入系统监督树，则可以参考本节。否则直接跳过本节。

给 s6-svscan 指定一个扫描目录使其正常运行。在本例中我们使用位于 `tmp` 的目录，其是 RAM 文件系统且便于任意用户写入。

```
mkdir /tmp/${USER}
mkdir /tmp/${USER}/service
s6-svscan /tmp/${USER}/service
```

这里特意将 s6-svscan 留在前台运行。在另一个终端内运行 s6-rc：

```
s6-rc-init -c /home/${USER}/.local/share/s6/rc/compiled -l /tmp/${USER}/s6-rc /tmp/${USER}/service
s6-rc -l /tmp/${USER}/s6-rc -up change default
```

如此，只要 s6-svscan 在运行，你就有一个完整本地的 s6-rc 实例可用。

## 本地 s6 用户服务的使用

和正常的 s6-rc 命令一样。不同之处在于需要使用 `-l` 参数来指定正确的数据库。以 udiskie 为例，这样启动：

```
s6-rc -l /tmp/${USER}/s6-rc -u change udiskie
```

使用这条命令关闭用户默认 bundle：

```
s6-rc -l /tmp/${USER}/s6-rc -d change default
```

注意这些命令不需要 root 权限。

## 传递环境变量

如果从根监督树运行用户服务，您可能会发现这些服务缺少环境变量。因为 s6/s6-rc 被设计为运行在一个清洁、可复现的环境中，并且其作为 PID1 确实没有任何环境变量。而一些用户服务需要某些环境变量才能正常运行。不过这些变量可以轻松从 s6-svscan 继承。以下是一些简化操作的技巧。

回看前文提到的 `/etc/s6/config/user-services.conf` 文件，给它添加更多变量：

```
# environment variables for the local s6-rc database
DISPLAY=:0
UID=1000
USER=username
```

按需修改。若脚本使用 execline（推荐），则只允许使用键值对。更多更复杂的变量定义可以保存到实际的 run 脚本中。

编辑 `/etc/s6/adminsv/local-s6-user/run` 文件：

```
#!/bin/execlineb -P
envfile /etc/s6/config/user-services.conf
importas -i UID UID
importas -i USER USER
export HOME /home/${USER}
export XDG_RUNTIME_DIR /run/user/${UID}

foreground { install -d -o ${USER} -g ${USER} /run/${USER} }
foreground { install -d -o ${USER} -g ${USER} /run/${USER}/service }
s6-setuidgid ${USER} exec s6-svscan -d 3 /run/${USER}/service
```

现在，`HOME`、`UID`、`USER` 和 `XDG_RUNTIME_DIR` 变量可供所有用户服务使用，都从 s6-svscan 继承这些变量。若要在脚本中使用，只需一行 `importas` 命令。例如：

```
#!/bin/execlineb -P
importas -i USER USER
exec xrdb /home/${USER}/.Xresources
```

