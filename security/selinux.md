# SELinux

SELinux stands for Security Enhanced Linux and it provides an additional layer of security through **MAC** (Mandatory Access Control) policies which are added to the DAC (Discretionary Access Control).

## DAC (Discretionary Access Control)

DAC is file permission model that specifies that every file has a user owner and a group owner, which means that the permissions (`r`, `w`, `x`) of every file are set for the user owner, the group owner and for the others.

## MAC (Mandatory Access Control)

MAC rules apply to all users and cannot be bypassed by simply changing the permissions of a file. SELinux uses this MAC policies to have granular control over who and what can access specific files, directories and ports.

## SELinux operational modes

SELinux can be in 3 different modes:
- **Enforcing**: the policies provided are enforced (default mode)
- **Permissive**: the policies are loaded but not enforced. Violations are logged (useful for testing and troubleshooting)
- **Disabled**: the policies are not loaded. Strongly discouraged!

You can view the operational mode with `getenforce`:

```bash
[s-gas@localhost /]$ getenforce
Enforcing
```

You can change the mode (enforcing/permissive) with `setenforce`:

```bash
[root@localhost ~]# setenforce
usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce
Permissive
[root@localhost ~]# setenforce 1
[root@localhost ~]# getenforce
Enforcing
```

> It is recommended to reboot the machine when changing the mode.

To disable SELinux, pass `selinux=0` during boot.

You can also configure SELinux mode by modifying the `/etc/selinux/config` file:

```bash
[root@localhost ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

## Context labels

Every resource (file, process, directory) is labelled with a SELinux context, which matches a policy rule that tells what can access that resource. By default, an SELinux policy does not allow any access unless an explicit rule grants access.

SELinux context labels have the following fields:
- `user`
- `role`
- `type`
- `security level`

Many commands include the `-Z` flag, which display the context labels:

```bash
[s-gas@localhost /]$ ls -Z /etc/login.defs 
system_u:object_r:etc_t:s0 /etc/login.defs
```

In this case, `/etc/login.defs` is labelled with:
- `user`: `system_u`
- `role`: `object_r`
- `type`: `etc_t`
- `security level`: `s0`

Filenames are listed together with a label policy in `/etc/selinux/targeted/contexts/files/`. Files that don't match an existing labeling policy inherit the same label of the parent directory. In this way it is guaranteed that every file has a SELinux context label.

This also means that copying a file from a directory to another might create a file with a different context label than the original. To preserve the context label when using the `cp` command, run it as `cp --preserve=context`:

```bash
[s-gas@localhost ~]$ cp /etc/ssh/ssh_config ~
[s-gas@localhost ~]$ ls -Z ssh_config 
unconfined_u:object_r:user_home_t:s0 ssh_config
[s-gas@localhost ~]$ rm ssh_config 
[s-gas@localhost ~]$ cp --preserve=context /etc/ssh/ssh_config ~
[s-gas@localhost ~]$ ls -Z ssh_config 
system_u:object_r:etc_t:s0 ssh_config
```

Moving a file, on the other hand, since it does not create a new inode, doesn't modify the context of that file. If you want to set a new context for that file according to its new path, run `mv -Z`:

```bash
[root@localhost ~]# touch file.txt
[root@localhost ~]# mv file.txt /etc/
[root@localhost ~]# ls -Z /etc/file.txt
unconfined_u:object_r:admin_home_t:s0 /etc/file.txt
[root@localhost ~]# rm -rf /etc/file.txt
[root@localhost ~]# touch file.txt
[root@localhost ~]# mv -Z file.txt /etc/
[root@localhost ~]# ls -Z /etc/file.txt
unconfined_u:object_r:etc_t:s0 /etc/file.txt
```

### Change context labels

To manually change a label (temporary change), you can use `chcon`:

```bash
[root@localhost ~]# touch file.txt
[root@localhost ~]# ls -Z file.txt 
unconfined_u:object_r:admin_home_t:s0 file.txt
[root@localhost ~]# chcon -t 'default_t' file.txt 
[root@localhost ~]# ls -Z file.txt 
unconfined_u:object_r:default_t:s0 file.txt
```

This relabel technique will survive reboot, but it is still considered temporary because, if `restorecon` is run on that file, it will replace the modified label with its correct context (based on the database `/etc/selinux/targeted/context/files/`):

```bash
[root@localhost ~]# ls -Z file.txt 
unconfined_u:object_r:default_t:s0 file.txt
[root@localhost ~]# restorecon -v file.txt 
Relabeled /root/file.txt from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:admin_home_t:s0
```

To restore the right policy for a directory and its children, run `restorecon -R`:

```bash
[root@localhost /]# restorecon -R /new
```

The recommended way of changing a context label is through the `semanage fcontext` command, which displays and modifies the policies that determine the context label of a file.

The command is not installed by default, you need to install the `policycoreutils-python-utils` package:

```bash
[root@localhost /]# dnf install policycoreutils-python-utils -y > /dev/null
```

To list all the policies, you can run `semanage fcontext -l`:

```bash
[root@localhost ~]# semanage fcontext -l | grep '(/.*)?' | tail -5
/var/www/svn/hooks(/.*)?                           all files          system_u:object_r:httpd_sys_script_exec_t:s0 
/var/www/usage(/.*)?                               all files          system_u:object_r:webalizer_rw_content_t:s0 
/var/www/wiki[0-9]?(/.*)?                          all files          system_u:object_r:mediawiki_rw_content_t:s0 
/var/yp(/.*)?                                      all files          system_u:object_r:var_yp_t:s0 
/xen(/.*)?                                         all files          system_u:object_r:xen_image_t:s0 
```

SELinux policies allow regex and many policies use **the pirate** (`(/.*)?`), which is appended to a directory and matches patterns where "a group of characters that start with `/` and followed by any number of characters (`.*`) is present, or not (`?`). This is used to apply the same policy to all the children of a directory.

To create a new policy, run `semanage fcontext -a`:

```bash
[root@localhost /]# mkdir /new
[root@localhost /]# ls -dZ /new
unconfined_u:object_r:default_t:s0 /new
[root@localhost /]# semanage fcontext -a -t 'tmp_t' '/new(/.*)?'
[root@localhost /]# restorecon -v /new
Relabeled /new from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:tmp_t:s0
```

With `semanage fcontext -l -C` you can list only the local policies:

```bash
[root@localhost /]# semanage fcontext -l -C
SELinux fcontext                                   type               Context

/new(/.*)?                                         all files          system_u:object_r:tmp_t:s0 
```

To delete a policy, run `semanage fcontext -d`:

```bash
[root@localhost /]# semanage fcontext -d '/new(/.*)?'
[root@localhost /]# ls -dZ /new
unconfined_u:object_r:tmp_t:s0 /new
[root@localhost /]# restorecon -v /new
Relabeled /new from unconfined_u:object_r:tmp_t:s0 to unconfined_u:object_r:default_t:s0
```

## SELinux booleans

The SELinux policy of an application can also include optional behaviours/features. These behaviours can be enable or disabled through SELInux booleans.

The command to interact with SELinux is `semanage boolean`.

To list all the booleans, run `semanage boolean -l`:

```bash
[root@localhost /]# semanage boolean -l | grep 'httpd'
httpd_anon_write               (off  ,  off)  Allow httpd to anon write
httpd_builtin_scripting        (on   ,   on)  Allow httpd to builtin scripting
httpd_can_check_spam           (off  ,  off)  Allow httpd to can check spam
httpd_can_connect_ftp          (off  ,  off)  Allow httpd to can connect ftp
httpd_can_connect_ldap         (off  ,  off)  Allow httpd to can connect ldap
httpd_can_connect_mythtv       (off  ,  off)  Allow httpd to can connect mythtv
httpd_can_connect_zabbix       (off  ,  off)  Allow httpd to can connect zabbix
httpd_can_manage_courier_spool (off  ,  off)  Allow httpd to can manage courier spool
httpd_can_network_connect      (off  ,  off)  Allow httpd to can network connect
httpd_can_network_connect_cobbler (off  ,  off)  Allow httpd to can network connect cobbler
httpd_can_network_connect_db   (off  ,  off)  Allow httpd to can network connect db
httpd_can_network_memcache     (off  ,  off)  Allow httpd to can network memcache
httpd_can_network_redis        (off  ,  off)  Allow httpd to can network redis
httpd_can_network_relay        (off  ,  off)  Allow httpd to can network relay
httpd_can_sendmail             (off  ,  off)  Allow httpd to can sendmail
httpd_dbus_avahi               (off  ,  off)  Allow httpd to dbus avahi
httpd_dbus_sssd                (off  ,  off)  Allow httpd to dbus sssd
httpd_dontaudit_search_dirs    (off  ,  off)  Allow httpd to dontaudit search dirs
httpd_enable_cgi               (on   ,   on)  Allow httpd to enable cgi
httpd_enable_ftp_server        (off  ,  off)  Allow httpd to enable ftp server
httpd_enable_homedirs          (off  ,  off)  Allow httpd to enable homedirs
httpd_execmem                  (off  ,  off)  Allow httpd to execmem
httpd_graceful_shutdown        (off  ,  off)  Allow httpd to graceful shutdown
httpd_manage_ipa               (off  ,  off)  Allow httpd to manage ipa
httpd_mod_auth_ntlm_winbind    (off  ,  off)  Allow httpd to mod auth ntlm winbind
httpd_mod_auth_pam             (off  ,  off)  Allow httpd to mod auth pam
httpd_read_user_content        (off  ,  off)  Allow httpd to read user content
httpd_run_ipa                  (off  ,  off)  Allow httpd to run ipa
httpd_run_preupgrade           (off  ,  off)  Allow httpd to run preupgrade
httpd_run_stickshift           (off  ,  off)  Allow httpd to run stickshift
httpd_serve_cobbler_files      (off  ,  off)  Allow httpd to serve cobbler files
httpd_setrlimit                (off  ,  off)  Allow httpd to setrlimit
httpd_ssi_exec                 (off  ,  off)  Allow httpd to ssi exec
httpd_sys_script_anon_write    (off  ,  off)  Allow httpd to sys script anon write
httpd_tmp_exec                 (off  ,  off)  Allow httpd to tmp exec
httpd_tty_comm                 (off  ,  off)  Allow httpd to tty comm
httpd_unified                  (off  ,  off)  Allow httpd to unified
httpd_use_cifs                 (off  ,  off)  Allow httpd to use cifs
httpd_use_fusefs               (off  ,  off)  Allow httpd to use fusefs
httpd_use_gpg                  (off  ,  off)  Allow httpd to use gpg
httpd_use_nfs                  (off  ,  off)  Allow httpd to use nfs
httpd_use_opencryptoki         (off  ,  off)  Allow httpd to use opencryptoki
httpd_use_openstack            (off  ,  off)  Allow httpd to use openstack
httpd_use_sasl                 (off  ,  off)  Allow httpd to use sasl
httpd_verify_dns               (off  ,  off)  Allow httpd to verify dns
```

The output displays two states for each boolean, `(off  , off)`, the first is the current state, the second is the persistent state.

To change the current state of a boolean, run `setsebool`:

```bash
[root@localhost /]# setsebool httpd_use_nfs on
[root@localhost /]# semanage boolean -l | grep httpd_use_nfs
httpd_use_nfs                  (on   ,  off)  Allow httpd to use nfs
```

You can also view the current state of a specific boolean with `getsebool`:

```bash
[root@localhost /]# getsebool httpd_use_nfs
httpd_use_nfs --> on
```

To change also the persistent state, run `setsebool -P`:

```bash
[root@localhost /]# setsebool -P httpd_use_nfs on
[root@localhost /]# semanage boolean -l | grep httpd_use_nfs
httpd_use_nfs                  (on   ,   on)  Allow httpd to use nfs
```

As for `semanage fcontext`, also for booleans you can list only booleans that have been modified. Use `semanage boolean -l -C`:

```bash
[root@localhost /]# semanage boolean -l -C
SELinux boolean                State  Default Description

httpd_use_nfs                  (on   ,   on)  Allow httpd to use nfs
```
