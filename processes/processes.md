# Processes

A process is a running instance of a program.

It consists of the following items:

- address space (allocated memory)
- security properties (ownership, privileges)
- threads
- process state

The **environment** of a process consists of:

- local and global variables
- current scheduling context
- allocated system resources (ports, file descriptors)

Every process has a process ID (`PID`) and can create a child process with the `fork` system call. The main process is `systemd` which its PID is 1.

```bash
systemd(1)─┬─NetworkManager(782)─┬─{NetworkManager}(783)
           │                     ├─{NetworkManager}(784)
           │                     └─{NetworkManager}(785)
           ├─agetty(810)
           ├─auditd(710)───{auditd}(711)
           ├─chronyd(719)
           ├─containerd(814)─┬─{containerd}(826)
           │                 ├─{containerd}(827)
           │                 ├─{containerd}(828)
           │                 ├─{containerd}(829)
           │                 ├─{containerd}(830)
           │                 ├─{containerd}(831)
           │                 └─{containerd}(832)
           ├─crond(805)
           ├─dbus-broker-lau(717)───dbus-broker(718)
           ├─dockerd(843)─┬─{dockerd}(913)
           │              ├─{dockerd}(914)
           │              ├─{dockerd}(915)
           │              ├─{dockerd}(1043)
           │              ├─{dockerd}(1073)
           │              ├─{dockerd}(1076)
           │              └─{dockerd}(1226)
           ├─firewalld(721)─┬─{firewalld}(786)
           │                ├─{firewalld}(1516)
           │                └─{firewalld}(1518)
           ├─python3(846)───ttyd(2021)
           ├─rsyslogd(847)─┬─{rsyslogd}(865)
           │               └─{rsyslogd}(866)
           ├─sshd(793)───sshd-session(4074)───sshd-session(4096)───bash(4097)───su(4295)───bash(4299)───pstree(5646)
           ├─systemd(4085)───(sd-pam)(4087)
           ├─systemd-journal(577)
           ├─systemd-logind(722)
           ├─systemd-udevd(623)
           ├─systemd-userdbd(593)─┬─systemd-userwor(5642)
           │                      ├─systemd-userwor(5643)
           │                      └─systemd-userwor(5644)
           └─tmux: server(2019)───bash(2020)
```

## Process states

Each CPU (or CPU core) can work on one process at a time. This means that not all processes can run at the same time, which means that processes can be in different states:

| State      | Flags              |
|------------|--------------------|
| Running    | `R`                |
| Sleeping   | `S`, `D`, `K`, `I` |
| Stopped    | `T`                |
| Zombie     | `Z`, `X`           |

You can view the state of a process with `top` (`S` column):

```bash
[s-gas@localhost ~]$ top -b | head -10
top - 00:28:04 up 1 day,  2:48,  1 user,  load average: 0.01, 0.05, 0.00
Tasks: 107 total,   1 running, 106 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us, 16.7 sy,  0.0 ni, 83.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :   1568.4 total,    829.5 free,    348.6 used,    481.1 buff/cache     
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   1219.9 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   6325 s-gas     20   0   17608   7388   5204 S  10.0   0.5   0:04.31 sshd-se+
   6537 s-gas     20   0  231828   4648   2644 R  10.0   0.3   0:00.01 top
      1 root      20   0   45012  35932  10040 S   0.0   2.2   0:03.32 systemd
```

You can also use `ps` (`STAT` column):

```bash
[s-gas@localhost ~]$ ps aux | head -5
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  2.2  45012 35932 ?        Ss   Sep22   0:03 /usr/lib/systemd/systemd --switched-root --system --deserialize=50
root           2  0.0  0.0      0     0 ?        S    Sep22   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    Sep22   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   Sep22   0:00 [kworker/R-rcu_gp]
```
