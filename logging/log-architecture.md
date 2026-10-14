# System Log Architecture

The kernel and other processes record a log of events while they are running. These logs are useful to audit and to troubleshoot the system.

`systemd` provides the `systemd-journald` service which collects the logs from:
- kernel
- output from the early stages of the boot process
- `stdout` and `stderr` of daemons
- syslog events

After collecting the logs from these different sources, `systemd-journald` restructures the logs in a standard format and writes them into a journal, which is stored in a file system that does not persist reboot (`/run/log/journal`).

There is also another service, `rsyslog` that reads the logs restructured by `systemd-journald` and records them in its log files or forwards them to other services. The log files of `rsyslog` are stored in `/var/log/` and they do persist reboot.

`rsyslog` organizes in `/var/log` its logs in different files:
- `/var/log/messages`: miscellaneous logs
- `/var/log/secure`: security and authentication logs
- `/var/log/maillog`: mail server logs
- `/var/log/cron`: `cron` logs
- `/var/log/boot.log`: boot process logs

This organization is defined in `/etc/rsyslog.conf`.

Some applications, like Apache Web Server or Nginx, don't use `rsyslog` and write directly into `/var/log`.
