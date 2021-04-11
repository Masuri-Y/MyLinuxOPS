## Linux作业管理

Linux的作业分为两种，前台作业和后台作业。

* 前台作业：通过终端启动，且启动后一直占据终端

* 后台作业：可通过终端启动，但启动后即转入后台运行（释放终端）

让前台作业运行于后台：

* 运行中的作业：`ctrl + z`
* 尚未启动的作业：`COMMAND &`

注意：后台作业虽然被送往后台运行，但其依然与终端相关；退出终端，将关闭后台作业。如果希望送往后台后，剥离与终端的关系可以使用以下两种方法。

1. 使用`nohup`配合`&`来执行命令

   ```bash
   nohup COMMAND &> /dev/null &
   ```

2. 使用`screen`命令创建出窗口，在窗口中执行命令。

   ```bash
   screen; COMMAND
   ```

查看当前终端所有作业：

```bash
jobs
```

作业控制：

* 把指定的后台作业调回前台

  ```bash
  fg [[%]JOB_NUM]
  ```

* 让送往后台的作业在后台继续运行

  ```bash
  bg [[%]JOB_NUM]
  ```

* 终止指定的作业

  ```bash
  kill [%JOB_NUM]： 
  ```

* 暂停后端的作业

  ```bash
  kill 19 [%JOB_NUM]
  # 或
  killall 19 PROGRAM
  ```

* 继续执行后段作业

  ```bash
  kill 18 [%JOB_NUM]
  # 或
  killall 18 PROGRAM
  ```

  