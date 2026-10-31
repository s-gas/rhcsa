# Block Devices

Block devices are files that provide access to storage devices. The `/dev` directory stores these block device files, which are created automatically by the kernel when it detects a device.

```bash
[root@rocky ~]$ ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 Oct  9 18:02 /dev/sda
```

Block devices are identified with `b`.

Block devices are exposed by the kernel as an array of fixed-size blocks (sectors), which are usually 512 bytes or 4KiB, that can be read or written.

To list the block devices, use `lsblk`:

```bash
[s-gas@localhost ~]$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sdb            8:16   0    2G  0 disk 
sdc            8:32   0    2G  0 disk 
sdd            8:48   0    2G  0 disk 
sr0           11:0    1 1024M  0 rom  
```
