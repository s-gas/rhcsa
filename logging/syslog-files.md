# Syslog files

Syslog files are logging files that follow the syslog protocol.

They are categorized by:
- facility: the sub-system that generated the log
- priority: severity of the log

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
