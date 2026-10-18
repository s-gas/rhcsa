# SFTP

SFTP stands for Secure File Transfer Protocol and is provided by OpenSSH for transfering files over SSH.

The command `sftp` uses the SFTP protocol and it lets you interactively upload or download files from a system running a SSH server:

```bash
❯ sftp rocky
Connected to rocky.
sftp> exit
```

In the interactive session you can use the same commands that you would use locally:

```bash
❯ sftp rocky
Connected to rocky.
sftp> ls
file.txt  
sftp> rm file.txt
Removing /home/user/file.txt
sftp> pwd
Remote working directory: /home/user
sftp> exit
```

`sftp` provides also extra commands, as you can see from typing `help` in the interactive session:

```bash
sftp> help
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp [-h] grp path                Change group of file 'path' to 'grp'
chmod [-h] mode path               Change permissions of file 'path' to 'mode'
chown [-h] own path                Change owner of file 'path' to 'own'
copy oldpath newpath               Copy remote file
cp oldpath newpath                 Copy remote file
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afpR] remote [local]         Download file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
lumask umask                       Set local umask to 'umask'
mkdir path                         Create remote directory
progress                           Toggle display of progress meter
put [-afpR] local [remote]         Upload file
pwd                                Display remote working directory
quit                               Quit sftp
reget [-fpR] remote [local]        Resume download file
rename oldpath newpath             Rename remote file
reput [-fpR] local [remote]        Resume upload file
rm path                            Delete remote file
rmdir path                         Remove remote directory
symlink oldpath newpath            Symlink remote file
version                            Show SFTP version
!command                           Execute 'command' in local shell
!                                  Escape to local shell
?                                  Synonym for help
```

To upload a file or a directory, use the `put` command (`put -r` for recursion):

```bash
sftp> lls
bash			logging			processes		scheduling		software-packages
filesystem		networking		README.md		security
sftp> put -r bash 
Uploading bash/ to /home/user/bash
Entering bash/
regex.md                                                                                                                                                                           100% 2344     1.2MB/s   00:00    
expansions.md                                                                                                                                                                      100% 1999     2.6MB/s   00:00    
scripts.md                                                                                                                                                                         100%   16KB  12.9MB/s   00:00    
variables.md                                                                                                                                                                       100% 2067     2.6MB/s   00:00    
redirection.md                                                                                                                                                                     100% 2225     3.8MB/s   00:00    
sftp> ls
bash  
```

To download from the server, use the `get` command (`get -r` for recursion):

```bash
sftp> get -r /etc/yum.repos.d/
Fetching /etc/yum.repos.d/ to yum.repos.d
Retrieving /etc/yum.repos.d
rocky-addons.repo                                                                                                                                                                  100% 5490     3.3MB/s   00:00    
rocky-devel.repo                                                                                                                                                                   100% 1168     1.4MB/s   00:00    
rocky-extras.repo                                                                                                                                                                  100% 2393     3.2MB/s   00:00    
rocky-security.repo                                                                                                                                                                100% 1428     1.8MB/s   00:00    
rocky.repo                                                                                                                                                                         100% 3426     5.2MB/s   00:00    
epel-testing.repo                                                                                                                                                                  100% 1714     1.7MB/s   00:00    
epel.repo                                                                                                                                                                          100% 1612     2.2MB/s   00:00    
docker-ce.repo       
```

You can also download a file without entering the interactive mode by specifying the path to the file:

```bash
❯ sftp rocky:/etc/yum.repos.d/rocky.repo
Connected to rocky.
Fetching /etc/yum.repos.d/rocky.repo to rocky.repo
rocky.repo                                                                                                                                                                         100% 3426     2.9MB/s   00:00    
Connection closed.  
```

Another command that, now, uses the SFTP protocol is `scp`. The commands let's you copy a file remotely by specifying where in the remote filesystem to upload the file:

```bash
❯ scp ~/code/rhcsa/bash/expansions.md rocky:/home/user
expansions.md                                                                                                                                                                      100% 1999     1.7MB/s   00:00    
❯ ssh rocky
Last login: Sun Oct 18 04:59:58 2026 from 10.0.2.2
[user@rocky ~]$ ls
expansions.md
```
