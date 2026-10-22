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

`/etc/login.defs` is labelled with:
- `user`: `system_u`
- `role`: `object_r`
- `type`: `etc_t`
- `security level`: `s0`
