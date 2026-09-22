# Users and Groups

## User

Every user has a `username` and it is identified by its **user ID** (UID).

There are three types of user:
- superuser:  `root`, it has UID 0 and it has full system access.
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
s-gas       2079  0.0  0.2 228568  3796 pts/1    Ss   21:40   0:00 -bash
root        2172  0.0  0.2 233400  4524 pts/1    S    21:41   0:00 su - user
user        2176  0.0  0.2 228672  3780 pts/1    S    21:41   0:00 -bash
user        2215  0.0  0.2 230760  3676 pts/1    R+   21:43   0:00 ps -au
```
