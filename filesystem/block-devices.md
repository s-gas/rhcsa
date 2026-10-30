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
[root@localhost ~]$ lsblk -p
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
[root@localhost ~]$ parted /dev/sda print
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
[root@localhost ~]$ parted /dev/sda
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
[root@localhost ~]$ parted /dev/sda unit 'GiB' print
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
[root@localhost ~]$ parted /dev/sdb help mklabel
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition table)

	LABEL-TYPE is one of: aix, amiga, bsd, dvh, gpt, mac, msdos, pc98, sun, atari, loop
[root@localhost ~]$ parted /dev/sdb mklabel msdos
Information: You may need to update /etc/fstab.
```

Then create a partition with `parted mkpart`:

```bash
[root@localhost ~]$ df -h /dev/sdb
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        766M     0  766M   0% /dev
[root@localhost ~]$ parted /dev/sdb help mkpart
  mkpart PART-TYPE [FS-TYPE] START END     make a partition

	PART-TYPE is one of: primary, logical, extended
        FS-TYPE is one of: udf, btrfs, nilfs2, ext4, ext3, ext2, f2fs, fat32, fat16, hfsx, hfs+, hfs, jfs, swsusp, linux-swap(v1), linux-swap(v0), ntfs, reiserfs, hp-ufs, sun-ufs, xfs, apfs2, apfs1, asfs, amufs5,
        amufs4, amufs3, amufs2, amufs1, amufs0, amufs, affs7, affs6, affs5, affs4, affs3, affs2, affs1, affs0, linux-swap, linux-swap(new), linux-swap(old)
        START and END are disk locations, such as 4GB or 10%.  Negative values count from the end of the disk.  For example, -1s specifies exactly the last sector.
        
        'mkpart' makes a partition without creating a new file system on the partition.  FS-TYPE may be specified to set an appropriate partition ID.
[root@localhost ~]$ parted /dev/sdb mkpart primary xfs 2048s 200MB
Information: You may need to update /etc/fstab.

[root@localhost ~]$ lsblk                                                 
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
[root@localhost ~]$ parted /dev/sdb mklabel gpt
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? y
Information: You may need to update /etc/fstab.

```

The `/dev/sdb` block device already had a label, relabeling removes the previous label and makes all its data inaccessible.

Then create a partition with `parted mkpart`:

```bash
[root@localhost ~]$ parted /dev/sdb mkpart primary xfs 2048s 100MB
Information: You may need to update /etc/fstab.

[root@localhost ~]$ lsblk /dev/sdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0  200M  0 disk 
└─sdb1   8:17   0   94M  0 part 
```

For GPT partitions, the first argument of `parted mkpart` is not the partition type but the name that you want to give to that partition!

### Delete a partition

To delete a partition, run `parted rm`:

```bash
[root@localhost ~]$ parted /dev/sdb print                                 
Model: VBOX HARDDISK (scsi)
Disk /dev/sdb: 210MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  99.6MB  98.6MB               primary

[root@localhost ~]$ parted /dev/sdb rm 1
Information: You may need to update /etc/fstab.

[root@localhost ~]$ lsblk /dev/sdb                                        
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb    8:16   0  200M  0 disk 
```

## LVM (Logical Volumes Management)

Storage devices can be organized also in logical volumes through **LVM** (Logical Volume Management). Block devices (**physical volumes**) get aggregated into a **volume group** and its disk space is separated into **logical volumes**, which is functionally equivalent to partitioning.

LVM creates a directory in `/dev` matching the name of the volume group, which contains symbolic links to the logical volumes.

The advantage of using logical volumes is that they can be easily resized. This saves you from the pain of, in case you end up with a full partition or disk, restoring the data in a bigger partition or disk. With logical volumes you can simply assign more disk space to a logical volumes from the volume group that it was created. In case that also the volume group has used all its disk space, you can extend the volume group with an extra physical volume.

### Create physical volumes

You can label block devices as LVM physical volumes with `pvcreate`:

```bash
[root@localhost ~]$ lsblk -p
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
/dev/sda                   8:0    0   20G  0 disk 
├─/dev/sda1                8:1    0  600M  0 part /boot/efi
├─/dev/sda2                8:2    0    2G  0 part /boot
└─/dev/sda3                8:3    0 17.4G  0 part 
  ├─/dev/mapper/rlm-root 253:0    0 15.4G  0 lvm  /
  └─/dev/mapper/rlm-swap 253:1    0    2G  0 lvm  [SWAP]
/dev/sdb                   8:16   0    2G  0 disk 
/dev/sdc                   8:32   0    2G  0 disk 
/dev/sdd                   8:48   0    2G  0 disk 
/dev/sr0                  11:0    1 1024M  0 rom  

[root@localhost ~]$ pvcreate /dev/sd{b,c}
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
```

You can use `pvs` to show the physical volumes:

```bash
[root@localhost ~]$ pvs
  PV         VG   Fmt  Attr PSize  PFree 
  /dev/sda3  rlm  lvm2 a--  17.41g     0 
  /dev/sdb   vg01 lvm2 a--  <2.00g <2.00g
  /dev/sdc   vg01 lvm2 a--  <2.00g <2.00g
```

### Create a volume group

From the physical volumes, you can create a volume group with `vgcreate`:

```bash
[root@localhost ~]$ pvs
  PV         VG   Fmt  Attr PSize  PFree 
  /dev/sda3  rlm  lvm2 a--  17.41g     0 
  /dev/sdb   vg01 lvm2 a--  <2.00g <2.00g
  /dev/sdc   vg01 lvm2 a--  <2.00g <2.00g
[root@localhost ~]$ vgcreate vg01 /dev/sdb /dev/sdc
  Volume group "vg01" successfully created
[root@localhost ~]$ vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  rlm    1   2   0 wz--n- 17.41g    0 
  vg01   2   0   0 wz--n-  3.99g 3.99g
```

`vgs` displays the volume groups.

### Create logical volumes

You can create a logical volume with `lvcreate`, through which you can specify its name (`-n`), its size in binary format (`-L`) and the volume group to create the logical volume from:

```bash
[root@localhost ~]$ vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  rlm    1   2   0 wz--n- 17.41g    0 
  vg01   2   0   0 wz--n-  3.99g 3.99g
[root@localhost ~]$ lvcreate -n lv01 -L 2GB vg01
  Logical volume "lv01" created.
[root@localhost ~]$ lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rlm  -wi-ao---- 15.41g                                                    
  swap rlm  -wi-ao----  2.00g                                                    
  lv01 vg01 -wi-a-----  2.00g                                                    
[root@localhost ~]$ vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  rlm    1   2   0 wz--n- 17.41g    0 
  vg01   2   1   0 wz--n-  3.99g 1.99g
```

### Create a filesystem in a logical volume

To create a filesystem into a logical volume you need to specify the block device in one of the following ways:
- `/dev/<vg_name>/<lv_name>`
- `/dev/mapper/<vg_name>-<lv_name>`

Looking at the block devices in `/dev/` you can see that both are soft links to the same file:

```bash
[root@localhost dev]$ ls -l /dev/vg01/lv01
lrwxrwxrwx. 1 root root 7 Oct 30 13:05 /dev/vg01/lv01 -> ../dm-2
[root@localhost dev]$ ls -l /dev/mapper/vg01-lv01
lrwxrwxrwx. 1 root root 7 Oct 30 13:05 /dev/mapper/vg01-lv01 -> ../dm-2
```

You can run `mkfs.xfs` (for XFS file system) with one of these two options:

```bash
[root@localhost dev]$ mkfs.xfs /dev/mapper/vg01-lv01
meta-data=/dev/mapper/vg01-lv01  isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
[root@localhost dev]$ lsblk 
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sdb            8:16   0    2G  0 disk 
└─vg01-lv01  253:2    0    2G  0 lvm  
sdc            8:32   0    2G  0 disk 
└─vg01-lv01  253:2    0    2G  0 lvm  
sdd            8:48   0    2G  0 disk 
sr0           11:0    1 1024M  0 rom  
```

After you created the filesystem you can mount it by adding the entry to `/etc/fstab`.

### Display LVM informations

`pvs`, `vgs`, and `lvs` do not display the entire information. 

To display all the information of a physical volume, run `pvdisplay`:

```bash
[root@localhost ~]$ pvs
  PV         VG   Fmt  Attr PSize  PFree
  /dev/sda3  rlm  lvm2 a--  17.41g    0 
  /dev/sdb   vg01 lvm2 a--  <2.00g    0 
  /dev/sdc   vg01 lvm2 a--  <2.00g 1.99g
[root@localhost ~]$ pvdisplay /dev/sdb
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               vg01
  PV Size               2.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              511
  Free PE               0
  Allocated PE          511
  PV UUID               SWkpEc-MLdz-7n9J-5EId-ZXTk-VdMZ-uFDBDc
```

To display information about a volume group, run `vgdisplay`.

To display information about a logical volume, run `lvdisplay`.

### Extend a volume group

You can extend a volume group by adding a physical volume to an existing volume group with `vgextend`:

```bash
[root@localhost ~]$ pvs
  PV         VG   Fmt  Attr PSize  PFree
  /dev/sda3  rlm  lvm2 a--  17.41g    0 
  /dev/sdb   vg01 lvm2 a--  <2.00g    0 
  /dev/sdc   vg01 lvm2 a--  <2.00g 1.99g
  /dev/sdd        lvm2 ---   2.00g 2.00g
[root@localhost ~]$ vgextend vg01 /dev/sdd
  Volume group "vg01" successfully extended
[root@localhost ~]$ vgdisplay vg01
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <5.99 GiB
  PE Size               4.00 MiB
  Total PE              1533
  Alloc PE / Size       512 / 2.00 GiB
  Free  PE / Size       1021 / <3.99 GiB
  VG UUID               fSRNrE-QAaF-XUZE-TKRY-w33E-fC15-3JeeuR
```

### Extend a logical volume

You can increase a size of a logical volume without any downtime by running `lvextend` and specifying the size of the extension (`-L`) in binary format (bytes, mebibytes, gibibytes):

```bash
[root@localhost ~]$ lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rlm  -wi-ao---- 15.41g                                                    
  swap rlm  -wi-ao----  2.00g                                                    
  lv01 vg01 -wi-ao----  2.00g                                                    
[root@localhost ~]$ vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  rlm    1   2   0 wz--n- 17.41g     0 
  vg01   3   1   0 wz--n- <5.99g <3.99g
[root@localhost ~]$ lvextend -L +1G /dev/mapper/vg01-lv01
  Size of logical volume vg01/lv01 changed from 2.00 GiB (512 extents) to 3.00 GiB (768 extents).
  Logical volume vg01/lv01 successfully resized.
[root@localhost ~]$ lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rlm  -wi-ao---- 15.41g                                                    
  swap rlm  -wi-ao----  2.00g                                                    
  lv01 vg01 -wi-ao----  3.00g                  
```

Extending a logical volume does NOT extend the filesystem within it:

```bash
[root@localhost ~]$ lvs /dev/mapper/vg01-lv01
  LV   VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01 vg01 -wi-ao---- 3.00g  
[root@localhost ~]$ df -h /extra
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/vg01-lv01  2.0G   71M  1.9G   4% /extra
```

To resize the filesystem together with the logical volume, you have to run `lvextend -r`:

```bash
[root@localhost ~]$ lvextend -r -L 3G /dev/mapper/vg01-lv01
  New size (768 extents) matches existing size (768 extents).
  File system xfs found on vg01/lv01 mounted at /extra.
  Size of logical volume vg01/lv01 unchanged from 3.00 GiB (768 extents).
  Extending file system xfs to 3.00 GiB (3221225472 bytes) on vg01/lv01...
xfs_growfs /dev/vg01/lv01
meta-data=/dev/mapper/vg01-lv01  isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
data blocks changed from 524288 to 786432
xfs_growfs done
  Extended file system xfs on vg01/lv01.
  Logical volume vg01/lv01 successfully resized.
[root@localhost ~]$ df -h /extra
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/vg01-lv01  3.0G   91M  2.9G   3% /extra
```

To extend logical volumes that are used a **swap space**, remember first to turn off the swap area with `swapoff`, then resize the logical volume. After resizing you will need to flag it again as swap space with `mkswap` and finally you can turn it on with `swapon`.

### Remove LVM Storage

To remove LVM the process is the opposite, first unmount with `umount` the filesystem and remove its entry from `/etc/fstab`:

```bash
[root@localhost ~]$ umount /extra
[root@localhost ~]$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sdb            8:16   0    2G  0 disk 
└─vg01-lv01  253:2    0    3G  0 lvm  
sdc            8:32   0    2G  0 disk 
└─vg01-lv01  253:2    0    3G  0 lvm  
sdd            8:48   0    2G  0 disk 
sr0           11:0    1 1024M  0 rom  
```

Then you can remove the logical volume(s) with `lvremove`:

```bash
[root@localhost ~]$ lvremove /dev/mapper/vg01-lv01
Do you really want to remove active logical volume vg01/lv01? [y/n]: y
  Logical volume "lv01" successfully removed.
[root@localhost ~]$ lvs
  LV   VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rlm -wi-ao---- 15.41g                                                    
  swap rlm -wi-ao----  2.00g      
```

To remove the volume group, run `vgremove`:

```bash
[root@localhost ~]$ vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  rlm    1   2   0 wz--n- 17.41g     0 
  vg01   3   0   0 wz--n- <5.99g <5.99g
[root@localhost ~]$ vgremove vg01
  Volume group "vg01" successfully removed
```

To remove the physical volumes, run `pvremove`:

```bash
[root@localhost ~]$ pvs
  PV         VG  Fmt  Attr PSize  PFree
  /dev/sda3  rlm lvm2 a--  17.41g    0 
  /dev/sdb       lvm2 ---   2.00g 2.00g
  /dev/sdc       lvm2 ---   2.00g 2.00g
  /dev/sdd       lvm2 ---   2.00g 2.00g
[root@localhost ~]$ pvremove /dev/sd{b,c,d}
  Labels on physical volume "/dev/sdb" successfully wiped.
  Labels on physical volume "/dev/sdc" successfully wiped.
  Labels on physical volume "/dev/sdd" successfully wiped.
[root@localhost ~]$ pvs
  PV         VG  Fmt  Attr PSize  PFree
  /dev/sda3  rlm lvm2 a--  17.41g    0 
```
