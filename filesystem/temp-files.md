# Temporary files

Many applications use temporary files and directories. These files need to purged in order not to fill the disk space.

## Manage temporary files with `systemd-tmpfiles`

RHEL has `systemd-tmpfiles` that provides a structured method to create and delete temporary files and directories.

There are two `systemd` units that use the `systemd-tmpfiles`:
- `systemd-tmpfiles-setup.service`
- `systemd-tmpfiles-clean.timer`

### `systemd-tmpfiles-setup.service`

The first (`systemd-tmpfiles-setup.service`) is one of the first services launched by `systemd` and it creates and deletes the files and the directories (`systemd-tmpfiles --create --remove`) specified in:
- `/usr/lib/tmpfiles.d/*.conf`
- `/run/tmpfiles.d/*.conf`
- `/etc/tmpfiles.d/*.conf`

As always, `/usr/lib/` contains the standard configuration, while `/etc/` is where the custom configuration should be written, in which case it will override the default one.

```bash
[root@rocky /]$ cat /usr/lib/tmpfiles.d/tmp.conf 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# See tmpfiles.d(5) for details.

# Clear tmp directories separately, to make them easier to override
q /tmp 1777 root root 10d
q /var/tmp 1777 root root 30d
```

This is the default configuration for `/tmp` and `/var/tmp` and the `systemd-tmpfiles-setup` service ensures that these directories exists and have the right permissions (`1777`) and the right ownership (`root:root`).

### `systemd-tmpfiles-clean.timer`

The `systemd-tmpfiles-clean.timer` triggers at regular intervals the `systemd-tmpfiles-clean.service`, which executes the `systemd-tmpfiles --clean` command on the files and directories specified in the configuration files:
- `/usr/lib/tmpfiles.d/*.conf`
- `/run/tmpfiles.d/*.conf`
- `/etc/tmpfiles.d/*.conf`

The timer unit has a configuration file for configuring when to trigger it:

```bash
[root@rocky /]$ cat /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

This shows that the `systemd-tmpfiles-clean.service` gets triggered 15 minutes after boot and it is triggered again if more than 24 hours has passed since the last trigger.

```bash
[root@rocky /]$ cat /usr/lib/tmpfiles.d/tmp.conf 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# See tmpfiles.d(5) for details.

# Clear tmp directories separately, to make them easier to override
q /tmp 1777 root root 10d
q /var/tmp 1777 root root 30d
```

The `systemd-tmpfiles-clean.service` is the service that will purge the content of `/tmp` and `/var/tmp` according to their purging policy.
