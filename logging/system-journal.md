# System journal

`systemd-journald` stores all the logs in a structured way in a binary file called `journal`. These files are in `/run/log` which by default does not persist reboot.

```bash
[root@rocky /]$ file /run/log/journal/c66ffaecff7e43a3a1c23ff480c4027d/system.journal 
/run/log/journal/c66ffaecff7e43a3a1c23ff480c4027d/system.journal: Journal file, Wed Oct 14 17:39:09 2026, online, keyed hash siphash24, compressed zstd, compact, header size 0x110, entries 0x751
```

To view the log messages in the journal use `journalctl`, which without any flags or arguments will list all the logs:

```bash
[root@rocky /]$ journalctl | tail -5
Oct 15 05:23:48 rocky systemd[1]: fwupd.service: Deactivated successfully.
Oct 15 05:26:57 rocky systemd[1]: Starting dnf-makecache.service - dnf makecache...
Oct 15 05:26:57 rocky dnf[3597]: Metadata cache refreshed recently.
Oct 15 05:26:57 rocky systemd[1]: dnf-makecache.service: Deactivated successfully.
Oct 15 05:26:57 rocky systemd[1]: Finished dnf-makecache.service - dnf makecache.
```

Like with `less`, you can use `journalctl -n` to directly specify how many entries to print:

```bash
[root@rocky /]$ journalctl -n 5
Oct 15 05:23:48 rocky systemd[1]: fwupd.service: Deactivated successfully.
Oct 15 05:26:57 rocky systemd[1]: Starting dnf-makecache.service - dnf makecache...
Oct 15 05:26:57 rocky dnf[3597]: Metadata cache refreshed recently.
Oct 15 05:26:57 rocky systemd[1]: dnf-makecache.service: Deactivated successfully.
Oct 15 05:26:57 rocky systemd[1]: Finished dnf-makecache.service - dnf makecache.
```

Again, like `less`, you can also use `journalctl -f` to keep outputting new journal logs.

You can also filter the logs by priority with `journal -p`. It will display all logs with the priority specified or higher:

```bash
[root@rocky /]$ journalctl -p crit -n 3
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: nft_compat_module_init
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: ip_set
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: ip_set_init
```

For specifying the logs of a service use `journalctl -u`:

```bash
[root@rocky /]$ journalctl -u sshd.service -n 3 -p info
Oct 15 04:37:42 rocky sshd-session[3374]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
Oct 15 04:37:53 rocky sshd-session[3423]: Accepted publickey for user from 10.0.2.2 port 64347 ssh2: ED25519 SHA256:qm4570fHteIKnYegN7WbD9EBNQGFxJceoJmkR0ISo8c
Oct 15 04:37:53 rocky sshd-session[3423]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
```

Another useful feature is filtering entries by date and time with `journalctl --since --until`:

```bash
[root@rocky /]$ journalctl --since "2026-10-14 19:38" --until "2026-10-14 19:40" -p err
Oct 14 19:39:10 rocky kernel: Warning: Unmaintained driver is detected: e1000
Oct 14 19:39:10 rocky kernel: Warning: Unmaintained driver is detected: e1000_init_module
Oct 14 19:39:24 rocky firewalld[724]: ERROR: NAME_CONFLICT: new_policy_object(): 'docker-forwarding'
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: nft_compat
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: nft_compat_module_init
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: ip_set
Oct 14 19:39:24 rocky kernel: Warning: Unmaintained driver is detected: ip_set_init
```

You can also view additional information about logs, with `journalctl -o verbose`:

```bash
[root@rocky /]$ journalctl -o verbose -n 1
Thu 2026-10-15 06:19:01.650702 CEST [s=ed4fab7673424847818fc3850fa01b5f;i=758;b=6c701e7f03df46039adae38e393b4b3d;m=8f05df13c;t=65dd958efa55a;x=4b45d8286e8c6ef1]
    PRIORITY=6
    _BOOT_ID=6c701e7f03df46039adae38e393b4b3d
    _MACHINE_ID=c66ffaecff7e43a3a1c23ff480c4027d
    _HOSTNAME=rocky
    SYSLOG_FACILITY=3
    _SYSTEMD_SLICE=system.slice
    _RUNTIME_SCOPE=system
    _TRANSPORT=syslog
    SYSLOG_IDENTIFIER=chronyd
    SYSLOG_PID=722
    _PID=722
    _COMM=chronyd
    _EXE=/usr/sbin/chronyd
    _CMDLINE=/usr/sbin/chronyd -n -F 2
    _SELINUX_CONTEXT=system_u:system_r:chronyd_t:s0
    _SYSTEMD_CGROUP=/system.slice/chronyd.service
    _SYSTEMD_UNIT=chronyd.service
    _SYSTEMD_INVOCATION_ID=9278cb19bffb4aecbf6866a3337ce1ef
    _UID=997
    _GID=997
    _CAP_EFFECTIVE=2000400
    SYSLOG_TIMESTAMP=Oct 15 06:19:01 
    MESSAGE=Source 131.188.3.222 replaced with 2a13:7840:ba5e::2 (2.rocky.pool.ntp.org)
    _SOURCE_REALTIME_TIMESTAMP=1792037941650702
```

Knowing all these fields, you can also filter logs that have that specific value, for example `journalctl MESSAGE=Source 131.188.3.222 replaced with 2a13:7840:ba5e::2 (2.rocky.pool.ntp.org)`:

```bash
[root@rocky /]$ journalctl MESSAGE="Source 131.188.3.222 replaced with 2a13:7840:ba5e::2 (2.rocky.pool.ntp.org)"
Oct 15 06:19:01 rocky chronyd[722]: Source 131.188.3.222 replaced with 2a13:7840:ba5e::2 (2.rocky.pool.ntp.org)
```

## Configure persistent system journals

In order to store journals persistently, you need first to create `/var/log/journal`:

```bash
[root@rocky /]$ mkdir /var/log/journal
```

Then you need to change the configuration of `systemd-journald`. The default configuration is in `/usr/lib/systemd/journald.conf`:

```bash
[root@rocky /]$ cat /usr/lib/systemd/journald.conf
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it under the
#  terms of the GNU Lesser General Public License as published by the Free
#  Software Foundation; either version 2.1 of the License, or (at your option)
#  any later version.
#
# Entries in this file show the compile time defaults. Local configuration
# should be created by either modifying this file (or a copy of it placed in
# /etc/ if the original file is shipped in /usr/), or by creating "drop-ins" in
# the /etc/systemd/journald.conf.d/ directory. The latter is generally
# recommended. Defaults can be restored by simply deleting the main
# configuration file and all drop-ins located in /etc/.
#
# Use 'systemd-analyze cat-config systemd/journald.conf' to display the full config.
#
# See journald.conf(5) for details.

[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitIntervalSec=30s
#RateLimitBurst=10000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#SystemMaxFiles=100
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=0
#MaxFileSec=1month
#ForwardToSyslog=no
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
#MaxLevelSocket=debug
#LineMax=48K
#ReadKMsg=yes
Audit=
```

To update the configuration, create `/etc/systemd/journald.conf` and set `Storage=persistent`, so that it persists across reboots:

```bash
[root@rocky /]$ cat /etc/systemd/journald.conf
[Journal]
Storage=persistent
[root@rocky /]$ systemctl reload-or-restart systemd-journald
```

Here you might need to `reboot` in case `/var/log/journal` does not get populated. After reboot:

```bash
[user@rocky ~]$ ls /var/log/journal/
c66ffaecff7e43a3a1c23ff480c4027d
```

> Technically changing the `Storage` value to `persistent` wasn't necessary, because when set to `auto`, which is default, it checks if the directory `/var/log/journal/` exists. If yes, `systemd-journald` writes its logs there, otherwise it stores them in the volatile directory.

To limit the output to a specific boot you can use `journalctl -b` with the number of the specific boot:

```bash
[user@rocky ~]$ journalctl -b -1 -n 5
Oct 15 22:06:24 rocky kernel: audit: type=1131 audit(1792094784.862:220): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=systemd-reboot comm="systemd" exe="/usr/lib/systemd/>
Oct 15 22:06:25 rocky systemd-shutdown[1]: Syncing filesystems and block devices.
Oct 15 22:06:25 rocky systemd-shutdown[1]: Sending SIGTERM to remaining processes...
Oct 15 22:06:25 rocky systemd-journald[577]: Received SIGTERM from PID 1 (systemd-shutdow).
Oct 15 22:06:25 rocky systemd-journald[577]: Journal stopped
```

To view information about the boots, run `journalctl --list-boots`:

```bash
[user@rocky ~]$ journalctl --list-boots
IDX BOOT ID                          FIRST ENTRY                  LAST ENTRY                  
 -1 c284dd65cccb488ba87b992defbba696 Thu 2026-10-15 22:00:50 CEST Thu 2026-10-15 22:06:25 CEST
  0 15640176e2d14b05a94df4442abd058e Thu 2026-10-15 22:06:32 CEST Thu 2026-10-15 22:08:04 CEST
```
