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
[user@rocky ~]$ pstree -p
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

You can specify the PID of the process that you want to view with `ps`:

```bash
[root@localhost ~]# sleep 1000 &
[1] 2268
[root@localhost ~]# ps 2268
    PID TTY      STAT   TIME COMMAND
   2268 pts/1    S      0:00 sleep 1000
```

If you only know the name of the process you can combine `ps` with `pgrep`:

```bash
[root@localhost ~]# for i in {1..3}; do sleep 1000 & done
[1] 2272
[2] 2273
[3] 2274
[root@localhost ~]# ps u $(pgrep 'sleep')
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        2272  0.0  0.1 226648  1736 pts/1    S    21:32   0:00 sleep 1000
root        2273  0.0  0.1 226648  1732 pts/1    S    21:32   0:00 sleep 1000
root        2274  0.0  0.1 226648  1740 pts/1    S    21:32   0:00 sleep 1000
```

## Process scheduler

To give the illusion that multiple processes are running at the same time, the **process scheduler** rapidly changes between processes. This is called **time-slicing** or **multi-tasking**.

## Process priority

The priority of a process can be modified through its **nice** value:
- `-20`: highest priority
- `0`: default priority
- `19`: lowest priority

Processes inherit the nice value from their parent process.

A nice value lower than `0` can be set only by `root`!

These values affect the CPU time that a process receives only when the CPU is overloaded, in which case processes with a lower nice value will have priority.

The `top` commands displays the nice value in the `NI` column, the `PR` column is for the priority:

```bash
[root@localhost ~]# top -b1 | head
top - 20:42:31 up  1:19,  1 user,  load average: 0.00, 0.03, 0.00
Tasks: 107 total,   1 running, 106 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  9.1 sy,  0.0 ni, 90.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :   1568.4 total,   1087.2 free,    268.6 used,    289.1 buff/cache     

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0   43992  34924  10044 S   0.0   2.2   0:00.75 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_wo+
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker+
```

From this example you can see that `systemd` has a default nice value (`0`) which maps to a priority of `20`, while `kworker+` has a nice value of `-20` which maps to a priority of `0` (highest priority).

### Change process priority

You can change the nice value directly through the `top` command by typing `r` (renice) and then the PID of the process that you want to modify.

Another way is by using the `renice` command:

```bash
[root@localhost ~]# sleep 1000 &
[2] 2204
[root@localhost ~]# renice -n -19 2204
2204 (process ID) old priority 0, new priority -19
```

### Start a process with a nice value

To start a process with a specified nice value, use the `nice` command:

```bash
[root@localhost ~]# nice -n 19 sleep 1000 &
[1] 2214
[root@localhost ~]# top -b1 | grep 'sleep$'
   2214 root      39  19  226648   1740   1636 S   0.0   0.1   0:00.00 sleep
```
