# linux-lvm
## LVM概念

```
LVM
PV: physical volumes
	物理卷在逻辑卷管理系统最底层，可为整个物理硬盘或实际物理硬盘上的分区
PP: physical partions
	物理分区
VG: volumes group
	卷组建立在物理卷上，一个卷组中至少要包括一个物理卷，卷组建立后可动态的添加卷到卷	组	中，一个逻辑卷管理系统工程中可有多个卷组。
LV: logical volumes
	逻辑卷建立在卷组基础上，卷组中未分配空间可用于建立新的逻辑卷，逻辑卷建立后可以动	态扩展和缩小空间。
PE: physical extent
	物理区域是物理卷中可用于分配的最小存储单元，物理区域大小在建立卷组时指定，一旦确	定不能更改，同一卷组所有物理卷的物理区域大小需一致，新的pv加入到vg后，pe的大小	自动更改为vg中定义的pe大小。
LE: logical extent
	逻辑区域是逻辑卷中可用于分配的最小存储单元，逻辑区域的大小取决于逻辑卷所在卷组中的物理区域的大小。
```
## 示例
```
物理磁盘
/dev/sdb 
/dev/sdc
/dev/sdd 

> lvmdiskscan
  /dev/sdb  [       1.82 TiB]
  /dev/sdc  [       1.82 TiB]
  /dev/sdd  [       1.82 TiB]
  3 disks
```

### 1. 物理分区
```
命令
fdisk

示例
> fdisk /dev/sdb
  Welcome to fdisk (util-linux 2.23.2).
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.
  Command (m for help): m
  Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-3907029167, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-3907029167, default 3907029167):
Using default value 3907029167
Partition 1 of type Linux and of size 1.8 TiB is set

Command (m for help): w


> lvmdiskscan
  /dev/sdb1 [       1.82 TiB]
  /dev/sdc1 [       1.82 TiB]
  /dev/sdd1 [       1.82 TiB]
```

### 2. 创建物理卷
```
> pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
  
> pvdisplay
  "/dev/sdb1" is a new physical volume of "1.82 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               1.82 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               33qGjr-NVE6-4yVP-vFeX-HjDX-Od2t-hlnO9w

  "/dev/sdc1" is a new physical volume of "1.82 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc1
  VG Name
  PV Size               1.82 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               poCgho-q0wb-j25m-iOge-Cix3-yKn0-XCWEp4

  "/dev/sdd1" is a new physical volume of "1.82 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdd1
  VG Name
  PV Size               1.82 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               N2jF2K-g0KE-ECgz-xGWA-kt1n-ZKAH-zFU9VH
```

### 3. 创建卷组
```
> vgcreate vg1 /dev/sdb1
  Volume group "vg1" successfully created
> vgextend vg1 /dev/sdc1
  Volume group "vg1" successfully extended
> vgextend vg1 /dev/sdd1
  Volume group "vg1" successfully extended

以上可以一个命令完成
> vgcreate vg1 /dev/sdb1 /dev/sdc1 /dev/sdd1

  
> vgdisplay
    --- Volume group ---
  VG Name               vg1
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               5.46 TiB
  PE Size               4.00 MiB
  Total PE              1430793
  Alloc PE / Size       0 / 0
  Free  PE / Size       1430793 / 5.46 TiB
  VG UUID               fUTv80-7QMh-jJMm-uOcR-02vF-esoz-ipeN0P
```

### 4. 创建逻辑卷
```
> lvcreate -L 1024G vg1 -n lv1
  Logical volume "lv1" created.
  
> lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                ZL68fH-eQG4-K80C-yOQm-bo07-XDXb-kMj2nl
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-22 11:00:47 +0800
  LV Status              available
  # open                 0
  LV Size                1.00 TiB
  Current LE             262144
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
  
> ls /dev/vg1/
  lv1

> ls /dev/mapper/
  control  vg1-lv1
  
现在可通过/dev/mapper/vg1-lv1访问逻辑卷lv1，你可以在逻辑卷上创建文件系统并像普通分区一样挂载它了。
```

### 5. 创建文件系统并挂载逻辑卷
```
> mkfs.ext4 /dev/mapper/vg1-lv1

> mount /dev/mapper/vg1-lv1 /data

```
