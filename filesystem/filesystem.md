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

## Files

Files are identified by their **inode** (index node), which has information about:
- file type
- ownership
- permissions
- timestamps
- file size
- number of hard links
- pointers to the data blocks where the content lives

Inodes do not keep track of the name of the file.

To view the inode of a file:

```bash
[user@localhost ~]$ ls -li file.txt 
25515864 -rw-r--r--. 2 user user 0 Sep 22 21:42 file.txt
```

Inodes are file system specific. For example, the `xfs` file system (default for RHEL) generates inodes of 512 bytes.
