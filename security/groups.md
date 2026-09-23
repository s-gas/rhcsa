# Groups

A group is a collection of users that need to share access to files and other system resources.

Every group has a `groupname` and it is identified by its **group ID** (GID).

Information about users is stored in `/etc/group`:

```bash
[user@localhost ~]$ cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
s-gas:x:1000:
docker:x:995:
user:x:1001:
```

Every entry consist of 4 fields separated by a colon:
- `groupname`.
- `x`: placeholder.
- `GID`: group ID.
- list of users that are part of the group as supplementary group.

## Configuration file

The file `/etc/login.defs` includes also configuration for groups, such as `GID` settings:

```bash
[user@localhost ~]$ cat /etc/login.defs | grep 'GID'
GID_MIN          1000
GID_MAX          60000
SYS_GID_MIN      201
SYS_GID_MAX      999
SUB_GID_MIN		   524288
SUB_GID_MAX		   600100000
SUB_GID_COUNT		 65536
```

## Primary group vs supplementary group

Every user has only one primary group, which owns the file that the user creates.

When a user is created, it is assigned a primary group with the same name as the user.

A user can also be part of supplementary groups. Users are granted access to files based on whether any of their groups have access, regardless of whether the groups are primary or supplementary.

This means that a user with primary group `user` can access a file owned by the group `wheel` if it has `wheel` as supplementary group.

## Create a group

To create a group, use the `groupadd` command:

```bash
[root@localhost etc]# groupadd admins
```

## Modify a group

To modify a group, use the `groupmod` command. Common usecases are:

- modify the name of a group:

```bash
[root@localhost etc]# groupmod -n new_group_name admins
```

- modify the GID of a group:

```bash
[root@localhost etc]# groupmod -g 1111 new_group_name
```

## Delete a group

To delete a group, use the `groupdel` command:

```bash
[root@localhost etc]# groupdel new_group_name
```

You cannot delete a group if it is the primary group of a user!

## Temporary change primary group

Use the `newgrp` command to switch the primary group of the current user to a supplementary group that it belongs to. This effect lasts only for the current shell session.

```bash
[user@localhost ~]$ id
uid=1001(user) gid=1001(user) groups=1001(user),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[user@localhost ~]$ newgrp wheel
[user@localhost ~]$ id
uid=1001(user) gid=10(wheel) groups=10(wheel),1001(user) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[user@localhost ~]$ touch new_file
[user@localhost ~]$ ls -l new_file 
-rw-r--r--. 1 user wheel 0 Sep 23 05:16 new_file
```
