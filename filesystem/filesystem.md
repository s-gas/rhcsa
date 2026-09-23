# File System

Files on a Linux system are organized into a single inverted tree of directories.

## Hierarchy

| location | files  |
|----------|----------|
| `/`      | root of the hierarchy |
| `/boot`  | boot process |
| `/dev`   | devices |
| `/etc`   | system configuration |
| `/home`  | home directories |
| `/root`  | `root` home directory |
| `/run`   | runtime data |
| `/tmp`   | temporary data |
| `/usr`   | installed software, shared libraries |
| `/var`   | system persistent dynamic data |

## `/etc`: useful files and directories

- `/etc/profile` and `/etc/profile.d/`

  Variables available to all users

- `/etc/sudoers` and `/etc/sudoers.d/`

  Configuration for `sudo` access

- `/etc/passwd`

  Information about users

- `/etc/group`

  Information about groups

- `/etc/shadow`

  Information about passwords

- `/etc/login.defs`

  Configuration for users and groups
