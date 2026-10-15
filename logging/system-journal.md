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

For specifying the logs of a service use `journal -u`:

```bash
[root@rocky /]$ journalctl -u sshd.service -n 3 -p info
Oct 15 04:37:42 rocky sshd-session[3374]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
Oct 15 04:37:53 rocky sshd-session[3423]: Accepted publickey for user from 10.0.2.2 port 64347 ssh2: ED25519 SHA256:qm4570fHteIKnYegN7WbD9EBNQGFxJceoJmkR0ISo8c
Oct 15 04:37:53 rocky sshd-session[3423]: pam_unix(sshd:session): session opened for user user(uid=1001) by user(uid=0)
```
