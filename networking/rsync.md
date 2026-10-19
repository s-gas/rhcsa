# `rsync`

The `rsync` command provides a way to synchronize local and remote files. It uses an algorithm that allows to copy only the portions of the files that differs. This allows for a faster synchronization comparing to copying files or directories.

To do a dry run, which means to simulate the outcome of the command, use `rsync -nv` (`-v` is for verbose):

```bash
❯ rsync -nv .bashrc rocky:/home/user/.bashrc
Transfer starting: 1 files
.bashrc

sent 51 bytes  received 26 bytes  70000 bytes/sec
total size is 183  speedup is 2.38
```

To transfer files or directories recursively and preserving some metadata (permissions, timestamps, etc), use `rsync -av`:

```bash
❯ rsync -av .bashrc rocky:/home/user/.bashrc
Transfer starting: 1 files
.bashrc

sent 320 bytes  received 48 bytes  175238 bytes/sec
total size is 183  speedup is 0.50
```

> `rsync -a` does not preserve hard links, ACLs and SELinux file contexts

You can of course synchronize also from the remote server to the local machine:

```bash
❯ rsync -av rocky:/etc/ssh server-ssh 
rsync: [sender] opendir "/etc/ssh/sshd_config.d" failed: Permission denied (13)
Transfer starting: 13 files
ssh/
ssh/moduli
rsync: [sender] send_files failed to open "/etc/ssh/ssh_host_ecdsa_key": Permission denied (13)
rsync: [sender] send_files failed to open "/etc/ssh/ssh_host_ed25519_key": Permission denied (13)
rsync: [sender] send_files failed to open "/etc/ssh/ssh_host_rsa_key": Permission denied (13)
rsync: [sender] send_files failed to open "/etc/ssh/sshd_config": Permission denied (13)
ssh/ssh_config
ssh/ssh_host_ecdsa_key.pub
ssh/ssh_host_ed25519_key.pub
ssh/ssh_host_rsa_key.pub
ssh/ssh_config.d/
ssh/ssh_config.d/50-redhat.conf
ssh/sshd_config.d/

sent 254 bytes  received 545723 bytes  5459770000 bytes/sec
total size is 552130  speedup is 1.01
rsync(85365): warning: child 85366 exited with status 23
```
