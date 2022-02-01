---
title: Ubuntu扩展根目录空间（扩展原有分区）
date: 2021-12-01 14:39:43
tags:
---
# 0.概念说明
一切开始的开始 先说明一些概念

>**物理卷（Physical Volume）即PV：**  物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
>**卷组（Volume Group）即VG：** LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
>**逻辑卷（Logical Volume）即LV：** LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。

总之 LV建立在VG之上，VG建立在PV之上，是PV的集合，PV即物理卷。

**本文的总体思路为： 扩容PV-> 扩容LV -> 扩容根目录空间**

# 1.VM添加空间
使用vmware增加空间 ↓
![在这里插入图片描述](https://img-blog.csdnimg.cn/6bcae82012ad45d6afcd2a9a55bda7a4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55m96Iy2U3lyYVRp,size_20,color_FFFFFF,t_70,g_se,x_16)

# 2.使用fdisk扩展原有分区
此处参考了[Linux下使用fdisk扩展分区容量](https://www.linuxprobe.com/linux-fdisk-size.html)，原文关键部分摘录如下

> 对/dev/sda4进行扩容

```shell 
[root@linuxprobe ~]: fdisk /dev/sda #对/dev/sda进行操作

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): p        #查看分区表信息

Disk /dev/sda: 32.2 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0005210c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          26      204800   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              26        1301    10240000   83  Linux
/dev/sda3            1301        1497     1572864   82  Linux swap / Solaris
/dev/sda4            1497        2611     8952832   83  Linux

Command (m for help): d           #删除分区
Partition number (1-4): 4         #删除需要扩容的分区（这里为sda4）

Command (m for help): p       #再次查看分区信息，/dev/sda4已被删除

Disk /dev/sda: 32.2 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0005210c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          26      204800   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              26        1301    10240000   83  Linux
/dev/sda3            1301        1497     1572864   82  Linux swap / Solaris

Command (m for help): n      #创建新的分区
Command action
   e   extended
   p   primary partition (1-4)
p             #创建为主分区
Selected partition 4
First cylinder (1497-3916, default 1497):          #经对比，正好和上一个磁盘柱一致，默认即可
Using default value 1497
Last cylinder, +cylinders or +size{K,M,G} (1497-3916, default 3916): 
Using default value 3916              #直接默认就可以

Command (m for help): p               #查看分区表信息

Disk /dev/sda: 32.2 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0005210c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          26      204800   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              26        1301    10240000   83  Linux
/dev/sda3            1301        1497     1572864   82  Linux swap / Solaris
/dev/sda4            1497        3916    19436582   83  Linux

Command (m for help): wp       #保存并退出，如果创建有误，直接退出不要保存即可
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```
# 3.刷新PV空间
对分区扩容完成后，会发现空间依旧没有增加，此时需要先刷新pv大小，使用``pvresize``:
```shell
[root@syra-ti_ubuntu~]: pvdisplay #先列出pv
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               ubuntu-vg
  PV Size               <19.00 GiB / not usable 16.50 KiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              9983
  Free PE               0
  Allocated PE          9983
  PV UUID               2CF0fb-auL8-N1Xk-Lpo1-pdzo-MLN5-R0QAIX
  
[root@syra-ti_ubuntu~]: pvresize /dev/sda4 #调用pvresize进行刷新
  Physical volume "/dev/sda3" changed
  1 physical volume(s) resized / 0 physical volume(s) not resized

[root@syra-ti_ubuntu~]: pvdisplay #再次查看pv大小 发现pv大小已经正常
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               ubuntu-vg
  PV Size               <39.00 GiB / not usable 16.50 KiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              9983
  Free PE               0
  Allocated PE          9983
  PV UUID               2CF0fb-auL8-N1Xk-Lpo1-pdzo-MLN5-R0QAIX
  
```
# 4.为LV增加空间
对pv扩容完成后，会发现lv的空间依旧没有增加，此时需要扩展lv大小，使用``lvextend``:
```shell
[root@syra-ti_ubuntu~]: lvdisplay #列出lv信息 发现lv空间没有增加
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                3THGqb-Me9D-KFEK-QxT5-dP8Y-hUqa-knbAa3
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2021-10-12 06:49:31 +0000
  LV Status              available
  # open                 1
  LV Size                <19.00 GiB
  Current LE             9983
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
[root@syra-ti_ubuntu~]: lvextend -L +20G /dev/ubuntu-vg/ubuntu-lv /dev/sda4 #调用lvextend进行扩容
[root@syra-ti_ubuntu~]: lvdisplay #再次列出lv信息 发现lv空间已经增加
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                3THGqb-Me9D-KFEK-QxT5-dP8Y-hUqa-knbAa3
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2021-10-12 06:49:31 +0000
  LV Status              available
  # open                 1
  LV Size                <39.00 GiB
  Current LE             9983
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

```
# 5.扩展根目录空间
对lv扩容完成后，会发现根目录的空间依旧没有增加，此时需要刷新根目录大小，使用``resize2fs``:
```shell
[root@syra-ti_ubuntu~]: df -h #发现/根目录空间还是没有增加
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.9G     0  1.9G   0% /dev
tmpfs                              393M  1.8M  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   19G   18G   19G  99% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop0                          56M   56M     0 100% /snap/core18/2128
/dev/loop1                          56M   56M     0 100% /snap/core18/2253
/dev/loop3                          43M   43M     0 100% /snap/snapd/14066
/dev/loop2                         117M  117M     0 100% /snap/docker/1125
/dev/loop4                          33M   33M     0 100% /snap/snapd/13270
/dev/sda2                          976M  150M  759M  17% /boot
tmpfs                              393M     0  393M   0% /run/user/0

[root@syra-ti_ubuntu~]: resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv #使用resize2fs进行刷新
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 5
The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 10222592 (4k) blocks long.

[root@syra-ti_ubuntu~]: df -h #再次查看，发现/根目录空间已经增加
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.9G     0  1.9G   0% /dev
tmpfs                              393M  1.8M  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   39G   18G   19G  49% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop0                          56M   56M     0 100% /snap/core18/2128
/dev/loop1                          56M   56M     0 100% /snap/core18/2253
/dev/loop3                          43M   43M     0 100% /snap/snapd/14066
/dev/loop2                         117M  117M     0 100% /snap/docker/1125
/dev/loop4                          33M   33M     0 100% /snap/snapd/13270
/dev/sda2                          976M  150M  759M  17% /boot
tmpfs                              393M     0  393M   0% /run/user/0

```

以上！まいど～

---