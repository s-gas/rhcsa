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

The file type can determined with the command `file`:

```bash
[user@localhost ~]$ file file
file: empty
```

Another way is by using `ls -l`:

```bash
[user@localhost ~]$ ls -l file 
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
```

The first character of the long listing is the file type, and is interpreted as follows:

- `-` is a regular file
- `d` is a directory
- `l` is a symbolic link
- `c` is a character device file.
- `b` is a block device file.
- `p` is a named pipe file.
- `s` is a local socket file.

## File permissions

Files have a user owner and a group owner. With file permissions you can set different permissions for:
- owning user (`user` permissions)
- owning group (`group` permissions)
- users that are not the owning user and not part of the owning group (`other` permissions)

For all these categories, you can set up 3 permission types:

| Permission   | Effect on files | Effect on directories            |
|--------------|-----------------|----------------------------------|
| `r`(read)    | Can be read     | Can be listed                    |
| `w`(write)   | Can be modified | Files can be created and deleted |
| `x`(execute) | Can be executed | Can become the working directory |

```bash
[user@localhost ~]$ ls -l file
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
```

The file permissions are the 9 characters that follow the file type. In this example:
- the owning user (`user`) have read (`r`) and write (`w`) permissions
- the owning group (`user`) have read (`r`) permissions
- others have read (`r`) permissions

### How to change file permissions

To change file permissions, use `chmod`, which can be used with 2 methods:

- symbolic method:

```bash
[user@localhost ~]$ ls -l file 
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
[user@localhost ~]$ chmod g+w file
[user@localhost ~]$ ls -l file
-rw-rw-r--. 1 user user 0 Sep 23 06:45 file
```

- octal method:

```bash
[user@localhost ~]$ ls -l file
-rw-rw-r--. 1 user user 0 Sep 23 06:45 file
[user@localhost ~]$ chmod 644 file
[user@localhost ~]$ ls -l file
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
```

The permissions for each category (user, group, other) are represented as a set of 3 bits in octal.

`644` in binary is `110-100-100`, the bits that are on represent the allowed permissions.

### How to change file ownership

To change file ownership you can use the `chown` command.

Only `root` can change the user owner of a file. The group owner can be changed by `root` and the user owner of that file, but, in case is changed by a regular user, the group owner can be changed only to a group that the regular user is part of.

To change the user owner:

```bash
[root@localhost user]# ls -l file
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
[root@localhost user]# chown root file
[root@localhost user]# ls -l file
-rw-r--r--. 1 root user 0 Sep 23 06:45 file
```

To change the group owner:

```bash
[root@localhost user]# chown :root file
[root@localhost user]# ls -l file
-rw-r--r--. 1 root root 0 Sep 23 06:45 file
```

Alternatively, you can change the group owner also with `chgrp`:

```bash
[root@localhost user]# chgrp root file
[root@localhost user]# ls -l file
-rw-r--r--. 1 user root 0 Sep 23 06:45 file
```

To change both the user owner and the group owner:

```bash
[root@localhost user]# chown user:user file
[root@localhost user]# ls -l file
-rw-r--r--. 1 user user 0 Sep 23 06:45 file
```
