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

## Local file system

RHEL uses `XFS` as default file system. `ext4` is also supported.

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
