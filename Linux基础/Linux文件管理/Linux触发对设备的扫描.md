### Linux触发对设备的扫描

在Linux上新增设备后，Linux不会立即识别出设备，需要对Linux进行重启或者触发其对设备的扫描

> Linux触发对设备的扫描

```bash
[root@mylinuxops ~]# echo '- - -' > /sys/class/scsi_host/host0/scan 
```

此条命令中`scsi_host`目录下有多个`host[*]`的目录，需要依次进行执行扫描。