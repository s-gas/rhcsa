# SSH

SSH stands for Secure Shell and it's a protocol through which systems can communicate in a secure and encrypted way.

To log into another machine you can use the `ssh` and specify the user and the hostname:

```bash
❯ ssh user@localhost -p 2222
user@localhost's password: 
Last login: Thu Sep 24 21:29:20 2026
[user@localhost ~]$ 
```

In this example also the port is specified because it's using a non default port; the default is `22`.

You can also run directly commands into the remote machine without entering the interactive shell:

```bash
❯ ssh user@localhost -p 2222 uname -a
user@localhost's password: 
Linux localhost.localdomain 6.12.0-211.16.1.el10_2.0.1.aarch64 #1 SMP PREEMPT_DYNAMIC Sun May 24 13:38:45 UTC 2026 aarch64 GNU/Linux
```

## SSH Keys

Communication is secured via public-key encryption. When an SSH client connects to a SSH server, the server sends a copy of its public host key to the client, then it checks for a copy of the server's host public key in `/etc/ssh/ssh_known_hosts` and in `~/.ssh/known_hosts`. 

If the public host key is not found, the client treats as a new connection and prompts the user to confirm the server's fingerprint. If confirmed, the public host key is then saved as an entry in `~/.ssh/known_hosts`:

```bash
❯ ssh user@localhost -p 2222
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:FvqNdE6J0IL/4eCAmBxUWBujIXVQHZbRtM2Ke7jjbYs
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[localhost]:2222' (ED25519) to the list of known hosts.
user@localhost's password: 
Last failed login: Fri Sep 25 09:55:32 CEST 2026 from 10.0.2.2 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Fri Sep 25 09:43:17 2026 from 10.0.2.2
```

If the public host key is found but does not match the one received, by default the client refuses to connect, since this could indicate that the network traffic is compromised. This behavior could be altered by setting `StrictHostKeyChecking` to `no`, but it is not advisable. This will also print a warning message:

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:hxttxb/qVi1/ycUU2wXF6mfGH++Ya7WYZv0r+tIkg4I.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/user/.ssh/known_hosts:12
ECDSA host key for server1.example.com has changed and you have requested strict checking.
Host key verification failed.
```

## SSH Known Hosts

As already said, the public host keys of the server can be saved either in:
- `/etc/ssh/ssh_known_hosts` (this file does not exist by default, it needs to be created)
- `~/.ssh/known_hosts`

During a SSH connection, the system looks first in the system-wide configuration file (`/etc/ssh/ssh_known_hosts`) and, if it does not find the key, then looks at user's configuration file (`~/.ssh/known_hosts`).

Each entry looks like this:

```bash
[user@localhost ~]$ cat ~/.ssh/known_hosts
server1 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOmiLKMExRnsS1g7OTxMsOmgHuUSGQBUxHhuUGcv19uT
```

It has 3 fields:
- hostname
- encryption algorithm
- key

## SSH Key-based Authentication

You can use key-based authentication to connect via SSH without having do use a password. To do so, you need to generate a pair of user keys, one is private and one is public. The former needs to be stored securely, while the latter will be shared on the remote server to allow the key-based authentication.

To generate a user key pair, run `ssh-keygen`. By default it saves your keys in `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`. If these two files already exist, the command will NOT silently override them, but it will prompt you:

```bash
❯ ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/simonegasparini/.ssh/id_ed25519): 
/Users/simonegasparini/.ssh/id_ed25519 already exists.
Overwrite (y/n)? 
```

You can also specify the filenames in which to save the keys with `ssh-keygen -f`:

```bash
❯ ssh-keygen -f ~/.ssh/test
Generating public/private ed25519 key pair.
Enter passphrase for "/Users/simonegasparini/.ssh/test" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/simonegasparini/.ssh/test
Your public key has been saved in /Users/simonegasparini/.ssh/test.pub
The key fingerprint is:
SHA256:W0mILKDlvJqFzmvTtrfun+1qA+pB6tlzaWTYfYrwDpg simonegasparini@Simones-MacBook-Pro.local
The key's randomart image is:
+--[ED25519 256]--+
|  o              |
| = . . . .       |
|. o . o . .      |
| . . .   . .     |
|. o.o . S o      |
|o+=o = . +       |
|oE.o* + +        |
|.o+=oB +o        |
|.+++X=+++o       |
+----[SHA256]-----+
```

Now that you have a key pair, the public key needs to be copied with `ssh-copy-id`:

```bash
❯ ssh-copy-id -i ~/.ssh/test.pub -p 2222 s-gas@localhost
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/simonegasparini/.ssh/test.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
s-gas@localhost's password: 

Number of key(s) added:        1

Now try logging into the machine, with: "ssh -i /Users/simonegasparini/.ssh/test -p 2222 's-gas@localhost'"
and check to make sure that only the key(s) you wanted were added.
```

In this example, the public key is specified with the `-i` flag because it is not the default key.

Now you can access the remote server without using the password:

```bash
❯ ssh -p 2222 -i ~/.ssh/test s-gas@localhost
Last login: Fri Sep 25 05:31:36 2026 from 10.0.2.2
[s-gas@localhost ~]$ 
```

If you decide to set a passphrase for your keys, you will need to enter it every time you try to authenticate with that key. To solve that, you can configure `ssh-agent` key manager to cache the passphrase. This of course needs to be done on the client.

To start the `ssh-agent`:

```bash
❯ eval $(ssh-agent)
Agent pid 43775
```

The command `eval` will run the output of `ssh-agent`, which contains the environment variables needed to use `ssh-add`:

```bash
❯ ssh-keygen -f ~/.ssh/test-passphrase
Generating public/private ed25519 key pair.
Enter passphrase for "/Users/simonegasparini/.ssh/test-passphrase" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/simonegasparini/.ssh/test-passphrase
Your public key has been saved in /Users/simonegasparini/.ssh/test-passphrase.pub
The key fingerprint is:
SHA256:sOeeQndDg/FWqAYFU9yfD7FBIScqqwtDPZc5iKcTW3E simonegasparini@Simones-MacBook-Pro.local
The key's randomart image is:
+--[ED25519 256]--+
|      o=o.oo+.   |
|      ..o.o++    |
|    . E..= o =   |
|   o + B+ = =    |
|  + * B.So . o   |
| . * +.+. o   .  |
|  * .. ... .     |
|   + ... .       |
|    .  .o        |
+----[SHA256]-----+


❯ ssh-copy-id -i test-passphrase.pub -p 2222 s-gas@localhost
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "test-passphrase.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
s-gas@localhost's password: 

Number of key(s) added:        1

Now try logging into the machine, with: "ssh -i ./test-passphrase -p 2222 's-gas@localhost'"
and check to make sure that only the key(s) you wanted were added.


❯ ssh-add ~/.ssh/test-passphrase
Enter passphrase for /Users/simonegasparini/.ssh/test-passphrase: 
Identity added: /Users/simonegasparini/.ssh/test-passphrase (simonegasparini@Simones-MacBook-Pro.local)


❯ ssh -p 2222 s-gas@localhost
Last login: Fri Sep 25 11:33:28 2026 from 10.0.2.2
[s-gas@localhost ~]$ 
```

## Troubleshooting

SSH provides 3 levels of verbosity: `-v`, `-vv`, `-vvv`.

## SSH Client Configuration

To avoid having to specify command parameters every time, you can create a `~/.ssh/config` file with the wished configuration:

```bash
❯ cat ~/.ssh/config 
host rocky
	HostName	 	localhost
	User		 	user
	Port		 	2222
	IdentityFile	~/.ssh/test-passphrase

❯ ssh rocky
Last login: Fri Sep 25 11:48:56 2026 from 10.0.2.2
```

This configuration lets me run `ssh rocky` without having to specify all the parameters (hostname, user, port, key).

## SSH Server Configuration

The SSH service is provided by the `sshd` daemon. You can configure the server by editing `/etc/ssh/sshd_config` or by adding a `.conf` file into the drop-in directory `/etc/ssh/sshd_config.d/`.

Common good practices are:
- prohibit `root` access:

```bash
[root@localhost ssh_config.d]$ echo "PermitRootLogin no" > /etc/ssh/sshd_config.d/00-prohibit-root.conf
[root@localhost ssh_config.d]$ systemctl reload-or-restart sshd
```

> Remember to reload the service when modifying the configuration files!

- disable password-based authentication:

```bash
[root@localhost ssh]$ echo "PasswordAuthentication no" > /etc/ssh/sshd_config.d/01-no-password.conf
[root@localhost ssh]$ systemctl reload-or-restart sshd
```

Files should be named with a numer prefix, because files are included in alphabetical order and the first valued obtained wins:

```bash
[root@localhost sshd_config.d]$ echo "PasswordAuthentication no" > /etc/ssh/sshd_config.d/00-no-pass.conf
[root@localhost sshd_config.d]$ echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/01-no-pass.conf
[root@localhost sshd_config.d]$ sshd -T | grep -i 'PasswordAuthentication'
passwordauthentication no
```

This example shows that the configuration of `sshd` does not allow password authentication.
