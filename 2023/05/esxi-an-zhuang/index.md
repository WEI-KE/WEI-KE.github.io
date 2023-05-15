# ESXI相关


> 之前折腾usb硬盘安装esxi遇到的一些问题
<!--more-->

## 挂载USB存储：

在不接入USB硬盘的情况下输入以下命令：

```bash
/etc/init.d/usbarbitrator stop
chkconfig usbarbitrator off
```

接入硬盘

```bash
esxcli storage core device list |grep -i usb
```

查看分区表

```bash
cd /dev/disks/
partedUtil getptbl mpx.vmhba32:C0:T0:L0

示例 ：
[root@localhost:/dev/disks] partedUtil getptbl mpx.vmhba32:C0:T0:L0 
gpt 
31130 255 63 500118192 
1 64 204863 C12A7328F81F11D2BA4B00A0C93EC93B systemPartition 128 
5 208896 8595455 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0 
6 8597504 16984063 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
```

创建新分区

```bash
partedUtil setptbl mpx.vmhba32:C0:T0:L0 gpt \
"1 64 204863 C12A7328F81F11D2BA4B00A0C93EC93B  128" \
"5 208896 8595455 EBD0A0A2B9E5443387C068B6B72699C7  0" \
"6 8597504 16984063 EBD0A0A2B9E5443387C068B6B72699C7  0" \
"3 16984064 500118158 AA31E02A400F11DB9590000C2911D1B8 0"
```

再次查看

```bash
partedUtil getptbl mpx.vmhba32:C0:T0:L0
```

格式化分区

```bash
vmkfstools -C vmfs6 -b 1m -S UsbDatastore mpx.vmhba32:C0:T0:L0:3
```

参考来源：[https://vt.wooomooo.com/archives/39786](https://vt.wooomooo.com/archives/39786)

## SATA直通

查看SATA控制器

```bash
lspci -v | grep "Class 0106" -B 1
```

示例：

```bash
[root@localhost:/dev/disks] lspci -v | grep "Class 0106" -B 1 
0000:00:1f.2 Mass storage controller SATA controller: Intel Corporation Lynx Point AHCI Controller [vmhba0] 
         Class 0106: 8086:8c02
```

编辑直通列表

```bash
vi /etc/vmware/passthru.map
```

在末尾添加

```bash
#Intel Corporation Lynx Point AHCI Controller
8086   8c02    d3d0    fasle

替换第二位设备ID即可
```

重启

```bash
reboot
```

参考来源：https://vt.wooomooo.com/archives/13671

## 解决 未配置任何 coredump 目标。无法保存主机核心转储

查看coredump状态。

```bash
esxcli system coredump file list 
```

看到最后一条的Actiive状态为false。

![](/post-images/1683023840294.jpeg)

启用文件

```bash
esxcli system coredump file set -e true
```

5.再次查看coredump状态，已经改为true。

![](/post-images/1683023882209.jpeg)

6.回到ESXI，告警信息已经没有了。

7.如果dumpfile不存在，要将 ESXi 配置为在 VMFS 上生成文件形式的 coredump，请执行以下操作：

使用 SSH 连接到 ESXi 主机；

运行以下命令添加用作 coredump 的新文件：

```bash
esxcli system coredump file add
```

-d可以指定用于 coredump 文件的vmfs数据存储。如果未提供此选项，将自动选择大小足够的数据存储。

-f可以指定 coredump 文件的文件名。如果未提供此选项，则会创建唯一名称。

例如：

```bash
esxcli system coredump file add -d UsbDatastore -f test
```

运行以下命令获取具有访问权限的所有转储文件的列表：

```bash
esxcli system coredump file list
```

您会看到类似以下内容的输出：

![](/post-images/1683023905292.jpeg)

注意：如果没有指定 coredump 文件，则运行命令不会显示任何输出。

运行以下命令设置主机的转储文件：

```bash
esxcli system coredump file set -p /vmfs/volumes/DATASTORE_UUID/vmkdump/FILENAME
```

例如：

```bash
esxcli system coredump file set -p /vmfs/volumes/UsbDatastore/vmkdump/FCE0EAC0-D837-11DD-8EAD-F832E4BE8554.dumpfile
```

运行以下命令验证转储文件是否已配置并且处于活动状态：

```bash
esxcli system coredump file list
```

您会看到类似以下内容的输出：

![](/post-images/1683023905292.jpeg)

输出结果表明该文件的活动状态和已配置状态为 True。

## 解决 系统日志存储在非持久存储中

在存储内新建存放主机日志的文件夹，并记录一下这个存储路径
例如 [UsbDatastore] Log/

![](/post-images/1683024020085.jpeg)

修改主机-配置-高级系统设置-编辑

![](/post-images/1683024042115.jpeg)

修改 Syslog.global.logdir 值为新建的文件夹的存储路径

![](/post-images/1683024066033.jpeg)

点击确定，警告消失

## 开启嵌套虚拟化

```bash
vi /etc/vmware/config
vhv.enable = "TRUE"
```

---

> 作者: WAKE  
> URL: https://weiqinke.com/2023/05/esxi-an-zhuang/  

