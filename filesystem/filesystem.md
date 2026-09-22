# File System

## Hierarchy

| location | files  |
|----------|----------|
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

Files are identified by their *inode* (index node), which has information about:
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
ls -li <filename>
```

Inodes are file system specific. For example, the `xfs` file system (default for RHEL) generates inodes of 512 bytes.

## Links

### Hard links

Hard links are filenames that point to the same *inode*.

This means that when a file gets created, it starts with a single hard link.

A directory, on the other hand, starts with at least 2 hard links: `..` and `.`.

You can view how many hard links a file has with `ls -l`, the second column is the number of hard links.

You can create a hard link with the `ln` command:

```bash
ln <file> <link>
```

These two files will have the same inode. You can check that with `ls -li`.

#### Limitations of hard links

- They can be used only for regular files
- They can be used only on the same file system.

### Symbolic links (Soft link)

A symbolic link is a file that points to an existing file or directory.

You can create a symlink with `ln -s`:

```bash
ln -s <file> <symlink>
```

The two files will not have the same inode.

A symbolic link is not a regular file, this is confirmed by the fact that `ls -l` would display somethink like:

```bash
lrwxr-xr-x  1 user  group  4 Sep 22 15:37 link -> file
```

#### Limitations of soft links

If the file that a soft link points to gets deleted, the symlink becomes a *dangling symbolic link*. In case a new file gets created with the same inode as the deleted file, the symlink would point to this new file.
