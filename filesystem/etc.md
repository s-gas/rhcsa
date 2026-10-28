# `/etc`: useful files and directories

- `/etc/profile` and `/etc/profile.d/`

  Variables available to all users

- `/etc/sudoers` and `/etc/sudoers.d/`

  Configuration for `sudo` access

- `/etc/passwd`

  Information about users

- `/etc/group`

  Information about groups

- `/etc/shadow`

  Information about passwords

- `/etc/login.defs`

  Configuration for users and groups

- `/etc/ssh/`

  Configuration for SSH

- `/etc/hosts`

  Manual configuration for hostnames

- `/etc/hostname`

  Configuration for the machine's static hostname

- `/etc/NetworkManager/system-connection`

  Network configuration files

- `/etc/nsswitch.conf`

  Name Service Switch configuration file (where and in what order to look up admin and network databases)

- `/etc/resolv.conf`

  Configuration for DNS server

- `/etc/yum.repos.d/`

  Configuration for package repositories (`.repo` files)

- `/etc/crontab` and `/etc/cron.d/`

  Configuration for cron jobs

- `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, `/etc/cron.monthly/`

  Directories contains scripts to be executed in that interval

- `/etc/anacrontab`

  Configuration for anacron

- `/etc/systemd/system/`

  Directory containing custom configuration for `systemd` units. It has priority over `/usr/lib/systemd/system/`.

- `/etc/tmpfiles.d/`

  Drop-in directory containing custom configuration for `systemd-tmpfiles`. It has priority over `/usr/lib/tmpfiles.d/`

- `/etc/rsyslog.conf` and `/etc/rsyslog.d/`

  Configuration for the `rsyslog` service

- `/etc/logrotate.conf` and `/etc/logrotate.d/`

  Configuration for `logrotate`

- `/etc/systemd/journald.conf` and `/etc/systemd/journald.conf.d/`

  Custom configuration for `systemd-journald`. It has priority over `/usr/lib/systemd/journald.conf`

- `/etc/chrony.conf`

  Configuration for the `chronyd` service

- `/etc/tuned/tuned-main.conf`

  Main configuration file for the `tuned` daemon

- `/etc/selinux/`

  Configuration directory for SELinux

- `/etc/fstab`

  File containing the file systems to mount at boot time
