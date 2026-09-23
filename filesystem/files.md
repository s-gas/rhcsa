# Files

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


## File types

### Regular files

```bash
[user@localhost ~]$ ls -l file
-rw-r--r--. 1 user user 6 Sep 22 21:51 file
```

### Directories

```bash
[user@localhost ~]$ ls -ld dir
drwxr-xr-x. 2 user user 6 Sep 23 06:45 dir
```

### Symlinks

```bash
[user@localhost ~]$ ls -l link 
lrwxrwxrwx. 1 user user 4 Sep 23 06:45 link -> file
```
