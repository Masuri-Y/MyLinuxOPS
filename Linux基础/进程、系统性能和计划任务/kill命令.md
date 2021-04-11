## kill命令

`kill`命令：向进程发送控制信号，以实现对进程管理,每个信号对应一个数字，信号名称以SIG开头（可省略），不区分大小写

显示当前系统可用信号： `kill -l`或者`trap -l` 

常用信号：`man 7 signal`

| 信号 | 完整名称  | 说明                               |
| ---- | --------- | ---------------------------------- |
| 1    | `SIGHUP`  | 无须关闭进程而让其重读配置文件     |
| 2    | `SIGINT`  | 中止正在运行的进程；相当于`Ctrl+c` |
| 3    | `SIGQUIT` | 相当于`ctrl+\`                     |
| 9    | `SIGKILL` | 强制杀死正在运行的进程             |
| 15   | `SIGTERM` | 终止正在运行的进程                 |
| 18   | `SIGCONT` | 继续运行                           |
| 19   | `SIGSTOP` | 后台休眠                           |

#### `kill`命令

按`PID`发送信号：

```bash
kill [-SIGNAL] pid
kill –n SIGNAL pid
kill –s SIGNAL pid
```

#### `killall`命令

按名称发送信号：

```bash
killall [-SIGNAL] commond
```

#### `pkill`命令

按模式来发送信号：

```bash
pkill [options] pattern
```

选项：

* `-SIGNAL`

* `-u uid`: effective user，生效者

* `-U uid`: real user，真正发起运行命令者

* `-t terminal`: 与指定终端相关的进程

* `-l`: 显示进程名（`pgrep`可用）

* `-a`: 显示完整格式的进程名（`pgrep`可用）

* `-P pid`: 显示指定进程的子进程