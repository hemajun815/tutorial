# apt upgrade时忽略指定包

当使用 `apt upgrade` 命令时，默认会将所有需要更新的包都下载更新，但是有时候我们会有不升级某个包的需求。

在使用 `apt upgrade` 命令更新之前，先执行以下命令可以实现这个需求：

```
apt-mark hold xxx
```

执行此命令可以将指定的包 `xxx` 版本锁定住，在更新时 apt 就会忽略掉这个包。

当需要取消对 `xxx` 的锁定时，执行以下命令：

```
apt-mark unhold xxx
```
