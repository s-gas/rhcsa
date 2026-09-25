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
