# systemd

`systemd` is a **daemon** that manages the startup process of Linux, including the startup and the management of the processes.

A **daemon** is a process that either waits or runs in the background and, generally, starts automatically at startup and stops at shutdown. By convention, the names of the daemon processes end with `d`, like `systemd`.

A **service** is the `systemd` unit that manages one or more daemons.

In RHEL, the first process that starts is `systemd` and it has `PID 1`. This can be viewed with `pstree`:

```bash
[s-gas@localhost ~]$ pstree
systemd─┬─NetworkManager───3*[{NetworkManager}]
        ├─agetty
        ├─auditd───{auditd}
        ├─chronyd
        ├─containerd───7*[{containerd}]
        ├─crond
        ├─dbus-broker-lau───dbus-broker
        ├─dockerd───7*[{dockerd}]
        ├─firewalld───3*[{firewalld}]
        ├─python3───ttyd
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd─┬─sshd-session───sshd-session───bash───su───bash
        │      └─sshd-session───sshd-session───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        ├─systemd-userdbd───3*[systemd-userwor]
        └─tmux: server───bash
```

`systemd` provides the following features:
- parallelization: it starts multiple services simultaneously, improving boot speed
- on-demand starting of daemons without the need of other services
- service dependency management: this means that a service that requires another service will always start after that service
- tracking related processes together by using Linux control groups (`cgroups`)

## Units

A **unit** is the abstraction that `systemd` manages. Units are represented through **unit files**.

A unit has:
- a name
- a type

`crond.service` it's a unit with name `crond` and type `service`.

### Unit types

Some important unit types are:
- `service`: represent services
- `socket`: represent IPC (inter-process communication) sockets

## Interact with systemd

To interact with `systemd` you can use the `systemctl` command

### List units

Without any arguments, `systemctl` lists all the units that are loaded and active:

```bash
[s-gas@localhost etc]$ systemctl | head
  UNIT                                                                                        LOAD   ACTIVE     SUB       DESCRIPTION
  proc-sys-fs-binfmt_misc.automount                                                           loaded active     running   Arbitrary Executable File Formats File System Automount Point
  sys-devices-pci0000:00-0000:00:02.0-sound-card0-controlC0.device                            loaded active     plugged   /sys/devices/pci0000:00/0000:00:02.0/sound/card0/controlC0
  sys-devices-pci0000:00-0000:00:03.0-virtio0-host0-target0:0:0-0:0:0:0-block-sda-sda1.device loaded active     plugged   HARDDISK EFI\x20System\x20Partition
  sys-devices-pci0000:00-0000:00:03.0-virtio0-host0-target0:0:0-0:0:0:0-block-sda-sda2.device loaded active     plugged   HARDDISK 2
  sys-devices-pci0000:00-0000:00:03.0-virtio0-host0-target0:0:0-0:0:0:0-block-sda-sda3.device loaded active     plugged   HARDDISK 3
  sys-devices-pci0000:00-0000:00:03.0-virtio0-host0-target0:0:0-0:0:0:0-block-sda.device      loaded active     plugged   HARDDISK
  sys-devices-pci0000:00-0000:00:03.0-virtio0-host0-target0:0:1-0:0:1:0-block-sr0.device      loaded active     plugged   CD-ROM
  sys-devices-pci0000:00-0000:00:08.0-net-enp0s8.device                                       loaded active     plugged   82540EM Gigabit Ethernet Controller (PRO/1000 MT Desktop Adapter)
● sys-devices-pci0000:00-0000:00:08.0-net-eth0.device  
```

To list the units of a specific type you can use `systemctl list-units --type=`:

```bash
[s-gas@localhost etc]$ systemctl list-units --type=service | head
  UNIT                                                  LOAD   ACTIVE SUB     DESCRIPTION
  auditd.service                                        loaded active running Security Audit Logging Service
  chronyd.service                                       loaded active running NTP client/server
  containerd.service                                    loaded active running containerd container runtime
  crond.service                                         loaded active running Command Scheduler
  dbus-broker.service                                   loaded active running D-Bus System Message Bus
  docker.service                                        loaded active running Docker Application Container Engine
  dracut-shutdown.service                               loaded active exited  Restore /run/initramfs on shutdown
  firewalld.service                                     loaded active running firewalld - dynamic firewall daemon
  getty@tty1.service                                    loaded active running Getty on tty1
```

The previous command list all the units of type service that are currently active.

To list also inactive units, you need to add the `--all` flag:

```bash
[s-gas@localhost etc]$ systemctl list-units --type=service --all | head
  UNIT                                                  LOAD      ACTIVE   SUB     DESCRIPTION
● audit-rules.service                                   not-found inactive dead    audit-rules.service
  auditd.service                                        loaded    active   running Security Audit Logging Service
● autofs.service                                        not-found inactive dead    autofs.service
  chronyd.service                                       loaded    active   running NTP client/server
  containerd.service                                    loaded    active   running containerd container runtime
  crond.service                                         loaded    active   running Command Scheduler
  dbus-broker.service                                   loaded    active   running D-Bus System Message Bus
● display-manager.service                               not-found inactive dead    display-manager.service
  dm-event.service                                      loaded    inactive dead    Device-mapper event daemon
```

You can also filter by using `--state` with the value in the `LOAD`, `ACTIVE`, `SUB` field:

```bash
[s-gas@localhost etc]$ systemctl list-units --state=dead | head
  UNIT                                                                                 LOAD      ACTIVE   SUB  DESCRIPTION
● boot.automount                                                                       not-found inactive dead boot.automount
  dev-tpmrm0.device                                                                    loaded    inactive dead /dev/tpmrm0
● home.mount                                                                           not-found inactive dead home.mount
● sysroot.mount                                                                        not-found inactive dead sysroot.mount
  tmp.mount                                                                            loaded    inactive dead Temporary Directory /tmp
● audit-rules.service                                                                  not-found inactive dead audit-rules.service
● autofs.service                                                                       not-found inactive dead autofs.service
● display-manager.service                                                              not-found inactive dead display-manager.service
  dm-event.service                                                                     loaded    inactive dead Device-mapper event daemon
```

The command `systemctl list-units` lists only the units that `systemd` attempted to parse and load into memory. To view also units that are installed but not enabled use `systemctl list-unit-files`:

```bash
[s-gas@localhost etc]$ systemctl list-unit-files --type=service | head
UNIT FILE                                    STATE           PRESET
auditd.service                               enabled         enabled
autovt@.service                              alias           -
blk-availability.service                     disabled        disabled
bluetooth.service                            enabled         enabled
capsule@.service                             static          -
chrony-wait.service                          disabled        disabled
chronyd-restricted.service                   disabled        disabled
chronyd.service                              enabled         enabled
console-getty.service                        disabled        disabled
```

### View unit status

To view the status of a specific unit, use `systemctl status`:

```bash
[s-gas@localhost etc]$ systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-09-22 21:39:50 CEST; 2 days ago
 Invocation: c39517932a1a420fbc6fbb6d3d70869c
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 793 (sshd)
      Tasks: 1 (limit: 10659)
     Memory: 9M (peak: 26M)
        CPU: 644ms
     CGroup: /system.slice/sshd.service
             └─793 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Sep 23 07:31:30 localhost.localdomain sshd-session[4074]: Accepted password for s-gas from 10.0.2.2 port 59801 ssh2
Sep 23 07:31:30 localhost.localdomain sshd-session[4074]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 23 23:48:39 localhost.localdomain sshd-session[6307]: Accepted password for s-gas from 10.0.2.2 port 57351 ssh2
Sep 23 23:48:39 localhost.localdomain sshd-session[6307]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 24 02:18:05 localhost.localdomain sshd-session[6769]: Accepted password for s-gas from 10.0.2.2 port 59999 ssh2
Sep 24 02:18:06 localhost.localdomain sshd-session[6769]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 24 21:29:12 localhost.localdomain sshd-session[9251]: Accepted password for s-gas from 10.0.2.2 port 52886 ssh2
Sep 24 21:29:12 localhost.localdomain sshd-session[9251]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 25 05:31:36 localhost.localdomain sshd-session[10181]: Accepted password for s-gas from 10.0.2.2 port 58315 ssh2
Sep 25 05:31:36 localhost.localdomain sshd-session[10181]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
```

If the unit type is omitted, `systemd` tries to interpret the unit as a service unit:

```bash
[s-gas@localhost etc]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-09-22 21:39:50 CEST; 2 days ago
 Invocation: c39517932a1a420fbc6fbb6d3d70869c
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 793 (sshd)
      Tasks: 1 (limit: 10659)
     Memory: 9M (peak: 26M)
        CPU: 644ms
     CGroup: /system.slice/sshd.service
             └─793 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Sep 23 07:31:30 localhost.localdomain sshd-session[4074]: Accepted password for s-gas from 10.0.2.2 port 59801 ssh2
Sep 23 07:31:30 localhost.localdomain sshd-session[4074]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 23 23:48:39 localhost.localdomain sshd-session[6307]: Accepted password for s-gas from 10.0.2.2 port 57351 ssh2
Sep 23 23:48:39 localhost.localdomain sshd-session[6307]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 24 02:18:05 localhost.localdomain sshd-session[6769]: Accepted password for s-gas from 10.0.2.2 port 59999 ssh2
Sep 24 02:18:06 localhost.localdomain sshd-session[6769]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 24 21:29:12 localhost.localdomain sshd-session[9251]: Accepted password for s-gas from 10.0.2.2 port 52886 ssh2
Sep 24 21:29:12 localhost.localdomain sshd-session[9251]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
Sep 25 05:31:36 localhost.localdomain sshd-session[10181]: Accepted password for s-gas from 10.0.2.2 port 58315 ssh2
Sep 25 05:31:36 localhost.localdomain sshd-session[10181]: pam_unix(sshd:session): session opened for user s-gas(uid=1000) by s-gas(uid=0)
```

### Start and stop services

To start a service is `systemctl start`:

```bash
[root@localhost ~]$ systemctl start crond
[root@localhost ~]$ systemctl is-active crond
active
```

To stop a service is `systemctl stop`:

```bash
[root@localhost ~]$ systemctl stop crond
[root@localhost ~]$ systemctl is-active crond
inactive
```

### Enable and disable services

To enable a service means that it will be started at boot, to do so use `systemctl enable`:

```bash
[root@localhost ~]$ systemctl enable sshd
[root@localhost ~]$ systemctl is-enabled sshd
enabled
```

To enable and start a service at the same time use `systemctl enable --now`:

```bash
[root@localhost ~]$ systemctl enable --now sshd
[root@localhost ~]$ systemctl is-enabled sshd
enabled
[root@localhost ~]$ systemctl is-active sshd
active
```

To disable a service means that it will not start at boot, use `systemctl disable`:

```bash
[root@localhost ~]$ systemctl disable sshd
Removed '/etc/systemd/system/multi-user.target.wants/sshd.service'.
[root@localhost ~]$ systemctl is-enabled sshd
disabled
```

To disable and stop at the same time use `systemctl disable --now`:

```bash
[root@localhost ~]$ systemctl disable --now crond
Removed '/etc/systemd/system/multi-user.target.wants/crond.service'.
[root@localhost ~]$ systemctl is-enabled crond
disabled
[root@localhost ~]$ systemctl is-active crond
inactive
```

### Restart and reload services

To restart a service is `systemctl restart`. This changes the PID:

```bash
[root@localhost ~]$ systemctl status crond | grep 'Main PID'
   Main PID: 10620 (crond)
[root@localhost ~]$ systemctl restart crond
[root@localhost ~]$ systemctl status crond | grep 'Main PID'
   Main PID: 10632 (crond)
```

Some services can reload the configuration files without an actual restart, in which case you can use `systemctl reload`:

```bash
[root@localhost ~]$ systemctl reload crond
[root@localhost ~]$ systemctl status crond | grep 'Main PID'
   Main PID: 10632 (crond)
```

If you don't know if a service has the reload option, use `systemctl reload-or-restart`:

```bash
[root@localhost ~]$ systemctl status docker | grep 'Main PID'
   Main PID: 843 (dockerd)
[root@localhost ~]$ systemctl reload-or-restart docker
[root@localhost ~]$ systemctl status docker | grep 'Main PID'
   Main PID: 843 (dockerd)
```

### List unit dependencies

To view the units required for running a specific unit, use `systemctl list-dependencies`:

```bash
[root@localhost ~]$ systemctl list-dependencies docker.service | head
docker.service
● ├─containerd.service
● ├─docker.socket
● ├─system.slice
● ├─network-online.target
● │ └─NetworkManager-wait-online.service
● └─sysinit.target
●   ├─dev-hugepages.mount
●   ├─dev-mqueue.mount
●   ├─dracut-shutdown.service
```

To view the units that depend on a specific unit, use `systemctl list-dependencies --reverse`:

```bash
[root@localhost ~]$ systemctl list-dependencies --reverse sshd.service
sshd.service
● └─multi-user.target
○   └─graphical.target
```

### Mask units

To prevent two services to create conflicts, you could mask one service, which creates a link to `/dev/null` in the configuration directory of that service, which prevents it from starting. This is done through `systemctl mask`:

```bash
[root@localhost ~]$ systemctl mask crond
Created symlink '/etc/systemd/system/crond.service' → '/dev/null'.
[root@localhost ~]$ systemctl is-enabled crond
masked
```

To unmask, use `systemctl unmask`:

```bash
[root@localhost ~]$ systemctl unmask crond
Removed '/etc/systemd/system/crond.service'.
```
