# Users

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

## Switch user account

```bash
[user@localhost ~]$ su - user2
Password:
[user2@localhost ~]$
```

If the username is omitted, it switches to `root`:

```bash
[user@localhost ~]$ su -
Password:
[root@localhost ~]$
```

`su` can be run without the dash (`-`), in which case it will start a **non-login shell** as opposed to a login shell.
If you need the user's environment variables, always use `su -`.

## Superuser

The superuser is `root` and it overrides the normal privileges of the file system.

> It is recommended to always login as a regular user and to escalated to `root` only when necessary. This can be achieved with `sudo` instead of switching to `root`, which could be disabled.

## Configuration for user accounts

The file `/etc/login.defs` sets some default options for user accounts, such as the range of valid UID numbers and default password aging rules. The values in this file affect only newly created user accounts. A change to this file does not affect existing users:

```bash
[root@localhost etc]$ cat /etc/login.defs | grep ^[^#]
MAIL_DIR	        /var/spool/mail
UMASK             022
HOME_MODE         0700
PASS_MAX_DAYS     99999
PASS_MIN_DAYS	    0
PASS_MIN_LEN	    8
PASS_WARN_AGE	    7
UID_MIN           1000
UID_MAX           60000
SYS_UID_MIN       201
SYS_UID_MAX       999
SUB_UID_MIN		    524288
SUB_UID_MAX		    600100000
SUB_UID_COUNT		  65536
GID_MIN           1000
GID_MAX           60000
SYS_GID_MIN       201
SYS_GID_MAX       999
SUB_GID_MIN		    524288
SUB_GID_MAX		    600100000
SUB_GID_COUNT		  65536
PASS_CHANGE_TRIES	5
PASS_ALWAYS_WARN	yes
ENCRYPT_METHOD YESCRYPT
USERGROUPS_ENAB yes
CREATE_HOME	yes
HMAC_CRYPTO_ALGO SHA512
```

## Create a user account

To create a new user:

```bash
[root@localhost etc]$ useradd user
```

This will create a new user named `user`, a home directory with the same name, a primary private group with the same name. The UID will be the first number available above 1000.

The user cannot login until a password is set with `passwd`:

```bash
[root@localhost etc]$ passwd user
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: password updated successfully
```

## Modify a user account:

You can modify an existing user with `usermod`. Some common examples are:

- change the user name:

```bash
[root@localhost etc]$ usermod -l new_name user
```

This will rename `user` to `new_name`. The user's home directory or mail spool should probably be renamed manually to reflect the new login name.

- add a supplementary group to a user:

```bash
[root@localhost etc]$ usermod -aG wheel user
```

This will add the group `wheel` to the supplementary group list of `user`.

- modify a user's primary group:

```bash
[root@localhost etc]$ usermod -g admin user
```

- lock a user's password:

```bash
[root@localhost etc]$ usermod -L user
```

This will add a `!` in front of the encrypted password, effectively disabling the password.

The same can be achieved with `passwd -L`:

```bash
[root@localhost etc]$ passwd -L user
```

- unlock a user's password:

```bash
[root@localhost etc]$ usermod -U user
```

## Delete a user account:

To delete a user:

```bash
[root@localhost etc]$ userdel user
```

This will not delete its home directory and the mail spool!

To delete a user and its home directory and its mail spool:

```bash
[root@localhost etc]$ userdel -r user
```

> When you remove a user without the `-r` option, an unassigned UID owns the files of the deleted user. If you create a new user and that user is assigned that UID, then the new account owns those files, which is a security risk.

Typically, organization security policies disallow deleting user accounts, and instead lock them from being used, to avoid this scenario:

```bash
[root@localhost etc]$ usermod -L user
[root@localhost etc]$ chage -E 0 user
[root@localhost etc]$ usermod -s /sbin/nologin user
```

The `chage -E 0 user` command sets the expiration date of that account to day 0 from `epoch` (1970-01-01).
The same can be achieved with `usermod -e 0 user`.

Setting the default shell to `/sbin/nologin` prevents interactive use of the system.

## Password

The cryptographically hashed passwords are stored in `/etc/shadow` file, which only the root user can read.

```bash
[root@localhost ~]$ ls -l /etc/shadow
----------. 1 root root 690 Sep 23 05:12 /etc/shadow
```

The entries of `/etc/shadow` contain 9 fields separated by colons:

```bash
[root@localhost ~]$ cat /etc/shadow
root:!::0:99999:7:::
user:$y$j9T$dFrzz1pP9zcpQWPSLph8S/$wP6kMnCeo.p6wcRm/cBm.FLDhb51W21JfpoNnJsJDq/:20719:0:99999:7:::
```

Fields:
- username
- hashed password; if locked it starts with `!`
- days from `epoch` (1970-01-01) when the password was last changed
- minimum number of days between password changes
- maximum number of days between password changes
- number of days of warning before the password expires
- number of days of inactivity (from password expiration) before the account is locked
- days from `epoch` when the account expires
- empty (for future use)

The same information can be retrieved with `chage -l`:

```bash
[root@localhost ~]$ chage -l user
Last password change			                        : Sep 23, 2026
Password expires					                        : never
Password inactive					                        : never
Account expires						                        : never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

### Cryptographically Hashed Password Format

The cryptographically hashed password field stores three pieces of information: the hashing algorithm in use, the salt, and the cryptographical hash. Salt adds random data to the cryptographical hash, for creating a unique hash to strengthen the cryptographically hashed password. Each piece of information is delimited by the dollar (`$`) character.

When a user tries to login, the system looks up the entry for the user in the `/etc/shadow` file, and combines the salt for the user with the plain text typed password. The system then cryptographically hashes the combination of the salt and plain text password with the specified hashing algorithm. If the result matches the cryptographical hash, then the user typed the right password.

### Password Aging for existing users

To modify the password aging policy of existing users, you can use `chage`. This will not affect newly created users!

Examples: 

- modify the account to expire 30 days from now:

```bash
[root@localhost ~]$ chage -E $(date -d +30days +%F) user
```

In this example, `date -d` is used to calculate the date 30 days from now.

- set the maximum number of days during which a password is valid to 10:

```bash
[root@localhost ~]$ chage -M 10 user
```

- verify the policy of a user:

```bash
[root@localhost ~]$ chage -l user
Last password change					                    : Sep 23, 2026
Password expires					                        : Oct 03, 2026
Password inactive				                         	: never
Account expires			                        			: Oct 22, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 10
Number of days of warning before password expires	: 7
```

- force the user to change their password at next login:

```bash
[root@localhost ~]$ chage -d 0 user
[root@localhost ~]$ chage -l user
Last password change                    					: password must be changed
Password expires                        					: password must be changed
Password inactive				                        	: password must be changed
Account expires						                        : Oct 22, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 10
Number of days of warning before password expires	: 7
```

### Password Aging for new users

To modify the policy for new users, modify `/etc/login.defs`.
