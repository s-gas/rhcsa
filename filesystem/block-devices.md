# Block Devices

Block devices are files that provide access to storage devices. The `/dev` directory stores these block device files, which are created automatically by the kernel when it detects a device.

```bash
[root@rocky ~]$ ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 Oct  9 18:02 /dev/sda
```

Block devices are identified with `b`.

## Disk Partitions

Storage devices can be divided into partitions for better organization.

You can list block devices with `lsblk`, which displays also the partitioning:

```bash
[root@rocky ~]$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sr0           11:0    1 1024M  0 rom  
```

In the example, `sda1`, `sda2`, `sda3` are partitions of `sda`. Partitions are block devices as well:

```bash
[root@rocky ~]$ ls -l /dev/sda1
brw-rw----. 1 root disk 8, 1 Oct  9 21:49 /dev/sda1
```

### MBR

The MBR partitioning scheme is the standard on systems that run BIOS firmware.

Now it's legacy because of its 32-bit partition size, which allows up to 2 TiB disk and partition size.

### GPT

THE GPT partitioning scheme is the standard on system that run UEFI firmware.

It uses a 64-bit partition size and provides a GUID (globally unique identifier) to identify each disk and partition.


## Logical Volumes

Storage devices can be organized also in logical volumes through LVM (Logical Volume Management). Block devices get aggregated into a volume group and its disk space is separated into logical volumes, which is functionally equivalent to partitioning.
LVM creates a directory in `/dev` matching the name of the volume group, which contains symbolic links to the logical volumes.

The advantage of using logical volumes is that they can be easily resized.
