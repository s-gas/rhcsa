# Archives

An **archive** is a single regular file or a device file that contains multiple files. Archive files can be created with or without using compression to reduce the file size.

The command for managing archives is `tar`.

## Create an archive

To create an archive, use `tar -cf`:

```bash
[user@rocky ~]$ tar -cf files.tar file{1..3}.txt
[user@rocky ~]$ file files.tar 
files.tar: POSIX tar archive (GNU)
```

You can also create an archive directly from a directory:

```bash
[root@rocky ~]$ tar -cf etc.tar /etc
tar: Removing leading `/' from member names
```

In this case an absolute path was given and `tar` removes the forward slash (`/`) so that when extracting you don't end up overwriting the original directory or file.

## List files in an archive

To list the files contained in an archive, use `tar -tf`:

```bash
[user@rocky ~]$ tar -tf files.tar 
file1.txt
file2.txt
file3.txt
```

## Extract an archive

To extract the files from an archive in the current directory, use `tar -xf`:

```bash
[user@rocky ~]$ ls
file1.txt  file2.txt  file3.txt  files.tar
[user@rocky ~]$ rm file?.txt
[user@rocky ~]$ ls
files.tar
[user@rocky ~]$ tar -xf files.tar 
[user@rocky ~]$ ls
file1.txt  file2.txt  file3.txt  files.tar
```

This will not preserve the original permissions, it will apply the current `umask`:

```bash
[user@rocky ~]$ touch file.txt
[user@rocky ~]$ chmod 777 file.txt
[user@rocky ~]$ tar -cf file.tar file.txt 
[user@rocky ~]$ tar -xf file.tar
[user@rocky ~]$ ls -l file.txt 
-rwx------. 1 user user 0 Oct 17 22:23 file.txt
[user@rocky ~]$ umask
0077
```

To preserve the original permissions, use `tar -xpf`:

```bash
[user@rocky ~]$ tar -xpf file.tar
[user@rocky ~]$ ls -l file.txt 
-rwxrwxrwx. 1 user user 0 Oct 17 22:23 file.txt
```

## Create a compressed archive

The `tar` command supports different compression methods, the main ones are:
- `gzip`: earlier, fast, widely available
- `bzip2`: smaller archive size, less available
- `xz`: newer, slower, best compression ratio

To create a compressed archive with `gzip` compression, use `tar -czf`:

```bash
[user@rocky ~]$ tar -czf file.tar.gz file.txt 
```

To create a compressed archive with `bzip2` compression, use `tar -cjf`:

```bash
[user@rocky ~]$ tar -cjf file.tar.bz2 file.txt 
```

To create a compressed archive with `xz` compression, use `tar -cJf`:

```bash
[user@rocky ~]$ tar -cJf file.tar.xz file.txt 
```

Let's compress the same directory with all 3 algorithms to see the size difference:

```bash
[root@rocky ~]$ ls -lh etc* | awk '{print $5 " " $9}'
22M etc.tar
3.9M etc.tar.bz2
4.7M etc.tar.gz
2.9M etc.tar.xz
```

You can also list the content of compressed archive files:

```bash
[user@rocky ~]$ tar -tf file.tar.xz 
file.txt
```

## Extract compressed archives

The `tar` command recognizes the algorithm used for compression, so it does not require extra flags for extracting compressed archives. As for extracting regular archives, use `tar -xf`:

```bash
[user@rocky ~]$ tar -xf file.tar.xz 
```

## Compress and uncompress files

In case you have a file and not an archive, you can use directly `gzip`, `bzip2`, and `xz`.

With `gzip` you compress and with `gzip -d` you decompress:

```bash
[user@rocky ~]$ openssl rand -base64 1000 > file.txt
[user@rocky ~]$ ls -lh file.txt 
-rw-------. 1 user user 1.4K Oct 18 02:10 file.txt
[user@rocky ~]$ gzip file.txt 
[user@rocky ~]$ ls -lh file.txt.gz 
-rw-------. 1 user user 1.1K Oct 18 02:10 file.txt.gz
[user@rocky ~]$ gzip -d file.txt.gz 
[user@rocky ~]$ ls -lh file.txt 
-rw-------. 1 user user 1.4K Oct 18 02:10 file.txt
```
