# `rsyslog` service

The `rsyslog` service treats the syslog files according to the facility and the priority in the way that it specified in the configuration file `/etc/rsyslog.conf`:

```bash
[root@rocky ~]$ cat /etc/rsyslog.conf | grep -A28 '#### RULES ####'
#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.* action(type="omfile" file="/dev/console")

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none action(type="omfile" file="/var/log/messages")

# The authpriv file has restricted access.
authpriv.* action(type="omfile" file="/var/log/secure")

# Log all the mail messages in one place.
mail.* action(type="omfile" file="/var/log/maillog" sync="on")

# Log cron stuff
cron.* action(type="omfile" file="/var/log/cron")

# Everybody gets emergency messages
*.emerg action(type="omusrmsg" users="*")

# Save news errors of level crit and higher in a special file.
uucp,news.crit action(type="omfile" file="/var/log/spooler")

# Save boot messages also to boot.log
local7.* action(type="omfile" file="/var/log/boot.log")
```

The configuration shows the rule through which the logs get categorized, for example `cron.*`, which are cron related logs with any priority are saved in `/var/log/cron`.

A rule in the configuration that specifies the priority would store all the logs with that priority or higher in the specified location!

## Log rotation

The `logrotate` command rotates log files by renaming the log file with the date of the rotation as an extension:

```bash
[root@rocky /]$ ls /var/log/ | grep 'messages'
messages
messages-20261009
```

From the example you can see that the last time that the `messages` log have been rotated is the 09-10-2026, creating `messages-20261009`.

Most log files rotate weekly, a scheduled jobs runs `logrotate` daily to see if any logs need rotation.

The configuration file for logrotate is `/etc/logrotate.conf`:

```bash
[root@rocky /]$ cat /etc/logrotate.conf  | grep -A1 '# rotate log files weekly'
# rotate log files weekly
weekly
```

## Syslog entry

```bash
[root@rocky /]$ tail -1 /var/log/messages
Oct 15 04:09:48 rocky systemd[1]: fwupd.service: Deactivated successfully.
```

The entry can be divided in 4 parts:
- `Oct 15 04:09:48`: date and time
- `rocky`: hostname that sent the log message
- `systemd[1]`: service and PID that sent the log message
- `fwupd.service: Deactivated successfully`: message

## Observing logs

`tail -f` keeps outputting new logs as they appear in the specified file:

```bash
[root@rocky ~]$ tail -f /var/log/secure
Oct 15 04:37:42 rocky sshd-session[3374]: Accepted publickey for user from 10.0.2.2 port 64333 ssh2: ED25519 SHA256:qm4570fHteIKnYegN7WbD9EBNQGFxJceoJmkR0ISo8c
Oct 15 04:37:42 rocky (systemd)[3380]: pam_unix(systemd-user:session): session opened for user user(uid=1001) by user(uid=0)
Oct 15 04:37:42 rocky sshd-session[3374]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
Oct 15 04:37:44 rocky sshd-session[3391]: Received disconnect from 10.0.2.2 port 64333:11: disconnected by user
Oct 15 04:37:44 rocky sshd-session[3391]: Disconnected from user user 10.0.2.2 port 64333
Oct 15 04:37:44 rocky sshd-session[3374]: pam_unix(sshd:session): session closed for user user
Oct 15 04:37:53 rocky sshd-session[3423]: Accepted publickey for user from 10.0.2.2 port 64347 ssh2: ED25519 SHA256:qm4570fHteIKnYegN7WbD9EBNQGFxJceoJmkR0ISo8c
Oct 15 04:37:53 rocky sshd-session[3423]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
Oct 15 04:45:56 rocky sudo[3484]:    user : TTY=pts/1 ; PWD=/root ; USER=root ; COMMAND=/bin/bash
Oct 15 04:45:56 rocky sudo[3484]: pam_unix(sudo-i:session): session opened for user root(uid=0) by user(uid=1001)
```

## Send syslog manually

With `logger` you can send logs messages manually to `rsyslog`. By default it send logs with facility-priority `user.notice`. To modify that use `logger -p`:

```bash
[root@rocky ~]$ logger -p 'local0.info' "this is a info log from local0"
[root@rocky ~]$ tail -1 /var/log/messages
Oct 15 05:01:18 rocky root[3546]: this is a info log from local0
```
