# Swap space

A swap space is an area of a disk that is under the control of the Linux kernel memory management subsystem. It's used to supplement the RAM by storing inactive pages in memory. A system's **virtual memory** is obtained by combining the RAM with the swap space.

When a system runs out of memory, the kernel memory management looks in RAM for idle pages, which are written in swap area and the pages in RAM are reassigned to other processes. If a process requires a page stored in the swap area, the kernel locates another idle page in RAM and swaps it with the one required by the process.

Since the swap space is on disk, it is definitely slower than RAM.

## Swap space recommendations

| RAM | SWAP |
|-----|------|
| <= 2 GB | Twice the RAM |
| <= 8 GB | Same as RAM |
| > 8GB | At least 4GB |

## Create a swap space

Before creating a swap space you need first to create a partion and specifying the file system type to `linux-swap`:

```bash
[s-gas@localhost ~]$ parted /dev/sdb print
WARNING: You are not superuser.  Watch out for permissions.
Error: Error opening /dev/sdb: Permission denied
Retry/Cancel? ^C
[s-gas@localhost ~]$ sudo -i
[root@localhost ~]$ parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End  Size  File system  Name  Flags

[root@localhost ~]$ parted /dev/sdb mkpart primary linux-swap 2048s 1GB
Information: You may need to update /etc/fstab.

[root@localhost ~]$ parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  1000MB  999MB               primary  swap

```

Remember that with the GPT partition scheme, the first argument of the `parted mkpart` command is not the partition type but is the name that you give to that partition. In the example the partition is called `primary`.

Once you have a partition with `linux-swap` file system type, you can create a swap space by running `mkswap`:

```bash
[root@localhost ~]$ mkswap /dev/sdb1
Setting up swapspace version 1, size = 953 MiB (999288832 bytes)
no label, UUID=eaf1662b-aa32-4103-88a8-eec2ad012318
```

## Activate and deactivate swap space

To activate a swap space, run `swapon`:

```bash
[root@localhost ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.5Gi       284Mi       961Mi       8.3Mi       404Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@localhost ~]$ lsblk -f
NAME         FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                           
├─sda1       vfat        FAT32          B2AD-34AB                               585.6M     2% /boot/efi
├─sda2       xfs                        0d89c457-3fc9-4f6f-b542-8bd6f01999c5      1.7G    13% /boot
└─sda3       LVM2_member LVM2 001       at8gx2-OiRm-wkor-UJrW-s1tQ-nYr9-CrdaSs                
  ├─rlm-root xfs                        8e42d026-99ee-4d95-a3e7-f766cf0f2ca5     13.5G    12% /
  └─rlm-swap swap        1              5208dcad-d81b-41bd-b542-7cbd5add5281                  [SWAP]
sdb                                                                                           
└─sdb1       swap        1              eaf1662b-aa32-4103-88a8-eec2ad012318                  
sdc                                                                                           
└─sdc1       xfs                        76fd7521-b3a8-4ff1-904c-99d95d64673e      380M     8% /mnt/new
[root@localhost ~]$ swapon /dev/sdb1
[root@localhost ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.5Gi       283Mi       961Mi       8.3Mi       404Mi       1.3Gi
Swap:          2.9Gi          0B       2.9Gi

```

To deactivate, run `swapoff`:

```bash
[root@localhost ~]$ swapoff /dev/sdb1
[root@localhost ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.5Gi       283Mi       961Mi       8.3Mi       404Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
```

You can also use `swapon -s` to view the active swap spaces:

```bash
[s-gas@localhost ~]$ swapon -s
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	2097148		0		-2
```

## Activate swap space persistently

To add a swap space persistently, add an entry to `/etc/fstab`:

```bash
[root@localhost ~]$ lsblk -f /dev/sdb1
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb1 swap   1           eaf1662b-aa32-4103-88a8-eec2ad012318                
[root@localhost ~]$ grep 'swap' /etc/fstab
UUID=5208dcad-d81b-41bd-b542-7cbd5add5281 none                    swap    defaults        0 0
UUID=eaf1662b-aa32-4103-88a8-eec2ad012318 swap			  swap    defaults        0 0
```

The entry for swap space is the same as for other file systems, the difference is that instead of the mount point you can either specify `swap` or `none`.

You can also specify the priority of the swap area. A swap area with higher priority will get used before one with lower priority. By default the priority is set to `-2`. You can set the priority in the fourth column. Instead of `defaults` you can specify `pri=X`:

```bash
[root@localhost ~]$ grep 'swap' /etc/fstab 
UUID=5208dcad-d81b-41bd-b542-7cbd5add5281 none                    swap    pri=2        0 0
UUID=eaf1662b-aa32-4103-88a8-eec2ad012318 swap			  swap    defaults        0 0
```

To activate all the swap space entries in `/etc/fstab`, run `swapon -av`:

```bash
[s-gas@localhost ~]$ swapon -av
swapon: /dev/mapper/rlm-swap: already active -- ignored
```

If a swap area is already active and you modify the priority, you might need to deactivate it with `swapoff` and then reactivate it with `swapon -av`.
