# Signals

A signal is an asynchronous notification to a process.

## Process management signals

| Signal | Name       | Definition                                                                        |
|--------|------------|-----------------------------------------------------------------------------------|
| `1`    | `SIGHUP`   | `Hangup`: reports termination of the terminal. Also requests configuration reload |
| `2`    | `SIGINT`   | `Keyboard interrupt`: causes program termination via the keyboard (`Ctrl-C`)      |
| `3`    | `SIGQUIT`  | `Keyboard quit`: causes program termination and adds process dump via (`Ctrl-\`)  |
| `9`    | `SIGKILL`  | `Kill`: causes abrupt program termination.(unblockable)                           |
| `15`   | `SIGTERM`  | `Terminate`: causes program termination. Default signal for program termination   |
| `18`   | `SIGCONT`  | `Continue`: it resumes a program previously stopped. (unblockable)                |
| `19`   | `SIGSTOP`  | `Stop`: it suspends a program. (unblockable)                                      |
| `20`   | `SIGTSTP`  | `Keyboard stop`: it suspends a program via the keyboard (`Ctrl-Z`)                |

> Signal numbers vary between Linux hardware platforms, but signal names and meanings are standard. It is advised to use signal names rather than numbers when signaling.

## Sending signals

Keyboard signals (`SIGINT`, `SIGQUIT`, and `SIGTSTP`) can be used to interact with foreground processes. To send signals to processes that are not in the foreground, you can use `kill` and specifying the signal that you want to send, either by name or by signal number. To view the list of available signals, you can use `kill -l`:

```bash
[s-gas@localhost ~]$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
```

The `kill` command, when used without any flags, defaults to `SIGTERM`:

```bash
[s-gas@localhost ~]$ sleep 1000 &
[1] 6829
[s-gas@localhost ~]$ kill 6829
[s-gas@localhost ~]$ ps
    PID TTY          TIME CMD
   6787 pts/1    00:00:00 bash
   6831 pts/1    00:00:00 ps
[1]+  Terminated              sleep 1000
```

As said, you can use `kill` with the signal number:

```bash
[s-gas@localhost ~]$ sleep 1000 &
[1] 6834
[s-gas@localhost ~]$ kill -9 6834
[1]+  Killed                  sleep 1000
```

Or also with the signal name:

```bash
[s-gas@localhost ~]$ sleep 1000 &
[1] 6835
[s-gas@localhost ~]$ kill -SIGKILL 6835
[1]+  Killed                  sleep 1000
```

And also with the job number:

```bash
[user@localhost ~]$ sleep 1000 &
[1] 7477
[user@localhost ~]$ kill %1
[user@localhost ~]$ jobs
[1]+  Terminated              sleep 1000
```

Another command to send signals is `pkill`, which sends signals to the processes that match its criteria.

The default criteria is the command name:

```bash
[s-gas@localhost ~]$ sleep 100 &
[1] 6867
[s-gas@localhost ~]$ pkill sleep
[1]+  Terminated              sleep 100
```

Specifying the command name will send a signal to all the processes with that name:

```bash
[s-gas@localhost ~]$ sleep 100 &
[1] 6869
[s-gas@localhost ~]$ sleep 100 &
[2] 6870
[s-gas@localhost ~]$ sleep 100 &
[3] 6871
[s-gas@localhost ~]$ pkill sleep
[s-gas@localhost ~]$ ps
    PID TTY          TIME CMD
   6787 pts/1    00:00:00 bash
   6873 pts/1    00:00:00 ps
[1]   Terminated              sleep 100
[2]-  Terminated              sleep 100
[3]+  Terminated              sleep 100
```

You can use `pgrep` to *grep* the processes that match certain criteria, which are (almost) the same as the ones for `pkill`:

```bash
[s-gas@localhost ~]$ sleep 1000 &
[1] 6886
[s-gas@localhost ~]$ sleep 1000 &
[2] 6887
[s-gas@localhost ~]$ pgrep sleep
6886
6887
```

You can use the `-l` flag to display also the command name, and `-u` to use the user as criteria:

```bash
[s-gas@localhost ~]$ sleep 1000 &
[1] 6892
[s-gas@localhost ~]$ pgrep -l -u s-gas
6775 systemd
6777 (sd-pam)
6786 sshd-session
6787 bash
6892 sleep
```

The `w` command shows who is logged in and their `TTY`:

```bash
[s-gas@localhost ~]$ w
 02:53:11 up 1 day,  5:13,  2 users,  load average: 0.14, 0.06, 0.01
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
s-gas              02:18    2:39m  0.00s  0.01s sshd-session: s-gas [priv]
user     pts/3     02:49   57.00s  0.00s   ?    -bash
```

Knowing the terminal of a user can be useful to kill all processes related to the terminal session (`pkill -t`):

```bash
[root@localhost ~]$ pgrep -l -u user
6998 systemd
7000 (sd-pam)
7095 sleep
7158 bash
7188 sleep
[root@localhost ~]$ pkill -t 'pts/3'
[root@localhost ~]$ pgrep -l -u user
```
