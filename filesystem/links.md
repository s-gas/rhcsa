# Links

## Hard links

Hard links are filenames that point to the same *inode*.

This means that when a file gets created, it starts with a single hard link.

A directory, on the other hand, starts with at least 2 hard links: `..` and `.`.

You can view how many hard links a file has with `ls -l`, the second column is the number of hard links.

You can create a hard link with the `ln` command:

```bash
[user@localhost ~]$ ln file.txt link
```

These two files will have the same inode. You can check that with `ls -li`:

```bash
[user@localhost ~]$ ls -li
total 0
25515864 -rw-r--r--. 2 user user 0 Sep 22 21:42 file.txt
25515864 -rw-r--r--. 2 user user 0 Sep 22 21:42 link
```

### Limitations of hard links

- They can be used only for regular files
- They can be used only on the same file system.

## Symbolic links (Soft link)

A symbolic link is a file that points to an existing file or directory.

You can create a symlink with `ln -s`:

```bash
[user@localhost ~]$ ln -s file.txt symlink
```

The two files will not have the same inode.

A symbolic link is not a regular file, this is confirmed by the fact that `ls -l` would display somethink like:

```bash
[user@localhost ~]$ ls -li
total 0
25515864 -rw-r--r--. 2 user user 0 Sep 22 21:42 file.txt
25515864 -rw-r--r--. 2 user user 0 Sep 22 21:42 link
25540504 lrwxrwxrwx. 1 user user 8 Sep 22 21:44 symlink -> file.txt
```

### Limitations of soft links

If the file that a soft link points to gets deleted, the symlink becomes a *dangling symbolic link*. In case a new file gets created with the same inode as the deleted file, the symlink would point to this new file.
