# systemd

## 简述

systemd负责linux的启动管理，可以代替原有的sysvinit，目前理解，主要功能是负责系统的启动，启动依赖于target（单元）还管理，target单元不做任何事情，而是service单元绑定到target单元，相当于target为分组。target与sysv的runlevel存在一种对应关系，但是比runlevel更加灵活（文档上面这么说，需要进一步深入的了解），详细的对关系下面在细说。

>*systemd* 是一个 Linux 系统基础组件的集合，提供了一个系统和服务管理器，运行为 PID 1 并负责启动其它程序。功能包括：支持并行化任务；同时采用 socket 式与 [D-Bus](https://wiki.archlinux.org/index.php/D-Bus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 总线式激活服务；按需启动守护进程（daemon）；利用 Linux 的 [cgroups](https://wiki.archlinux.org/index.php/Cgroups) 监视进程；支持快照和系统恢复；维护挂载点和自动挂载点；各服务间基于依赖关系进行精密控制。*systemd* 支持 SysV 和 LSB 初始脚本，可以替代 sysvinit。除此之外，功能还包括日志进程、控制基础系统配置，维护登陆用户列表以及系统账户、运行时目录和设置，可以运行容器和虚拟机，可以简单的管理网络配置、网络时间同步、日志转发和名称解析等。

## 单元

### 描述

一个单元就相当于一个应用的启动配置文件，包括应用的描述，可执行文件位置，依赖，启动顺序，启动的环境变量等

>一个单元配置文件可以描述如下内容之一：系统服务（`.service`）、挂载点（`.mount`）、sockets（`.sockets`） 、系统设备（`.device`）、交换分区（`.swap`）、文件路径（`.path`）、启动目标（`.target`）、由 systemd 管理的计时器（`.timer`）。详情参阅 [systemd.unit(5)](https://jlk.fjfi.cvut.cz/arch/manpages/man/systemd.unit.5) 。
>
>使用 `systemctl` 控制单元时，通常需要使用单元文件的全名，包括扩展名（例如 `sshd.service` ）。但是有些单元可以在 `systemctl` 中使用简写方式。

### 系统服务(.service)

一般服务所在目录

```shell
/usr/lib/systemd/system
# 开机启动service会在该目录创建一个软连接
# 如 sshd.service -> /usr/lib/systemd/system/sshd.service
/etc/systemd/system 
```

.service 示例文件

```ini
[Unit]
Description=OpenSSH Daemon
Wants=sshdgenkeys.service
After=sshdgenkeys.service
After=network.target

[Service]
ExecStart=/usr/bin/sshd -D
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target

# This service file runs an SSH daemon that forks for each incoming connection.
# If you prefer to spawn on-demand daemons, use sshd.socket and sshd@.service.
```



#### 挂载点(.mount)

#### sockets(.sockets)

#### 系统设备(.device)

#### 交换分区(.swap)

#### 文件路径(.path)

#### 启动目标(.target)

#### 计时器(.timer)





参考文档：

1. [arch百科](https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

