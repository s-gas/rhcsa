# Users and Groups

## Users

Every user has a `username` and it is identified by its **user ID** (UID).

There are three types of user:
- superuser: `root`, it has UID 0 and it has full system access.
- system users: users used by **daemons**. 
- regular users.

You can use the `id` command to display information about a user:

```bash
[user@localhost ~]$ id
uid=1001(user) gid=1001(user) groups=1001(user) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Every file has a user owner.

With `ls -l` you can see the user owner of a file:

```bash
[user@localhost ~]$ ls -l file.txt 
-rw-r--r--. 1 user user 0 Sep 22 21:42 file.txt
```

The third column of the output is the owner of the file.

Every process runs as a particular user.

You can run `ps -au` to list all the processes with a terminal and display the user associated with that process:

```bash
[user@localhost ~]$ ps -au
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         810  0.0  0.1 229272  2192 tty1     Ss+  21:39   0:00 /sbin/agetty -o -- \u --noreset --noclear - linux
root        2020  0.0  0.2 228556  3824 pts/0    Ss+  21:39   0:00 -bash
root        2172  0.0  0.2 233400  4524 pts/1    S    21:41   0:00 su - user
user        2176  0.0  0.2 228672  3780 pts/1    S    21:41   0:00 -bash
user        2215  0.0  0.2 230760  3676 pts/1    R+   21:43   0:00 ps -au
```

Information about users is stored in `/etc/passwd`:

```bash
[user@localhost ~]$ cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
adm:x:3:4:adm:/var/adm:/usr/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/usr/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/usr/sbin/nologin
operator:x:11:0:operator:/root:/usr/sbin/nologin
games:x:12:100:games:/usr/games:/usr/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/usr/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
systemd-oom:x:999:999:systemd Userspace OOM Killer:/:/sbin/nologin
dbus:x:81:81:System Message Bus:/:/usr/sbin/nologin
sssd:x:998:998:User for sssd:/run/sssd/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:997:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:996:996:systemd Core Dumper:/:/usr/sbin/nologin
s-gas:x:1000:1000:Simone Gasparini:/home/s-gas:/bin/bash
user:x:1001:1001::/home/user:/bin/bash
```

Every entry consist of 7 fields separated by a colon:
- `username`.
- `x`: placeholder. The encrypted password used to be stored here.
- `UID`: user ID.
- `GID`: group ID of the user's primary group.
- comment about the user.
- home directory.
- default shell program.

## Groups

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
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:s-gas
cdrom:x:11:
mail:x:12:
man:x:15:
dialout:x:18:
floppy:x:19:
games:x:20:
tape:x:33:
video:x:39:
ftp:x:50:
lock:x:54:
audio:x:63:
users:x:100:
clock:x:103:
nobody:x:65534:
tss:x:59:
utmp:x:22:
utempter:x:35:
systemd-oom:x:999:
input:x:104:
kvm:x:36:
render:x:105:
sgx:x:106:
systemd-journal:x:190:
dbus:x:81:
sssd:x:998:
sshd:x:74:
chrony:x:997:
systemd-coredump:x:996:
s-gas:x:1000:
docker:x:995:
user:x:1001:
```

Every entry consist of 4 fields separated by a colon:
- `groupname`.
- `x`: placeholder.
- `GID`: group ID.
- list of users that are part of the group as supplementary group.

### Primary group vs supplementary group

Every user has only one primary group, which owns the file that the user creates.

When a user is created, it is assigned a primary group with the same name as the user.

A user can also be part of supplementary groups. Users are granted access to files based on whether any of their groups have access, regardless of whether the groups are primary or supplementary.

This means that a user with primary group `user` can access a file owned by the group `wheel` if it has `wheel` as supplementary group.
