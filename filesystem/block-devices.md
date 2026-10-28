# Block Devices

Block devices are files that provide access to storage devices. The `/dev` directory stores these block device files, which are created automatically by the kernel when it detects a device.

```bash
[root@rocky ~]$ ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 Oct  9 18:02 /dev/sda
```

Block devices are identified with `b`.

Block devices are exposed by the kernel as an array of fixed-size blocks (sectors), which are usually 512 bytes or 4KiB, that can be read or written.

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

The MBR partitioning scheme is the standard on systems that run ***BIOS** firmware.

Now it's legacy because of its 32-bit partition size, which allows up to 2 TiB disk and partition size. 

It also supports a maximum of 4 primary partitions. To overcome this limit, MBR has the `extended` type of partition, through which you can create up to 15 partitions.

### GPT

THE GPT partitioning scheme is the standard on system that run ****UEFI** firmware.

It uses a 64-bit partition size and provides a GUID (globally unique identifier) to identify each disk and partition.

### View partitions

You can manage partitions that uses MBR or GPT partition scheme through the `parted` command.

To view the partitions of a block device, run `parted print`:

```bash
[root@localhost ~]# lsblk -p
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
/dev/sda                   8:0    0   20G  0 disk 
├─/dev/sda1                8:1    0  600M  0 part /boot/efi
├─/dev/sda2                8:2    0    2G  0 part /boot
└─/dev/sda3                8:3    0 17.4G  0 part 
  ├─/dev/mapper/rlm-root 253:0    0 15.4G  0 lvm  /
  └─/dev/mapper/rlm-swap 253:1    0    2G  0 lvm  [SWAP]
/dev/sdb                   8:16   0  200M  0 disk 
/dev/sr0                  11:0    1    9G  0 rom  
/dev/sr1                  11:1    1 1024M  0 rom  
[root@localhost ~]# parted /dev/sda print
Model: VBOX HARDDISK (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  630MB   629MB   fat32        EFI System Partition  boot, esp
 2      630MB   2778MB  2147MB  xfs                                bls_boot
 3      2778MB  21.5GB  18.7GB                                     lvm
```

Running `parted` without any subcommand will open an interactive session:

```bash
[root@localhost ~]# parted /dev/sda
GNU Parted 3.6
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: VBOX HARDDISK (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  630MB   629MB   fat32        EFI System Partition  boot, esp
 2      630MB   2778MB  2147MB  xfs                                bls_boot
 3      2778MB  21.5GB  18.7GB                                     lvm
```

By default `parted` displays sizes in decimal format (KB, MB, GB). You can change the unit size by passing the `unit` parameter, which can be:
- `s`: sector
- `B`: byte
- `MiB`, `GiB`, `TiB`: binary format
- `MB`, `GB`, `TB`: decimal format

```bash
[root@localhost ~]# parted /dev/sda unit 'GiB' print
Model: VBOX HARDDISK (scsi)
Disk /dev/sda: 20.0GiB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start    End      Size     File system  Name                  Flags
 1      0.00GiB  0.59GiB  0.59GiB  fat32        EFI System Partition  boot, esp
 2      0.59GiB  2.59GiB  2.00GiB  xfs                                bls_boot
 3      2.59GiB  20.0GiB  17.4GiB                                     lvm
```

### Create a MBR Partition

First specify the partition scheme by providing a disk label (`msdos`), with `parted mklabel`:

```bash
[root@localhost ~]# parted /dev/sdb help mklabel
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition table)

	LABEL-TYPE is one of: aix, amiga, bsd, dvh, gpt, mac, msdos, pc98, sun, atari, loop
[root@localhost ~]# parted /dev/sdb mklabel msdos
Information: You may need to update /etc/fstab.
```

Then create a partition with `parted mkpart`:

```bash
[root@localhost ~]# df -h /dev/sdb
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        766M     0  766M   0% /dev
[root@localhost ~]# parted /dev/sdb help mkpart
  mkpart PART-TYPE [FS-TYPE] START END     make a partition

	PART-TYPE is one of: primary, logical, extended
        FS-TYPE is one of: udf, btrfs, nilfs2, ext4, ext3, ext2, f2fs, fat32, fat16, hfsx, hfs+, hfs, jfs, swsusp, linux-swap(v1), linux-swap(v0), ntfs, reiserfs, hp-ufs, sun-ufs, xfs, apfs2, apfs1, asfs, amufs5,
        amufs4, amufs3, amufs2, amufs1, amufs0, amufs, affs7, affs6, affs5, affs4, affs3, affs2, affs1, affs0, linux-swap, linux-swap(new), linux-swap(old)
        START and END are disk locations, such as 4GB or 10%.  Negative values count from the end of the disk.  For example, -1s specifies exactly the last sector.
        
        'mkpart' makes a partition without creating a new file system on the partition.  FS-TYPE may be specified to set an appropriate partition ID.
[root@localhost ~]# parted /dev/sdb mkpart primary xfs 2048s 200MB
Information: You may need to update /etc/fstab.

[root@localhost ~]# lsblk                                                 
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sdb            8:16   0  200M  0 disk 
└─sdb1         8:17   0  190M  0 part 
sr0           11:0    1    9G  0 rom  
sr1           11:1    1 1024M  0 rom  
```

The `parted mkpart` requires 4 arguments:
- partition type (primary, logical, extended)
- file system type (optional)
- start (`2048s` leaves enough sectors for MBR and any boot code that may be needed)
- end

### Create a GPT Partitions

First, label the block device for the GPT scheme (`gpt`) with `parted mklabel`:

```bash
[root@localhost ~]# parted /dev/sdb mklabel gpt
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? y
Information: You may need to update /etc/fstab.

```

The `/dev/sdb` block device already had a label, relabeling removes the previous label and makes all its data inaccessible.

Then create a partition with `parted mkpart`:

```bash
[root@localhost ~]# parted /dev/sdb mkpart primary xfs 2048s 100MB
Information: You may need to update /etc/fstab.

[root@localhost ~]# lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0  200M  0 disk 
└─sdb1   8:17   0   94M  0 part 
```

For GPT partitions, the first argument of `parted mkpart` is not the partition type but the name that you want to give to that partition!

### Delete a partition

To delete a partition, run `parted rm`:

```bash
[root@localhost ~]# parted /dev/sdb print                                 
Model: VBOX HARDDISK (scsi)
Disk /dev/sdb: 210MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  99.6MB  98.6MB               primary

[root@localhost ~]# parted /dev/sdb rm 1
Information: You may need to update /etc/fstab.

[root@localhost ~]# lsblk /dev/sdb                                        
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb    8:16   0  200M  0 disk 
```

## Logical Volumes

Storage devices can be organized also in logical volumes through LVM (Logical Volume Management). Block devices get aggregated into a volume group and its disk space is separated into logical volumes, which is functionally equivalent to partitioning.
LVM creates a directory in `/dev` matching the name of the volume group, which contains symbolic links to the logical volumes.

The advantage of using logical volumes is that they can be easily resized.
