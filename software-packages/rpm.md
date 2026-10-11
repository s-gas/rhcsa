# RPM

Red Hat provides all software for RHEL as RPM packages.

```bash
[user@rocky ~]$ rpm -q NetworkManager
NetworkManager-1.56.0-1.el10.aarch64
```

`rpm -q` displays the RPM filename for that packages, which consists of 4 parts:
- `NetworkManager`: name
- `1-56.0`: version
- `1.el10`: release
- `aarch64`: architecture

RPM packages are download from repositories and each RPM package contains:
- the files that the package installs in the system
- metadata about the package
- scripts that run when you install, update or remove the package

## Useful commands

To list all installed packages use `rpm -qa`:

```bash
[user@rocky ~]$ rpm -qa | head -5
libgcc-14.3.1-4.4.el10.aarch64
tzdata-2026a-1.el10.noarch
hwdata-0.379-10.8.el10.noarch
rocky-gpg-keys-10.2-1.1.el10.noarch
rocky-release-10.2-1.1.el10.noarch
```

To determine what package provides a specific file use `rpm -qf`:

```bash
[user@rocky ~]$ rpm -qf /etc/NetworkManager/NetworkManager.conf 
NetworkManager-1.56.0-1.el10.aarch64
```

```bash
[user@rocky ~]$ type pstree
pstree is hashed (/usr/bin/pstree)
[user@rocky ~]$ rpm -qf /usr/bin/pstree
psmisc-23.6-8.el10.aarch64
```

To list the files that a RPM package provides use `rpm -ql`:

```bash
[user@rocky ~]$ rpm -ql psmisc | head -5
/usr/bin/killall
/usr/bin/peekfd
/usr/bin/prtstat
/usr/bin/pslog
/usr/bin/pstree
```

## Install packages

Don't use `rpm` for installing packages, instead use `dnf`!
