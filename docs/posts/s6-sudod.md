---
date:
  created: 2026-06-29

tags:
  - Artix
  - Linux
  - s6
  - s6-sudo
  - s6-sudod
categories:
  - Linux 使用
---

# 创建一个 s6-sudod 服务

之前发现系统里面有个 s6-sudo，但是它不能直接使用<del>，而且我也不知道如何使用😂</del>，原本以为是和 sudo 一样的用法，但是我又发现居然还存在 s6-sudod 和 s6-sudoc。嘛，这下这事就不简单了😋......
<!-- more -->

很容易可以想到这玩意是 server-client 架构，应该是 s6-sudoc 负责接收命令发给 s6-sudod，s6-sudod 负责执行，那么 s6-sudod 就需要以高权限来运行——做成 service 刚刚好。看看但是我依旧不清楚这些工具的工作原理细节😭，只能看官方文档是怎么写的了。

## 官方示例

幸运的是[官方文档](https://skarnet.org/software/s6/s6-sudod.html)给出了一个用例，而且也是做成了 service 的形式：

```
#!/command/execlineb -P
fdmove -c 2 1
fdmove 1 3
s6-envuidgid serveruser
s6-ipcserver -U -1 -- serversocket
s6-ipcserver-access -v2 -l0 -i rules --
exec -c
s6-envdir env
s6-sudod
sargv
```

<del>嗯？这是什么？误闯天家了😭</del>
