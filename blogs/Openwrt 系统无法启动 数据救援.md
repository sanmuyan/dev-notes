# Openwrt 系统无法启动 数据救援

## 分区情况

`Openwrt` 文件系统挂载流程如下（已经简化，实际挂载过程比这个复杂）

| 挂载点      | 用途       | 挂载命令                                                                                                            |
| -------- | -------- | --------------------------------------------------------------------------------------------------------------- |
| /row     | 只读固件     | mount /dev/sda2 /row                                                                                            |
| /overlay | 用户数据     | losetup -o 96010240 /dev/loop0 /dev/sdb2 && mount /dev/loop0 /overlay                                           |
| /        | 合并后的文件系统 | mount -o noatime,lowerdir=/,upperdir=/overlay/upper,workdir=/overlay/upper -t overlay "overlayfs:/overlay" /mnt |


```shell
fdisk -l
Disk /dev/loop0: 420.44 MiB, 440860672 bytes, 861056 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
GPT PMBR size mismatch (1083423 != 1083454) will be corrected by write.
The backup GPT table is not on the end of the device.


Disk /dev/sda: 529.03 MiB, 554728960 bytes, 1083455 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C72F6ABB-6C0E-0D72-CF77-FDBE4292E600

Device      Start     End Sectors  Size Type
/dev/sda1    2048   34815   32768   16M Linux filesystem
/dev/sda2   34816 1083391 1048576  512M Linux filesystem
/dev/sda128    34    2047    2014 1007K BIOS boot

Partition table entries are not in disk order.


Disk /dev/zram0: 314 MiB, 329252864 bytes, 80384 sectors
Units: sectors of 1 * 4096 = 4096 bytes
Sector size (logical/physical): 4096 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

```shell
df -T
Filesystem           Type       1K-blocks      Used Available Use% Mounted on
/dev/root            squashfs       93952     93952         0 100% /rom
tmpfs                tmpfs         482312       376    481936   0% /tmp
/dev/loop0           f2fs          428480     56576    371904  13% /overlay
overlayfs:/overlay   overlay       428480     56576    371904  13% /
/dev/sda1            vfat           16334      8654      7680  53% /boot
/dev/sda1            vfat           16334      8654      7680  53% /boot
tmpfs                tmpfs            512         0       512   0% /dev
```

## 救援

把系统损坏的硬盘挂载到能正常运行的 `Openwrt` 系统上。镜像要和保持一致不然分区偏移量可能不一样

```shell
# 创建一个 loop 设备
opkg update
opkg install losetup
losetup -a
/dev/loop0: [0018]:14 (/sda2), offset 96010240
# 偏移量要保持一致
losetup -o 96010240 /dev/loop99 /dev/sdb2

# 挂载 overlay
mkdir /mnt/overlay
mount /dev/loop99 /mnt/overlay
```

所有变更过的文件都在 `/overlay/upper/` 里面