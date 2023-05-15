# print_req_error: I/0 error, dev fd0, sector 0


> 一次偶然发现Ubuntu在输出报错信息
> ``` print_req_error: I/0 error, dev fd0, sector 0```
> ，查了一下是软驱相关报错，找了一个解决方案如下：

<!--more-->

## Step 1 第一步

let’s list the block devices

列出块设备

```bash
lsblk
```

if you are not using any floppy disk, there is no reason to have fd0 listed in the output of the command above.

如果你没有使用任何软盘，则在输出中不应出现fd0。

## Step 2第二步

```bash
rmmod floppy
```

“rmmod is a simple program which removes (unloads) a module from the Linux kernel.” Now run step 1 again to verify that
the fd0 is no longer listed.

rmmod是一个简单程序，它从Linux内核中删除（卸载）一个模块。再次执行第一步，检查fd0是否被删除。

## Step 3第三步

Run the following command to make the changes permanent

运行以下命令以使更改永久生效

```bash
echo "blacklist floppy" | sudo tee /etc/modprobe.d/blacklist-floppy.conf
```

This command just create a file name blacklist-floppy.conf in the /etc/modprobe.d folder and appends the text “blacklist
floppy” to it.

此命令在 /etc/modprobe.d 文件夹中创建一个文件名 blacklist-floppy.conf，并将文本"blacklist floppy"附加到该文件夹中。

## Step 4第四步

The last step is to reconfigure the kernel

最后一步是重新配置内核

```bash
dpkg-reconfigure initramfs-tools
```

---

> 作者: WAKE  
> URL: https://weiqinke.com/2023/05/unbuntu-print_req_error/  

