# DNF

DNF improve RPM-based software installation. It can be used to install, update, delete and get information about software packages and their dependencies. It also provides a history of its activities.

## Useful commands

For listing the **installed** packages, use `dnf list`:

```bash
[user@rocky ~]$ dnf list | head -5
Last metadata expiration check: 0:00:12 ago on Fri 09 Oct 2026 06:17:35 PM CEST.
Installed Packages
NetworkManager.aarch64                                                                   1:1.56.0-1.el10                             @anaconda        
NetworkManager-libnm.aarch64                                                             1:1.56.0-1.el10                             @anaconda        
NetworkManager-tui.aarch64                                                               1:1.56.0-1.el10                             @anaconda        
```

To search for packages by a keyword, either included in the package name or summary, use `dnf search`:

```bash
[user@rocky ~]$ dnf search 'NetworkManager' | head
Last metadata expiration check: 0:08:02 ago on Fri 09 Oct 2026 06:17:35 PM CEST.
===================== Name Exactly Matched: NetworkManager =====================
NetworkManager.aarch64 : Network connection manager and user applications
==================== Name & Summary Matched: NetworkManager ====================
NetworkManager-adsl.aarch64 : ADSL device plugin for NetworkManager
NetworkManager-bluetooth.aarch64 : Bluetooth device plugin for NetworkManager
NetworkManager-cloud-setup.aarch64 : Automatically configure NetworkManager in cloud
NetworkManager-config-connectivity-redhat.noarch : NetworkManager config file for connectivity checking via Red Hat servers
NetworkManager-config-server.noarch : NetworkManager config file for "server-like" defaults
NetworkManager-l2tp.aarch64 : NetworkManager VPN plugin for L2TP and L2TP/IPsec
NetworkManager-l2tp-gnome.aarch64 : NetworkManager VPN plugin for L2TP and L2TP/IPsec - GNOME files
```

To get detailed information about a package, including if it installed or not, use `dnf info`:

```bash
[user@rocky ~]$ dnf info NetworkManager
Last metadata expiration check: 0:09:32 ago on Fri 09 Oct 2026 06:17:35 PM CEST.
Installed Packages
Name         : NetworkManager
Epoch        : 1
Version      : 1.56.0
Release      : 1.el10
Architecture : aarch64
Size         : 6.1 M
Source       : NetworkManager-1.56.0-1.el10.src.rpm
Repository   : @System
From repo    : anaconda
Summary      : Network connection manager and user applications
URL          : https://networkmanager.dev/
License      : GPL-2.0-or-later AND LGPL-2.1-or-later
Description  : NetworkManager is a system service that manages network interfaces and
             : connections based on user or automatic configuration. It supports
             : Ethernet, Bridge, Bond, VLAN, Team, InfiniBand, Wi-Fi, mobile broadband
             : (WWAN), PPPoE and other devices, and supports a variety of different VPN
             : services.

Available Packages
Name         : NetworkManager
Epoch        : 1
Version      : 1.56.0
Release      : 2.el10_2
Architecture : aarch64
Size         : 2.1 M
Source       : NetworkManager-1.56.0-2.el10_2.src.rpm
Repository   : baseos
Summary      : Network connection manager and user applications
URL          : https://networkmanager.dev/
License      : GPL-2.0-or-later AND LGPL-2.1-or-later
Description  : NetworkManager is a system service that manages network interfaces and
             : connections based on user or automatic configuration. It supports
             : Ethernet, Bridge, Bond, VLAN, Team, InfiniBand, Wi-Fi, mobile broadband
             : (WWAN), PPPoE and other devices, and supports a variety of different VPN
             : services.
```

To get what package provides a specific file, use `dnf provides`:

```bash
[user@rocky /]$ dnf provides /etc/NetworkManager/
Last metadata expiration check: 0:01:08 ago on Fri 09 Oct 2026 06:45:44 PM CEST.
NetworkManager-1:1.56.0-1.el10.aarch64 : Network connection manager and user applications
Repo        : @System
Matched from:
Other       : /etc/NetworkManager/

NetworkManager-1:1.56.0-1.el10.aarch64 : Network connection manager and user applications
Repo        : baseos
Matched from:
Other       : /etc/NetworkManager/

NetworkManager-1:1.56.0-2.el10_2.aarch64 : Network connection manager and user applications
Repo        : baseos
Matched from:
Other       : /etc/NetworkManager/
```

The `dnf provides` command works even without providing the full path. For example you might know only the name of the `command` that you want to use:

```bash
[root@localhost ~]# dnf provides sealert
Extra Packages for Enterprise Linux 10 - aarch64                                                                                                                                     5.1 MB/s |  18 MB     00:03    
Rocky Linux 10 - AppStream                                                                                                                                                           6.0 MB/s |  30 MB     00:05    
Last metadata expiration check: 0:00:03 ago on Sun 25 Oct 2026 12:03:52 PM CET.
setroubleshoot-server-3.3.35-4.el10.aarch64 : SELinux troubleshoot server
Repo        : appstream
Matched from:
Filename    : /usr/bin/sealert
```

To install a package, use `dnf install`:

```bash
[root@rocky ~]$ dnf install tldr
Last metadata expiration check: 0:26:17 ago on Fri 09 Oct 2026 06:22:33 PM CEST.
Dependencies resolved.
=====================================================================================================================================================================================================================
 Package                                                  Architecture                                  Version                                               Repository                                        Size
=====================================================================================================================================================================================================================
Installing:
 tldr                                                     noarch                                        3.3.0-3.el10_0                                        epel                                              37 k
Installing dependencies:
 python3-colorama                                         noarch                                        0.4.6-13.el10                                         appstream                                         74 k
 python3-shtab                                            noarch                                        1.7.1-3.el10_0                                        epel                                              40 k
 python3-termcolor                                        noarch                                        2.5.0-1.el10_0                                        epel                                              21 k

Transaction Summary
=====================================================================================================================================================================================================================
Install  4 Packages

Total download size: 172 k
Installed size: 431 k
Is this ok [y/N]: y
Downloading Packages:
(1/4): python3-shtab-1.7.1-3.el10_0.noarch.rpm                                                                                                                                       287 kB/s |  40 kB     00:00    
(2/4): python3-colorama-0.4.6-13.el10.noarch.rpm                                                                                                                                     1.3 MB/s |  74 kB     00:00    
(3/4): tldr-3.3.0-3.el10_0.noarch.rpm                                                                                                                                                147 kB/s |  37 kB     00:00    
(4/4): python3-termcolor-2.5.0-1.el10_0.noarch.rpm                                                                                                                                    79 kB/s |  21 kB     00:00    
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                152 kB/s | 172 kB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                             1/1 
  Installing       : python3-colorama-0.4.6-13.el10.noarch                                                                                                                                                       1/4 
  Installing       : python3-termcolor-2.5.0-1.el10_0.noarch                                                                                                                                                     2/4 
  Installing       : python3-shtab-1.7.1-3.el10_0.noarch                                                                                                                                                         3/4 
  Installing       : tldr-3.3.0-3.el10_0.noarch                                                                                                                                                                  4/4 
  Running scriptlet: tldr-3.3.0-3.el10_0.noarch                                                                                                                                                                  4/4 

Installed:
  python3-colorama-0.4.6-13.el10.noarch                  python3-shtab-1.7.1-3.el10_0.noarch                  python3-termcolor-2.5.0-1.el10_0.noarch                  tldr-3.3.0-3.el10_0.noarch                 

Complete!
```

The example shows how `dnf install` installs also the necessary dependencies for the desired package.

To update all packages, use `dnf update`:

```bash
[root@rocky ~]$ dnf update
Last metadata expiration check: 0:30:00 ago on Fri 09 Oct 2026 06:22:33 PM CEST.
Dependencies resolved.
Nothing to do.
Complete!
```

To remove a package, use `dnf remove`:

```bash
[root@rocky ~]$ dnf remove tldr
Dependencies resolved.
=====================================================================================================================================================================================================================
 Package                                                 Architecture                                 Version                                                 Repository                                        Size
=====================================================================================================================================================================================================================
Removing:
 tldr                                                    noarch                                       3.3.0-3.el10_0                                          @epel                                             82 k
Removing unused dependencies:
 python3-colorama                                        noarch                                       0.4.6-13.el10                                           @appstream                                       190 k
 python3-shtab                                           noarch                                       1.7.1-3.el10_0                                          @epel                                            126 k
 python3-termcolor                                       noarch                                       2.5.0-1.el10_0                                          @epel                                             33 k

Transaction Summary
=====================================================================================================================================================================================================================
Remove  4 Packages

Freed space: 431 k
Is this ok [y/N]: y
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                             1/1 
  Erasing          : tldr-3.3.0-3.el10_0.noarch                                                                                                                                                                  1/4 
  Erasing          : python3-colorama-0.4.6-13.el10.noarch                                                                                                                                                       2/4 
  Erasing          : python3-shtab-1.7.1-3.el10_0.noarch                                                                                                                                                         3/4 
  Erasing          : python3-termcolor-2.5.0-1.el10_0.noarch                                                                                                                                                     4/4 
  Running scriptlet: python3-termcolor-2.5.0-1.el10_0.noarch                                                                                                                                                     4/4 

Removed:
  python3-colorama-0.4.6-13.el10.noarch                  python3-shtab-1.7.1-3.el10_0.noarch                  python3-termcolor-2.5.0-1.el10_0.noarch                  tldr-3.3.0-3.el10_0.noarch                 

Complete!
```

The example shows how `dnf remove` also removes unused dependencies.

To view the history of the transactions (installation/removals), run `dnf history`:

```bash
[root@rocky ~]$ dnf history
ID     | Command line                                                                                                                                                   | Date and time    | Action(s)      | Altered
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    12 | install tldr                                                                                                                                                   | 2026-10-09 18:53 | Install        |    4   
    11 | remove tldr                                                                                                                                                    | 2026-10-09 18:53 | Removed        |    4   
    10 | update                                                                                                                                                         | 2026-10-09 18:51 | I, U           |  121   
     9 | install tldr                                                                                                                                                   | 2026-10-09 18:48 | Install        |    4   
     8 | install vim                                                                                                                                                    | 2026-09-13 12:13 | I, U           |    7   
     7 | install -y bzip2                                                                                                                                               | 2026-09-13 11:59 | Install        |    1   
     6 | install -y policycoreutils-python-utils                                                                                                                        | 2026-09-13 11:58 | I, U           |    8   
     5 | install -y docker-ce docker-ce-cli containerd.io                                                                                                               | 2026-09-13 11:57 | Install        |    9  <
     4 | install -y ttyd                                                                                                                                                | 2026-09-13 11:57 | Install        |    4 ><
     3 | install -y epel-release                                                                                                                                        | 2026-09-13 11:57 | Install        |    1 >E
     2 | install -y tmux                                                                                                                                                | 2026-09-13 11:57 | Install        |    1  <
     1 |                                                                                                                                                                | 2026-09-13 11:21 | Install        |  346 >E
```

You can also undo a transaction with `dnf history undo`:

```bash
[root@rocky ~]$ dnf history undo 12
Last metadata expiration check: 0:36:37 ago on Fri 09 Oct 2026 06:22:33 PM CEST.
Dependencies resolved.
=====================================================================================================================================================================================================================
 Package                                                 Architecture                                 Version                                                 Repository                                        Size
=====================================================================================================================================================================================================================
Removing:
 tldr                                                    noarch                                       3.3.0-3.el10_0                                          @epel                                             82 k
Removing dependent packages:
 python3-colorama                                        noarch                                       0.4.6-13.el10                                           @appstream                                       190 k
 python3-shtab                                           noarch                                       1.7.1-3.el10_0                                          @epel                                            126 k
 python3-termcolor                                       noarch                                       2.5.0-1.el10_0                                          @epel                                             33 k

Transaction Summary
=====================================================================================================================================================================================================================
Remove  4 Packages

Freed space: 431 k
Is this ok [y/N]: y
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                             1/1 
  Erasing          : tldr-3.3.0-3.el10_0.noarch                                                                                                                                                                  1/4 
  Erasing          : python3-colorama-0.4.6-13.el10.noarch                                                                                                                                                       2/4 
  Erasing          : python3-shtab-1.7.1-3.el10_0.noarch                                                                                                                                                         3/4 
  Erasing          : python3-termcolor-2.5.0-1.el10_0.noarch                                                                                                                                                     4/4 
  Running scriptlet: python3-termcolor-2.5.0-1.el10_0.noarch                                                                                                                                                     4/4 

Removed:
  python3-colorama-0.4.6-13.el10.noarch                  python3-shtab-1.7.1-3.el10_0.noarch                  python3-termcolor-2.5.0-1.el10_0.noarch                  tldr-3.3.0-3.el10_0.noarch                 

Complete!
```

## DNF Repositories

To view the **enabled** repositories, run `dnf repolist`:

```bash
[root@rocky ~]$ dnf repolist
repo id                                                                                    repo name
appstream                                                                                  Rocky Linux 10 - AppStream
baseos                                                                                     Rocky Linux 10 - BaseOS
docker-ce-stable                                                                           Docker CE Stable - aarch64
epel                                                                                       Extra Packages for Enterprise Linux 10 - aarch64
extras                                                                                     Rocky Linux 10 - Extras
```

To view **all** repositories, run `dnf repolist all`:

```bash
[root@rocky ~]$ dnf repolist all | head -5
repo id                    repo name                                    status
appstream                  Rocky Linux 10 - AppStream                   enabled
appstream-debuginfo        Rocky Linux 10 - AppStream - Debug           disabled
appstream-source           Rocky Linux 10 - AppStream - Source          disabled
baseos                     Rocky Linux 10 - BaseOS                      enabled
```

Repositories can be managed through `dnf config-manager` or by modifying directly the `.repo` files in `/etc/yum.repos.d/`:

```bash
[root@rocky ~]$ cat /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-$releasever&arch=$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
countme=1
enabled=1

[epel-debuginfo]
name=Extra Packages for Enterprise Linux $releasever - $basearch - Debug
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/$basearch/debug/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-debug-$releasever&arch=$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
enabled=0

[epel-source]
name=Extra Packages for Enterprise Linux $releasever - $basearch - Source
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-source-$releasever&arch=source
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
enabled=0
```

`GPG` is the key that `dnf` uses to verify that RPM packages are saved to download.

By running `dnf config-manager --disable` the `enabled` entry of the repository gets set to `0`:

```bash
[root@rocky ~]$ dnf config-manager --disable epel
[root@rocky ~]$ cat /etc/yum.repos.d/epel.repo 
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-$releasever&arch=$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
countme=1
enabled=0
```

By running `dnf config-manager --enable` the `enabled` entry of the repository gets set to `1`:

```bash
[root@rocky ~]$ dnf config-manager --enable epel
[root@rocky ~]$ cat /etc/yum.repos.d/epel.repo 
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-$releasever&arch=$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
countme=1
enabled=1
```

To add a new repository, you can run `dnf config-manager --add-repo` and specifying the URL:

```bash
[root@localhost ~]# dnf config-manager --add-repo 'file:///mnt/BaseOS'
Adding repo from: file:///mnt/BaseOS
```

This will create the `.repo` file in `/etc/yum.repo.d/` and enable the repository:

```bash
[root@localhost yum.repos.d]# cat mnt_BaseOS.repo 
[mnt_BaseOS]
name=created by dnf config-manager from file:///mnt/BaseOS
baseurl=file:///mnt/BaseOS
enabled=1
```

> This can be also done by manually creating the `.repo` file.

There are also packages which only purpose is to install a new repository, they are called **release packages**:

```bash
[root@localhost yum.repos.d]# dnf list epel-release
Last metadata expiration check: 1:45:00 ago on Sat 24 Oct 2026 05:01:24 PM CEST.
Available Packages
epel-release.noarch                                  10-7.el10_1                                   extras
[root@localhost yum.repos.d]# dnf install epel-release -y | tail -5

Installed:
  epel-release-10-7.el10_1.noarch                                               

Complete!
[root@localhost yum.repos.d]# ls
epel.repo          rocky-addons.repo  rocky-extras.repo  rocky-security.repo
epel-testing.repo  rocky-devel.repo   rocky.repo
```

Installing `epel-release` created two new repos: `epel.repo` and `epel-testing.repo`, which can be used to install other packages.
