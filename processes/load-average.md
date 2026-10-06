# Load average

Load average represents the perceived system load for a period of time.

## Load average calculation

The load number is obtained by calculating the average of the number of process that are in one of the following states:
- running or runnable state (`R`)
- uninterruptible sleep (`D`): waiting for disk or network I/O

Other UNIX systems consider only running processes to calculate the load average. Linux also includes disk or network usage, because the high usage of these resources can significantly impact system performance as CPU load.

## Interpret load average

The command `uptime` shows you, apart from the uptime of the machine, the load average:

```bash
[user@localhost ~]$ uptime
 21:29:23 up 1 day, 23:49,  1 user,  load average: 0.00, 0.04, 0.01
 ```

The 3 values represent the load average for the last:
- 1 minute
- 5 minutes
- 15 minutes

The reason of having 3 values is to see if the load average is increasing or decreasing.

Another command that displays the load average is `w`:

```bash
[s-gas@localhost ~]$ w
 06:21:40 up 2 days,  8:42,  2 users,  load average: 0.14, 0.07, 0.02
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
s-gas              21:29   30:07m  0.00s  0.01s sshd-session: s-gas [priv]
s-gas              05:31   30:07m  0.00s  0.03s sshd-session: s-gas [priv]
```

To understand what the values mean, you also need to know the number of CPUs in the system. To do that you can use `lscpu`:

```bash
[user@localhost ~]$ lscpu | head -15
Architecture:                            aarch64
CPU op-mode(s):                          64-bit
Byte Order:                              Little Endian
CPU(s):                                  1
On-line CPU(s) list:                     0
Vendor ID:                               Apple
Model name:                              -
Model:                                   0
Thread(s) per core:                      1
Core(s) per cluster:                     1
Socket(s):                               -
Cluster(s):                              1
Stepping:                                0x0
BogoMIPS:                                48.00
Flags:                                   fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm jscvt fcma lrcpc dcpop sha3 asimddp sha512 asimdfhm dit uscat ilrcpc flagm ssbs sb paca pacg dcpodp flagm2 frint bf16 afp
```

This example shows a system with a single CPU. A load average higher than the number of CPUs would indicate an overloaded system.

To give another example, a load average of `4.3` on a system with `4` CPUs would indicate that the system is overloaded because there are more processes in state `R` and `D` than the number of actual CPUs.

## Real-time process monitoring

The `top` command display dinamically information of the system's processes, including the load average.

```bash
[user@localhost ~]$ top -b | head 
top - 22:19:59 up 2 days, 40 min,  1 user,  load average: 0.00, 0.03, 0.00
Tasks: 110 total,   1 running, 109 sleeping,   0 stopped,   0 zombie
%Cpu(s):  9.1 us,  0.0 sy,  0.0 ni, 81.8 id,  0.0 wa,  0.0 hi,  9.1 si,  0.0 st 
MiB Mem :   1568.4 total,    825.5 free,    352.1 used,    481.1 buff/cache     
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   1216.3 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   9268 s-gas     20   0   17608   7392   5204 S   9.1   0.5   0:00.84 sshd-se+
      1 root      20   0   45012  35932  10040 S   0.0   2.2   0:05.45 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.08 kthreadd
```

The first line is the same information given by `uptime` and then you can also see information about each process. By default, the processes are sorted decrementally according to CPU use. This can be changed through keystrokes:

| Keystroke | Purpose                                                    |
|-----------|------------------------------------------------------------|
| `shift-M` | Sort processes by memory usage, in descending order        |
| `shift-P` | Sort processes by CPU usage, in descending order (default) |

Other keystrokes:

| Keystroke | Purpose                                                    |
|-----------|------------------------------------------------------------|
| `h` or `?`| Displays the help screen for the keystrokes                |
| `q`       | Quit                                                       |
| `u`       | Filter processes by user                                   |
| `k`       | Kill a process                                             |
