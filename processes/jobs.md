# Jobs

A job is a process (or group of processes) that the shell treats as a single unit.

This means that a job is associated with each pipeline entered as command through the shell.

```bash
[s-gas@localhost ~]$ sleep 2 | cat
```

These are two processes, but a single job.

Only one job can read input in a single shell session. The processes that are part of that job are the **foreground processes**.

On the other hand, **background processes** are the processes that are not part of the job that can read input from the keyboard. This does not mean that these processes cannot write output to the terminal.

The `ps` command shows the device name of the controlling terminal of the processes (`TTY` column):

```bash
[s-gas@localhost ~]$ ps aux | head -5
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  2.2  45012 35932 ?        Ss   Sep22   0:03 /usr/lib/systemd/systemd --switched-root --system --deserialize=50
root           2  0.0  0.0      0     0 ?        S    Sep22   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    Sep22   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   Sep22   0:00 [kworker/R-rcu_gp]
```

```bash
[s-gas@localhost ~]$ ps aux | tail -5
root        6526  0.0  0.3  16224  5644 ?        S    00:26   0:00 systemd-userwork: waiting...
root        6532  0.0  0.3  16224  5648 ?        S    00:27   0:00 systemd-userwork: waiting...
root        6536  0.0  0.0      0     0 ?        I    00:27   0:00 [kworker/0:2-events]
s-gas       6543  0.0  0.2 230760  3664 pts/1    R+   00:30   0:00 ps aux
s-gas       6544  0.0  0.1 226680  1784 pts/1    S+   00:30   0:00 tail -5
```

The entries with `?` under the `TTY` column are **system daemons**, which are not members of a job since they are started by the system and not by the shell. This also means, that they cannot be brought to the foreground.

## Run jobs in the background

Any job can be started in the background by appending `&` to the command:

```bash
[s-gas@localhost ~]$ sleep 5 &
[1] 6553
```

The output given by the job is the **job number** (`[1]`) and the PID of the process (`6553`).

If the job consists of a pipeline of multiple processes, the PID of the last process will be displayed:

```bash
[s-gas@localhost ~]$ find . -name *.conf 2>/dev/null | wc -l &
[2] 6561
```

With `jobs` you can view the jobs that are currently running in the background:

```bash
[s-gas@localhost ~]$ sleep 20 &
[1] 6564
[s-gas@localhost ~]$ sleep 10 &
[2] 6565
[s-gas@localhost ~]$ sleep 5 &
[3] 6566
[s-gas@localhost ~]$ jobs
[1]   Running                 sleep 20 &
[2]-  Running                 sleep 10 &
[3]+  Running                 sleep 5 &
```

You can bring a job to the foreground with `fg`:

```bash
[s-gas@localhost ~]$ sleep 10 &
[1] 6594
[s-gas@localhost ~]$ jobs
[1]+  Running                 sleep 10 &
[s-gas@localhost ~]$ fg %1
sleep 10
```

The job that is the current default is indicated with a `+`:

```bash
[s-gas@localhost ~]$ sleep 10 &
[1] 6644
[s-gas@localhost ~]$ sleep 20 &
[2] 6645
[s-gas@localhost ~]$ jobs
[1]-  Running                 sleep 10 &
[2]+  Running                 sleep 20 &
[s-gas@localhost ~]$ fg
sleep 20
^C
[1]   Done                    sleep 10
```

This shows that you can interact with the current default (`2` in this case) without specifying its number.
The process, after being brought to the foreground, is killed with `Ctrl-C` (keyboard `SIGINT`).

From the foreground you can a job to the background and suspend it with `Ctrl-Z` (keyboard `SIGTSTP`):

```bash
[s-gas@localhost ~]$ sleep 10
^Z
[1]+  Stopped                 sleep 10
[s-gas@localhost ~]$ jobs
[1]+  Stopped                 sleep 10
```

You can start a suspendend job with `bg`:

```bash
[s-gas@localhost ~]$ sleep 10
^Z
[1]+  Stopped                 sleep 10
[s-gas@localhost ~]$ bg %1
[1]+ sleep 10 &
```

`ps j` displays information about the processes and the session:

```bash
[s-gas@localhost ~]$ sleep 10 | sleep 20 &
[1] 6617
[s-gas@localhost ~]$ ps j
   PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
   6325    6326    6326    6326 pts/1       6618 Ss    1000   0:00 -bash
   6326    6616    6616    6326 pts/1       6618 S     1000   0:00 sleep 10
   6326    6617    6616    6326 pts/1       6618 S     1000   0:00 sleep 20
   6326    6618    6618    6326 pts/1       6618 R+    1000   0:00 ps j
```

It displays:
- `PPID`: parent process ID (the process that started it)
- `PID`: process ID
- `PGID`: group leader process ID (normally the first process in the pipeline)
- `SID`: session leader process ID (normally is the shell through which the process was run)
