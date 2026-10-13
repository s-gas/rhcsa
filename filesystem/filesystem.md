# File System

Files on a Linux system are organized into a single inverted tree of directories.

## Hierarchy

| location | files  |
|----------|----------|
| `/`      | root of the hierarchy |
| `/boot`  | boot process |
| `/dev`   | devices |
| `/etc`   | system configuration |
| `/home`  | home directories |
| `/root`  | `root` home directory |
| `/run`   | runtime data |
| `/tmp`   | temporary data |
| `/usr`   | installed software, shared libraries |
| `/var`   | system persistent dynamic data |
| `/mnt`   | temporary mountpoints |

## `/etc`: useful files and directories

- `/etc/profile` and `/etc/profile.d/`

  Variables available to all users

- `/etc/sudoers` and `/etc/sudoers.d/`

  Configuration for `sudo` access

- `/etc/passwd`

  Information about users

- `/etc/group`

  Information about groups

- `/etc/shadow`

  Information about passwords

- `/etc/login.defs`

  Configuration for users and groups

- `/etc/ssh/`

  Configuration for SSH

- `/etc/hosts`

  Manual configuration for hostnames

- `/etc/hostname`

  Configuration for the machine's static hostname

- `/etc/NetworkManager/system-connection`

  Network configuration files

- `/etc/nsswitch.conf`

  Name Service Switch configuration file (where and in what order to look up admin and network databases)

- `/etc/resolv.conf`

  Configuration for DNS server

- `/etc/yum.repos.d/`

  Configuration for package repositories (`.repo` files)

- `/etc/crontab` and `/etc/cron.d/`

  Configuration for cron jobs

- `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, `/etc/cron.monthly/`

  Directories contains scripts to be executed in that interval

- `/etc/anacrontab`

  Configuration for anacron

- `/etc/systemd/system/`

  Directory containing custom configuration for `systemd` units. It has priority over `/usr/lib/systemd/system/`

  

## Local file system

RHEL uses `XFS` as default file system. `ext4` is also supported.

Run `lsblk -f` to view the filesystem types in the block devices:

```bash
[root@rocky /]$ lsblk -f
NAME         FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                           
├─sda1       vfat        FAT32          B2AD-34AB                               585.6M     2% /boot/efi
├─sda2       xfs                        0d89c457-3fc9-4f6f-b542-8bd6f01999c5      1.6G    17% /boot
└─sda3       LVM2_member LVM2 001       at8gx2-OiRm-wkor-UJrW-s1tQ-nYr9-CrdaSs                
  ├─rlm-root xfs                        8e42d026-99ee-4d95-a3e7-f766cf0f2ca5     12.5G    19% /
  └─rlm-swap swap        1              5208dcad-d81b-41bd-b542-7cbd5add5281                  [SWAP]
sr0 
```


## Examine File System

To display the file-system devices and their mount point, use `df`:

```bash
[root@rocky ~]$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/rlm-root   16G  2.9G   13G  19% /
devtmpfs              766M     0  766M   0% /dev
tmpfs                 785M     0  785M   0% /dev/shm
efivarfs              256K   11K  246K   4% /sys/firmware/efi/efivars
tmpfs                 314M  6.6M  308M   3% /run
/dev/sda2             2.0G  336M  1.7G  17% /boot
/dev/sda1             599M   14M  586M   3% /boot/efi
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 157M  4.0K  157M   1% /run/user/1001
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
```

Use `du` to view disk usage of a specific directory:

```bash
[user@rocky /]$ sudo du -sh / 2>/dev/null
2.6G	/
```

Here the `-s` flag display the summary instead of displaying recursively the size of each file.

## Mount points

A mount point is an empty directory to which a filesystem is mounted. When the directory is mounted, it's content can be viewed and accessed from the system.

On a server, you might need to `mount` manually a storage device to access its filesystem. If you mount manually, remember to reboot the system:

```bash
[root@rocky /]$ mount /dev/sda2 /mnt/data
[root@rocky /]$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /mnt/data
│                                     /boot
└─sda3         8:3    0 17.4G  0 part 
├─rlm-root 253:0    0 15.4G  0 lvm  /
└─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sr0           11:0    1 1024M  0 rom  
```

Now the filesystem in `/dev/sda2` is mounted both in `/boot` (default) and in `/mnt/data`.

If a mount point had content inside before the mount, the existing files are not accessible when a filesystem is mounted there.

You can also `mount` by using the UUID of the device:

```bash
[root@rocky /]$ lsblk -pf
NAME                     FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
/dev/sda                                                                                                  
├─/dev/sda1              vfat        FAT32          B2AD-34AB                               585.6M     2% /boot/efi
├─/dev/sda2              xfs                        0d89c457-3fc9-4f6f-b542-8bd6f01999c5      1.6G    17% /boot
└─/dev/sda3              LVM2_member LVM2 001       at8gx2-OiRm-wkor-UJrW-s1tQ-nYr9-CrdaSs                
  ├─/dev/mapper/rlm-root xfs                        8e42d026-99ee-4d95-a3e7-f766cf0f2ca5     12.5G    19% /
  └─/dev/mapper/rlm-swap swap        1              5208dcad-d81b-41bd-b542-7cbd5add5281                  [SWAP]
/dev/sr0                                                                                                  
[root@rocky mnt]$ mount -U 0d89c457-3fc9-4f6f-b542-8bd6f01999c5 /mnt/data
[root@rocky mnt]$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda            8:0    0   20G  0 disk 
├─sda1         8:1    0  600M  0 part /boot/efi
├─sda2         8:2    0    2G  0 part /mnt/data
│                                     /boot
└─sda3         8:3    0 17.4G  0 part 
  ├─rlm-root 253:0    0 15.4G  0 lvm  /
  └─rlm-swap 253:1    0    2G  0 lvm  [SWAP]
sr0           11:0    1 1024M  0 rom  
```

To unmount, run `umount`:

```bash
[root@rocky /]$ umount /dev/sda2 /mnt/data
umount: /mnt/data: not mounted.
```

Remember that you cannot unmount a filesystem while that filesystem is in use. You can use `lsof` to view the files currently open and the processes that are using them:

```bash
[root@rocky /]$ cd /mnt/data
[root@rocky data]$ lsof /mnt/data
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    12678 root  cwd    DIR    8,2     4096  128 /mnt/data
lsof    12861 root  cwd    DIR    8,2     4096  128 /mnt/data
lsof    12862 root  cwd    DIR    8,2     4096  128 /mnt/data
```
