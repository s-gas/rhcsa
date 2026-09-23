# sudo

`sudo` stands for "superuser do" and allows a user to execute a command as `root`. It can also be configured to execute commands as other users.

A benefit of `sudo` is that it logs by default all executed commands in `/var/log/secure`.

In RHEL (7+), all members of `wheel` can use `sudo` to execute command as any user, including `root`.

To get interactive `sudo` access:

```bash
[s-gas@localhost ~]$ sudo -i
[sudo] password for s-gas: 
[root@localhost ~]# 
```

## Configuration

Sudoers syntax:

```bash
who  where = (as_whom)  what
```

Example:

```bash
alice   ALL=(ALL:ALL) ALL
```

The main configuration file is `/etc/sudoers`, in which there is already this entry:

```bash
## Allows people in group wheel to run all commands
%wheel	ALL=(ALL)	ALL
```

This line allows members of the group `wheel` to execute any command from any host, as any user. Groups are preceded by `%`.

To add extra configuration, instead of modifying directly `/etc/sudoers`, it is recommended to create new files inside the drop-in directory `/etc/sudoers.d/`. For example I could create a file `/etc/sudoers.d/admin`:

```bash
user ALL=(admin) NOPASSWD: ALL
```

This will allow `user` to execute any command as `admin` without asking for the password.

To make sure that there are no syntax errors, you can use `visudo` as editor, which will validate the file.
